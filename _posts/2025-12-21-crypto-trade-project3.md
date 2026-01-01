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

# 배포

## Docker file 작성

```Docker file
# 1. 빌드 스테이지 (uv를 활용한 의존성 추출)
FROM python:3.12-slim-bookworm AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app
# 환경변수 설정: 가상환경 생성 방지 및 시스템 파이썬 사용
ENV UV_SYSTEM_PYTHON=1

COPY pyproject.toml uv.lock ./
RUN uv pip install --no-cache -r pyproject.toml

# 2. 실행 스테이지
FROM python:3.12-slim-bookworm
WORKDIR /app

# Oracle Instant Client 실행에 필요한 libaio1 설치 (Oracle DB 필수)
RUN apt-get update && apt-get install -y libaio1 && rm -rf /var/lib/apt/lists/*

# 빌드 스테이지에서 설치된 패키지 복사
COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin

# 소스 코드 복사
COPY . .

# --- Oracle Wallet 설정 ---
# 1. Wallet 파일을 담을 디렉토리 생성
RUN mkdir -p /app/oracle_wallet
# 2. 로컬의 Wallet 파일들을 컨테이너로 복사 (로컬의 wallet 폴더 경로 확인 필요)
COPY ./Wallet_CATSDB /app/oracle_wallet/
# 3. Oracle 관련 환경 변수 설정 (TNS_ADMIN이 Wallet 위치를 가리켜야 함)
ENV TNS_ADMIN=/app/oracle_wallet
# -------------------------

# 보안을 위한 비관리자 유저 설정
RUN useradd -m appuser && chown -R appuser /app
USER appuser

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

지난번 고생했던 AMD 아키텍처의 VM에서 빌드하고, ARM 아키텍처의 VM에서 배포하는 경우에 대해서 문제가 없도록 작성하였다.

이후 아래 명령어로 같은 상황에 문제가 없도록 도커 이미지를 빌드한다.

```bash
docker run --privileged --rm tonistiigi/binfmt --install all
docker buildx create --name arm64-builder --driver docker-container --use
docker buildx inspect --bootstrap
docker buildx build --platform linux/arm64 -t backend-web:v1.$BUILD_NUMBER --load .
docker buildx rm arm64-builder
docker images
```

## Docker 이미지 Push

잘 빌드가 완료되었다면 아래 명령어로 Container Resitry에 Push 한다.

```bash
docker login ap-chuncheon-1.ocir.io
docker tag backend-web:v1.$BUILD_NUMBER ap-chuncheon-1.ocir.io/axqyrowq4jay/crypto-prd-repo/backend-web:latest
docker push ap-chuncheon-1.ocir.io/axqyrowq4jay/crypto-prd-repo/backend-web
docker images
```

## Deployments.yaml 파일 작성 및 최초 배포

마지막으로 아래와 같이 backend에 대한 deployments.yaml 파일을 작성하고 저장한 후, 아래 명령어로 배포를 진행한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: be-web-deployment
spec:
  selector:
    matchLabels:
      app: be-web
  replicas: 2
  template:
    metadata:
      labels:
        app: be-web
    spec:
      containers:
      - name: be-web
        image: ap-chuncheon-1.ocir.io/axqyrowq4jay/crypto-prd-repo/backend-web:latest
        imagePullPolicy: Always
        ports:
        - name: fe-desktop
          containerPort: 8000
          protocol: TCP
      imagePullSecrets:
      - name: ocirsecret
---
apiVersion: v1
kind: Service
metadata:
  name: be-web-service  # 이 이름이 내부 접속 주소가 됩니다.
spec:
  selector:
    app: be-web
  type: ClusterIP      # LoadBalancer 대신 ClusterIP 사용
  ports:
  - port: 8000           # 서비스 포트
    targetPort: 8000   # 컨테이너 포트
```

```bash
kubectl apply -f ./backend-web.yaml
kubectl get pods
```

정상적으로 pod가 떠있는 것을 확인하고, frontend와 통신이 되는지 아래 명령어로 확인하였다.

```bash
kubectl get endpoints be-web-service # Endpoint 항목에 private IP가 잘 나오는지 확인
kubectl logs -f be-web-deployment-{backend-pod명} # backend pod의 로그를 실시간으로 확인하며
kubectl exec -it fe-desktop-deployment-{frontend-pod명} -- /bin/sh # FE 포드 내부 접속 후
curl -v http://be-web-service:8000/ # 요청을 보내 정상응답을 받는지 확인
```

# Jenkins CICD 설정

이제 Jenkins를 통해서 백엔드 부분도 배포할 수 있도록 Jenkins 설정으로 들어갔다.

## Jenkins Webhook 설정

Jenkins → 새로운 Item
  - name: github-be-web-webhook
  - type: Freestyle project
  - 소스코드관리: Git
  - Repositories: private repository URL
  - Credentials: github user & pw
  - branches to build: */main
  - Triggers: GitHub hook trigger for GITScm polling

이후 깃허브의 연동하려고 했던 private repository의 Settings → Webhooks에 가면 Webhook이 생성되어 있다.

![jenkins_webhook_test]({{ juyoung-hong.github.io }}/assets/images/jenkins_webhook_test.jpg)

## Jenkins 파이프라인 생성

Jenkins → 새로운 Item
  - name: backend-web-cicd
  - type: pipeline
  - GitHub project: private repository URL

```pipeline.txt
//------------------------------------------------------------------------------
// git clone -> 도커 빌드 -> Container Registry 이미지 push -> oke 배포 단계로 수행
//------------------------------------------------------------------------------

pipeline {
    agent any
    stages {
        stage('Clone Git') {
            steps {
                script{
                    sh "pwd"
                    sh "rm -rf Backend-Web"
                    sh "git clone git@github.com:CryptoAutoTradingTeam/Backend-Web.git"
                }
            }
        }
        stage('Build Container') {
            steps{
                dir('Backend-Web'){
                    sh "pwd"
                    sh "docker container ls"
                    sh "docker run --privileged --rm tonistiigi/binfmt --install all"
                    sh "docker buildx rm arm64-builder || true"
                    sh "docker buildx create --name arm64-builder --driver docker-container --use"
                    sh "docker buildx inspect --bootstrap"
                    sh "docker buildx build --platform linux/arm64 -t backend-web:v1.$BUILD_NUMBER --load ."
                    sh "docker buildx rm arm64-builder"
                    sh "docker images"
                }
            }
        }
        stage('Push to Container Registry') {
            steps {
                script {
                    sh "docker login ap-chuncheon-1.ocir.io"
                    sh "docker tag backend-web:v1.$BUILD_NUMBER ap-chuncheon-1.ocir.io/axqyrowq4jay/crypto-prd-repo/backend-web:latest"
                    sh "docker push ap-chuncheon-1.ocir.io/axqyrowq4jay/crypto-prd-repo/backend-web"
                    sh "docker images"
                }
            }
        }
        stage('Deploy OKE') {
            steps{
                dir('Backend-Web') {
                    sh "kubectl rollout restart deployment be-web-deployment"
                    sh "kubectl get pods"
                    sh "echo 'done'"
                }
            }
        }
    }
}
```

위 설정을 저장한 이후 다시 github-be-web-webhook로 돌아와서 구성 → 빌드 후 조치 → Build other projects
  - Projects to build: backend-web-cicd
  - Trigger only if build is stable 을 선택하고 저장한다. 

## 테스트

이제 CICD도 정상적으로 동작하는지 확인하기 위해서, backend-web 소스코드에 변경사항을 반영하고 main에 push하였다.

main branch에 소스코드가 반영되고 나니, 정상적으로 빌드가 시작됐고, 몇분 기다린 뒤 정상적으로 빌드까지 완료됨을 확인하였다.

![jenkins_build_test]({{ juyoung-hong.github.io }}/assets/images/jenkins_build_test.jpg)

서비스가 배포된 이후에 API를 호출하여 회원가입 테스트를 진행해보았고, 그 이후 DB에 데이터가 잘 적재되었는지 확인하였다.

아래 이미지와 같이 DB에도 데이터가 잘 적재된 것을 확인하고 테스트를 마쳤다.

![signup_test_db_result]({{ juyoung-hong.github.io }}/assets/images/signup_test_db_result.jpg)