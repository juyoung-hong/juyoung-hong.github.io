---
title: "Antigravity2 - (실습) 로그인 구현"
classes: wide
categories:
  - vibecoding
tags:
  - antigravity
---

<br>

# 실습

지난글에서 실습한 내용을 바탕으로 지금 만들어보고 있는 코인 자동 매매 프로그램을 다시 구축해보기로 했다.

MSA는 너무 복잡하고 작업시간이 너무 오래 걸리는 문제가 있어, OKE로 배포할거긴 하지만 모놀리식 구조로 가장 간단한 방법으로 구축하고자 한다.

먼저 로그인 페이지를 구현하고 로그인 했을때 메인 화면으로 넘어갈 수 있는 기능을 만들려고 한다.

<br>

## 초기 설정

먼저 Git에서 프로젝트를 하나 생성하고 로컬에 클론해온 뒤 안티그래비티로 폴더를 열었다.

먼저 `.agent/rules` 폴더를 생성하고 프로젝트에 대한 규칙을 설정 했다.

Gemini Pro를 사용하여 만들고자 하는 프로젝트의 대략적인 내용에 대해서 설명했고, rule 파일로 작성할 내용에 대해서 최대한 상세하게 분할하여 작성해달라고 해서 아래와 같은 내용들을 rule로 정의하였다.

### architecture.md

```markdown
# Directory Structure:

project_root/
├── .gitignore               # Git 무시 파일 설정
├── Dockerfile               # Oracle Client 포함, 실행용 이미지
├── Wallet_CATSDB            # Oracle ADB에 사용하는 Wallet
├── requirements.txt         
├── app/                     # 애플리케이션 소스 코드 루트
│   ├── __init__.py
│   ├── main.py              # [Entry Point] 앱 진입점, 라우팅
│   ├── core/                # [Core] 공통 설정, DB 연결, 로깅
│   │   ├── __init__.py
│   │   ├── config.py        # 환경변수 로드 (Pydantic Settings)
│   │   ├── database.py      # Oracle Connection Pool 관리
│   │   ├── logger.py        # 시스템 로깅 설정
│   │   └── scheduler.py     # APScheduler 인스턴스 및 설정
│   ├── models/              # [Domain] 데이터 구조 정의 (Pydantic/DTO)
│   │   ├── __init__.py
│   │   ├── user_model.py    # 로그인 관련 사용자 객체 정의
│   │   └── trade_model.py   # 거래 기록 및 설정 객체 정의
│   ├── repositories/        # [Data Access] DB 쿼리 실행 (SQL)
│   │   ├── __init__.py
│   │   ├── user_repo.py     # 사용자 인증 관련 쿼리
│   │   └── trade_repo.py    # 거래 내역 저장/조회, 전략 설정 로드 쿼리
│   ├── services/            # [Business Logic] UI와 DB 사이의 로직 처리
│   │   ├── __init__.py
│   │   ├── auth_service.py  # 로그인/세션 관리 로직
│   │   └── trade_service.py # 대시보드 데이터 가공, 수동 주문 요청
│   ├── trading_engine/      # [Engine] 자동매매 핵심 로직 (독립 실행 가능)
│   │   ├── __init__.py
│   │   ├── executor.py      # 주기적(ex.5분)으로 전략을 실행하는 오케스트레이터
│   │   ├── base_classes.py  # 전략(Strategy) 및 거래소(Exchange) 추상 클래스
│   │   ├── exchanges/       # 거래소별 구현체 폴더
│   │   │   ├── __init__.py
│   │   │   ├── upbit_api.py
│   │   │   └── binance_api.py
│   │   └── strategies/      # 매매 전략 구현체 폴더
│   │       ├── __init__.py
│   │       └── rsi_strategy.py
│   └── views/               # [UI] Streamlit 화면 렌더링 (로직 포함 금지, 각 버튼에는 API 연동)
│       ├── __init__.py
│       ├── components.py    # 재사용 가능한 UI 컴포넌트 (헤더, 사이드바)
│       ├── login_view.py    # 로그인 화면
│       ├── home_view.py     # 메인 대시보드
│       └── settings_view.py # 전략 파라미터 설정 화면

# Detailed File Responsibilities

## Root Directory
- Dockerfile: Python 3.12 베이스. Oracle Instant Client 라이브러리 설치 필수. CMD는 streamlit run app/main.py.

- requirements.txt: streamlit, python-oracledb, pandas, pydantic-settings, apscheduler, ccxt, bcrypt 등 명시.

## app/core/ (Infrastructure)
- config.py: pydantic_settings를 사용하여 OS 환경 변수(DB Credential, Exchange API Keys)를 로드. 코드 내 하드코딩 금지.

- database.py: python-oracledb를 사용하여 Connection Pool을 생성하는 싱글톤 객체. get_db_connection() 컨텍스트 매니저 제공.

- scheduler.py: APScheduler의 BackgroundScheduler를 설정. 스케줄러가 중복 실행되지 않도록 Singleton 패턴 또는 Lock 처리.

## app/models/ (Domain Objects)
- user_model.py: User 정보를 담는 Pydantic 모델 (id, username, created_at).

- trade_model.py: 전략 파라미터(target_coin, investment_amount, interval) 및 거래 로그 데이터 구조 정의.

## app/repositories/ (Data Access Layer)
- 규칙: 이곳에서만 SQL이 작성되어야 함. 모든 SQL은 :bind_variable 문법 사용.

- user_repo.py: SELECT * FROM USERS WHERE USERNAME = :username 등 인증 쿼리.

- trade_repo.py:

  - get_active_strategies(): 활성화된 봇 설정 로드.

  - log_trade_execution(): 매매 결과 DB 저장.

## app/services/ (Business Logic Layer)
- 규칙: Streamlit UI(st.*)를 사용하지 않음. 순수 Python 로직.

- auth_service.py: verify_user(username, password) -> Password Hash 검증 후 User 객체 반환.

- trade_service.py: UI에서 요청한 데이터(차트용 OHLCV, 현재 자산)를 거래소나 DB에서 가져와 Pandas DataFrame으로 가공하여 반환.

## app/trading_engine/ (The Brain)
- base_classes.py:

  - BaseExchange: fetch_balance, create_order, fetch_ticker 추상 메서드 정의.

  - BaseStrategy: calculate_signal(market_data) -> buy/sell/hold 반환하는 추상 메서드 정의.

  - exchanges/*.py: ccxt 라이브러리 등을 래핑하여 거래소별 인증 및 API 호출 구현.

  - strategies/*.py: 구체적인 알고리즘(예: RSI < 30 이면 매수). DB나 Config에 의존하지 않고 오직 데이터 입력 -> 신호 출력에 집중.

- executor.py:

DB에서 활성 전략 및 설정(코인, 거래소, 금액) 로드.

적절한 Exchange와 Strategy 클래스 인스턴스화.

시장 데이터 조회 -> 전략 실행 -> 주문 집행 -> DB 로그 저장을 순차적으로 수행하는 함수 (run_trading_job). 이 함수가 스케줄러에 등록됨.

## app/views/ (Presentation Layer)
- 규칙: 비즈니스 로직 금지. 오직 st.session_state 조회 및 Service 호출 결과 렌더링.

  - login_view.py: st.text_input으로 ID/PW 입력받고 auth_service 호출. 성공 시 st.session_state['authenticated'] = True.

  - home_view.py: trade_service에서 받은 데이터로 대시보드(차트, 현재 수익률) 구성. 봇 Start/Stop 버튼 제공.

## app/main.py (Application Entry)
- 역할:

앱 시작 시 st.set_page_config 설정.

st.session_state 초기화 (로그인 상태 등).

core.scheduler가 실행 중인지 확인하고, 없으면 백그라운드 스레드로 시작 (executor.run_trading_job 등록).
```

### coding-standards.md

```markdown
# Coding Style & Standards

## 1. Python Style Guide
- PEP 8 준수: 모든 파이썬 코드는 PEP 8 스타일 가이드를 따른다.
- Type Hinting: 모든 함수의 인자(Arguments)와 반환 값(Return values)에 반드시 타입을 명시한다.

Python
def get_user_balance(username: str) -> float:
    ...

- Docstrings: 모든 클래스와 함수 상단에 Google 스타일의 Docstring을 작성하여 목적과 파라미터를 설명한다.

## 2. Naming Conventions
- Variables & Functions: snake_case를 사용한다. (예: user_balance, calculate_rsi)
- Classes: PascalCase를 사용한다. (예: AuthService, UpbitExchange)
- Constants: UPPER_SNAKE_CASE를 사용한다. (예: MAX_RETRIES, DEFAULT_INTERVAL)
- Database Columns: Oracle DB 관례에 따라 모델 정의 시 대문자를 고려하거나 명시적 매핑을 사용한다.

## 3. Modular Programming (Anti-Spaghetti)
- Functions: 하나의 함수는 하나의 일만 수행해야 하며, 50라인을 넘기지 않도록 노력한다.
- Dry Principle: 중복되는 코드는 반드시 core/ 또는 utils/로 공통화한다.
- Main Method Logic: app/main.py의 main() 함수는 오직 라우팅과 초기화 로직만 담는다. 구체적인 기능은 각 레이어(Views, Services) 파일에 작성한 후 호출한다.

## 4. Error Handling & Logging
- Try-Except: 외부 API 호출(거래소) 및 DB 연동 시 반드시 try-except 블록을 사용한다.
- Explicit Exceptions: 단순히 except Exception:을 지양하고, 가능한 구체적인 에러 타입을 지정한다.
- Logging: print() 대신 app/core/logger.py에 정의된 로거를 사용하여 로그 레벨(INFO, ERROR, DEBUG)을 구분한다.

## 5. Security Standards
- Hardcoding: API Key, DB 비밀번호 등 민감한 정보를 코드에 직접 작성하는 것을 절대 금지한다. 반드시 app/core/config.py를 통해 환경 변수에서 로드한다.
- SQL Injection: 모든 SQL은 바인딩 변수를 사용하며, 사용자 입력값을 쿼리 문자열에 직접 포함하지 않는다.
```

### techstack.md

```markdown
# Technology Stack & Environment Guidelines

## 1. Runtime & Package Management

### Rule 1.1: Python Version
- 모든 코드는 Python 3.12 환경에서 실행 가능해야 한다.
- 최신 문법(Type Hinting, f-strings 등)을 적극 활용하되, 호환성을 유지한다.

### Rule 1.2: Package Management with uv
- 프로젝트 패키지는 pip 대신 uv를 사용하여 관리한다.
- 에이전트는 라이브러리 추가 시 uv add [package] 형식을 따르며, uv.lock과 pyproject.toml 파일의 정합성을 유지해야 한다.
- 가상환경 경로는 프로젝트 루트의 .venv를 기본으로 한다.

## 2. Frontend & UI Framework

### Rule 2.1: Streamlit Implementation
- UI는 streamlit을 사용한다.
- Session Persistence: st.session_state를 사용하여 사용자 인증 상태 및 임시 데이터를 관리한다.
- Component modularization: 복잡한 UI는 app/views/components.py로 분리하여 재사용한다.

## 3. Database & Persistence

### Rule 3.1: Oracle Autonomous Database
- 드라이버: python-oracledb를 사용한다.
- Connection Mode: 배포 환경의 경량화를 위해 Thin Mode 사용을 우선하되, 복잡한 Oracle 기능 필요 시 Thick Mode로 전환할 수 있도록 설계를 유연하게 가져간다.
- Security: 모든 쿼리는 반드시 Bind Variables를 사용한다. (cursor.execute("... WHERE id = :1", [id]))

### Rule 3.2: Database Migration

- 테이블 스키마 관리는 초기 단계이므로 app/repositories/schema.sql에 정의하고, 앱 시작 시 테이블 존재 여부를 체크하여 자동 생성하는 로직을 포함할 수 있다.

## 4. Trading & Scheduling

### Rule 4.1: Task Scheduling
- APScheduler의 BackgroundScheduler를 사용하여 거래 로직을 주기적으로 실행한다.
- 메인 프로세스(Streamlit)와 스케줄러 프로세스가 자원을 효율적으로 공유하도록 설계한다.

### Rule 4.2: Exchange Integration
- 거래소 공통 인터페이스로 ccxt 라이브러리를 사용한다.
- 거래소별로 ccxt에서도 매수/매도 등 같은 동작을 하지만 다른 코드를 사용해야하므로 전략코드를 작성할때 수월하도록 거래소별 차이를 통합한 추상화 공통 클래스를 작성한다.
- 거래소별 API Key 및 Secret은 환경변수(app/core/config.py)를 통해 관리한다.

## 5. Deployment & Infrastructure

### Rule 5.1: Cross-Platform Docker Build (AMD64 to ARM64)
- 개발은 AMD(x86_64) 환경에서 진행하나, 최종 타겟은 ARM64 (OKE) 환경임을 명심한다.
- Multi-platform Build: Dockerfile 작성 시 특정 아키텍처에 종속된 바이너리 설치를 지양한다.
- Oracle Instant Client: ARM64용 Oracle Instant Client 라이브러리가 설치되도록 Dockerfile 내 아키텍처 분기 로직을 고려하거나, 아키텍처 중립적인 구성을 취한다.

### Rule 5.2: Containerization
- Base Image: python:3.12-slim 또는 python:3.12-bullseye를 사용하여 이미지 크기를 최적화한다.
- 환경변수 주입: OKE 배포 시 ConfigMap 또는 Secret을 통해 민감 정보를 주입받을 수 있도록 설계한다.
```

### db-connection.md

```markdown
# Oracle ADB & Connection Rules (Thin Mode)

## 1. Connection Strategy: Thin Mode
- No Instant Client Required: python-oracledb 드라이버의 Thin Mode를 사용하므로, 별도의 Oracle Instant Client 설치 없이 순수 파이썬 라이브러리만으로 접속한다.
- Wallet Location: 프로젝트 루트의 Wallet_CATSDB 폴더 내의 파일들을 사용한다.
- Connection Pool: 앱 실행 시 app/core/database.py에서 전역 커넥션 풀을 초기화하고, 세션마다 풀에서 커넥션을 획득하여 사용한다.

## 2. Wallet Configuration
- 에이전트는 Wallet_CATSDB 내의 tnsnames.ora, sqlnet.ora, ewallet.pem (또는 cwallet.sso) 파일을 인식해야 한다.
- sqlnet.ora 파일 내의 DIRECTORY 경로를 런타임 시 실제 컨테이너 내의 절대 경로로 동적 매핑하거나, connect_get_config() 옵션을 통해 설정한다.

## 3. Database Module Code Standard (app/core/database.py)
에이전트는 아래의 코드 스타일을 준수하여 DB 모듈을 생성해야 한다:

import oracledb
import os
from app.core.config import settings

# Thin Mode 설정 (기본값이지만 명시적 확인)
def init_db_pool():
    # Wallet 경로 설정
    wallet_dir = os.path.join(os.getcwd(), "Wallet_CATSDB")
    wallet_password = settings.DB_WALLET_PASSWORD # 환경변수에서 로드

    pool = oracledb.create_pool(
        user=settings.DB_USER,
        password=settings.DB_PASSWORD,
        dsn=settings.DB_DSN, # tnsnames.ora에 정의된 서비스명
        min=2,
        max=10,
        increment=1,
        wallet_location=wallet_dir,
        wallet_password=wallet_password
    )
    return pool
```

## 실행

위에 작성한 rule을 기반으로 전체적인 구현에 대한 계획작성을 우선 요청하였다.

Planning이 가장 중요하다고 생각하여 Gemini-Pro 모델을 선택하여 아래와 같은 프롬프트로 구현 계획을 상세히 단계별로 작성할 것을 요청하였다.

```python
# @architecture.md 아키텍처를 참고하여 비트코인을 자동매매하는 프로그램을 만들기 위한 상세하고 구체적인 단계별 계획을 세워보세요. 가장 먼저 구현하고자 하는 기능은 첫 진입시 로그인 화면이 등장하고, 로그인을 진행하고 나면 비로소 메인 화면으로 접근하고자 합니다. 복잡하게 만들지 말고 가장 간단한 방법으로 계획하세요
```

위 명령을 주자 아래 이미지와 같이 앞으로 할 일에 대해서 작성하였다.

내가 보고 뭔가 다듬어야 할 부분들이 있으면 코멘트를 추가하여 리뷰를 진행했고, 어느정도 리뷰가 끝난이후에는 에이전트를 Fast 모드로 바꾸고, Gemini3 Flash 모델을 사용하여 계획한대로 진행하라고 하였다.

![antigravity-plan]({{ juyoung-hong.github.io }}/assets/images/antigravity-plan.jpg)

이후 오른쪽 메뉴와 같이 코딩을 시작하였고, 테스트 중 간단히 발생하는 오류는 내가 직접 수정하거나 에이전트에게 요청하여 수정을 진행하니 이전의 코파일럿을 사용했을때보다 훨씬 빠르게 기능을 구현하고 테스트 중 발생하는 오류도 금방 잡아내는 것 같았다.

![antigravity-running]({{ juyoung-hong.github.io }}/assets/images/antigravity-running.jpg)

