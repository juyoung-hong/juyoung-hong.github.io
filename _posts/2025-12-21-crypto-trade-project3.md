---
title: "비트코인 자동매매 프로젝트3 - backend-web 구성"
classes: wide
categories:
  - cryptotrade
tags:
  - project
  - cryptotrade
  - python
---

<br>

# 개요

지난 글에서는 구성된 OKE 클러스터에 Jenkins CI/CD를 붙여서 Frontend-Desktop Repo에서 개발하면 그 내용을 배포하는 과정을 자동화 했다.

이제는 로그인 기능을 껍데기 뿐 아니라 실제로 동작하게끔 만들려고 한다.

DB에 로그인 관련 테이블 구성이 필요할거 같고, 로그인 로그를 기록하는 것과 frontend와의 연계 등의 작업이 필요할 것 같다.

<br>

# Github Repo

Frontend 에 맞추어서 자동매매랑 상관없이 웹서비스를 하는 것들을 다 모아서 "backend-web" service로 만들려고 한다.

우선 GitHub에 새로운 private repo를 하나 만들어 주었다.

```bash
git clone https://github.com/CryptoAutoTradingTeam/Backend-Web.git
uv init --python 3.12
uv add fastapi uvicorn
```

# 로그인 (인증, 인가) 구현

아래와 같은 폴더 구조를 만들어주고, 필요한 패키지를 설치해주었다.

```bash
backend-web/
├── app/
│   ├── api/
│   │   └── v1/
│   │       ├── users.py       # 회원가입 및 내 정보 조회
│   │       └── auth.py        # 로그인 (토큰 발행)
│   ├── core/
│   │   ├── config.py
│   │   └── security.py     # JWT 생성 및 비밀번호 해싱 로직
│   ├── models/
│   │   └── user.py         # User DB 모델
│   ├── repositories/
│   │   └── user_repository.py
│   ├── schemas/
│   │   ├── user.py         # User 관련 DTO
│   │   └── token.py        # Token 응답 형식
│   ├── services/
│   │   ├── user_service.py
│   │   └── auth_service.py # 로그인 비즈니스 로직
│   └── main.py
```

Router - Service - Repository 패턴으로 계층을 분리하고 로그인이 가능하도록 Gemini를 통해 코드를 짜달라고 했다.

코드가 완성된 이후에는 아래 명령어로 실행 후, 유저 생성, 생성된 아이디로 로그인, 로그인 정보로 내 정보 가져오기 시나리오로 오류가 발생하지 않을 때 까지 테스트했다.

```bash
uv run uvicorn app.main:app --reload
```

정상적으로 테스트를 완료한 이후에 pytest르ㄹ 추가하였고, backend-web 폴더 하위에 tests 폴더를 만들어, test_auth.py, test_users.py 파일을 생성하고 테스트 과정을 자동화했다.

```bash
uv run pytest
```

# ADB 생성

로그인 기능을 구현하였으니 로그인 및 회원 정보를 저장할 수 있도록 DB를 연결 해야 했고 오라클 프리티어에서 제공하는 ADB를 활용하고자 한다.

ADB가 위치할 Subnet을 생성하고, ADB에 대한 private-endpoint 구성 및 연결 작업을 진행해줄 것이다.

## DB Subent 생성

- OCI 콘솔의 Networking → Virtual Cloud Networks → crypto-prd-default-vcn → Subnets → Create Subnet
  - Name: dbs
  - IPv4 CIDR Block: 기존에 생성된 subnet들과 겹치지 않도록 IP 대역 구성
  - Subnet Access: Private Subnet

## Security List 구성

- OCI 콘솔의 Networking → Virtual Cloud Networks → crypto-prd-default-vcn → Security → Create Security List
  - Name: seclist-dbs
  - Ingress Rule: Kubernetes API endpoint, worker-nodes, pods subnet의 CIDR 대역으로 TCP 통신 해제
  - Egress Rule: 0.0.0.0/0 CIDR에대해서 All 해제

- OCI 콘솔의 Networking → Virtual Cloud Networks → crypto-prd-default-vcn → Subnets → dbs → Security → Add Security List
  - seclist-dbs

## Autonomous DB 생성

- OCI 콘솔의 Oracle AI Database → Autonomous AI Databases → Create Autonomous AI Database
  - Name: CATSDB
  - Workload Type: Transaction Processing
  - Database configuration: Always Free
  - Database Version: 26ai
  - Network Access: Secure Access from allowed IPs and VCNs only
    - Virtual Cloud Network: crypto-prd-default-vcn

DB 생성 중에 보니, free-tier라서 그런지 private endpoint는 만들수 없어서 위의 서브넷 생성 및 시큐리티 리스트 작업은 필요가 없었다.

## dbeaver 설치 및 접속

DB에 접속하기 위해 dbeaver community 버전을 다운로드 받아 실행하였다.

ADB는 특수한 방법으로 연결해줘야하는데, Oracle DB를 선택 후 연결하는 화면에서 Driver settings → Libraries에서 현재 노출되는 내용 전체 선택 후 Delete → Add Artifact 후 아래 내용 입력 후 확인

```text
implementation 'com.oracle.database.jdbc:ojdbc10:19.22.0.0'
implementation 'com.oracle.database.security:oraclepki:23.3.0.23.09'
implementation 'com.oracle.database.security:osdt_core:21.13.0.0'
implementation 'com.oracle.database.security:osdt_cert:21.13.0.0'
```

Download/Update 버튼 클릭 후 확인

- OCI 콘솔의 Oracle AI Database → Autonomous AI Databases → CATSDB → DataBase Connection → Download client credentials → Download Wallet

- Dbeaver Oracle Database 연결 페이지 → Custom → JDBC URL Template 입력
  - Username: ADMIN
  - Password: 기존에 설정했던 비밀번호

```text
jdbc:oracle:thin:@{Database 이름_high}?TNS_ADMIN={Wallet Directory}
```

Test Connect를 클릭해보고, Connected! 메시지를 보면 정상적으로 연결된 것이다.

# User 스키마 생성 및 테이블 생성

아래 명령어로 웹에서 공통적으로 사용할 스키마를 생성하였고,

```SQL
CREATE USER WEB_COMMON IDENTIFIED BY PW;
```

아래 명령어로 ADB에서 사용할 적절한 권한을 부여하였다.

```SQL
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW, CREATE SEQUENCE TO WEB_COMMON;
ALTER USER WEB_COMMON QUOTA UNLIMITED ON DATA;
```

GEMINI에게 요청해서 User 테이블에서 사용할 테이블 스키마를 구성하고, 인덱스 설정, update trigger 설정등을 진행하였다.

# 생성된 DB 및 테이블 기준으로 코드 수정

아래 명령어로 oracledb 패키지를 추가하였다.

```bash
uv add oracledb
```

이후 app/core/database.py 파일을 작성하고, repositories/user_repository.py 파일을 작성하고, models/user.py 파일을 작성하였다.

프로젝트의 상위에 .env파일을 작성하여 db 접속 관련 환경변수들을 입력해주었다.

db 접속이 성공적으로 가능한 것을 확인하고, 이전에 작성한 코드들에 db 의존성을 주입하여 코드를 완성하였다.

# TEST

마지막으로 아래 명령어로 TEST 용도의 스키마를 생성해주었다.

```SQL
CREATE USER WEB_COMMON_TEST IDENTIFIED BY PW;
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW, CREATE SEQUENCE TO WEB_COMMON_TEST;
ALTER USER WEB_COMMON_TEST QUOTA UNLIMITED ON DATA;
```

이후 tests폴더의 conftest.py 파일을 수정하고, .env 파일에도 TEST 스키마의 로그인 정보를 저장해주었다.

몇 차례 코드 수정 및 디버깅 이후 정상적으로 테스트 결과가 모두 pass 되어, main에 반영하였다.

```bash
PYTHONPATH=. uv run pytest 
```