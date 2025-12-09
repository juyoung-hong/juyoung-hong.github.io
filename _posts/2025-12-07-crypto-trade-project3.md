---
title: "비트코인 자동매매 프로젝트3 - 운영 환경 인프라 구성 (OCI)"
classes: wide
categories:
  - cryptotrade
tags:
  - project
  - cryptotrade
  - cloud
---

<br>

# 운영 환경 인프라 구성

일단 돌아가는 코드를 만들어 놓고, 개발 환경 VM에 올리려고 하니 git을 설치하다가 VM이 죽어버렸다.

겸사 겸사 지난번 개발 환경 네트워크를 구성할 때 아쉬웠던 부분들을 포함해서 운영환경의 인프라를 다시 구성해보려 한다.

<br>

## Compartment 생성

- OCI 콘솔의 Identity & Security → Identity → Compartments → Create compartment

root directory의 하위에 crypto-prd-compartment 를 생성하였다

<br>

## Domain 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-dev-compartment 컴파트먼트 선택 → Create Domain

Free Domain으로 crypto-prd-default-domain 을 추가하였다.

Remote-DR 설정을 끄고 춘천 리전에서 생성하였다.

Domain 생성 중 maximum domain limit에 걸려서 기존에 만든 crypto-dev-default-domain을 삭제 처리했다.

나머지 그룹, 유저, 권한 생성은 나중으로 건너뛰고 VCN을 생성하는 곳으로 넘어갔다.

<br>

## VCN 생성

- OCI 콘솔의 Networking → Virtual Cloud Networks → Actions → Start VCN Wizard → VCN with internet connectivity

- VCN 명: crypto-prd-default-vcn

<br>

### 서브넷 구성

컨셉을 좀더 명확히 알 수 있도록 기본적으로 구성되는 2개 subnet의 이름을 변경하였다.

- (기존) public subnet-crypto-prd-default-vcn → (변경) crypto-prd-dmz-subnet
- (기존) private subnet-crypto-prd-default-vcn → (변경) crypto-prd-web-subnet

<br>

#### 서브넷 추가 생성

마찬가지로 의미를 명확히 하기 위해서 was, db용 private subnet을 추가 구성하였다.

- OCI 콘솔의 Networking → Virtual Cloud Networks → crypto-prd-default-vcn → Subnets → Create Subnet

- name: crypto-prd-was-subnet
- name: crypto-prd-db-subnet

운영 환경이므로 IP 대역은 따로 표기하지 않았지만, 겹치지 않게 잘 구성하였다.

<br>

## 가상 머신 삭제

무료티어에서는 가상머신, 블록스토리지 등 리소스 제한이 있으므로 기존에 생성했던 crypto-dev-web-ap01 머신을 삭제하였다.

<br>

## 가상 머신 생성

- OCI 콘솔의 Compute → Instances → Create Instance

아래의 VM을 각각 1 OCPU, 6GB RAM 으로 ARM머신으로 Oracle Linux 9버전을 사용하여 구성하였다.

- crypto-prd-web-ap01

<br>

## Bastion 생성

OCI에서는 별도의 Bastion 가상 머신을 구성하지 않고도 Bastion 서비스를 통해 private 서브넷에 접근할 수 있다.

- OCI 콘솔의 Identity & Security → Bastion → Create bastion
  - bastion name: cryptoprdwebbastion
  - target subnet 지정 및 CIDR block allow list 작성 (우리집 공인아이피/32)

![create_bastion]({{ juyoung-hong.github.io }}/assets/images/create_bastion.jpg)

<br>

### Bastion 세션 추가

- OCI 콘솔의 Identity & Security → Bastion → cryptoprdwebbastion → Sessions → Create Session
  - Session Type: SSH port forwarding session
  - Session Name: crypto-prd-web-ap01-session
  - target으로 crypto-prd-web-ap01 선택

![create_bastion_session]({{ juyoung-hong.github.io }}/assets/images/create_bastion_session.jpg)

<br>

### SSH 연결

- OCI 콘솔의 Identity & Security → Bastion → cryptoprdwebbastion → Sessions → crypto-prd-web-ap01-session → Copy SSH Command
  - 복사된 command에서 privateKey 경로와 localPort 변경 (ex.2222)
  - 완성된 command 터미널 입력 후 별도의 터미널에서 ssh 접속

#### vscode - remote 설정 값

```yaml
Host crypto-prd-web-ap01
  HostName localhost
  Port <localPort>
  User opc
  IdentityFile <privateKey 경로>
```

## 어플리케이션 배포

### Git 설치

오라클 리눅스에서는 아래의 명령어로 git을 설치한다.

```bash
sudo dnf install git -y
git --version
```

성공적으로 설치된 이후에는 설정을 진행했다.

```bash
git config --global user.name "Juyoung Hong"
git config --global user.email "hjy_stat@naver.com"
```

이후 application을 사용할 appuser 계정을 생성하였고, appuser 계정에 sudoer 권한을 추가하였다. 이후 어플리케이션 설정은 appuser로 진행하였다.

```bash
sudo useradd appuser
sudo passwd appuser
sudo visudo
su appuser
```

배포용 서버를 구성하고 git repo를 클론해 왔다.

```bash
sudo mkdir /srv/website
cd /srv/website
sudo git clone https://github.com/CryptoAutoTradingTeam/CryptoAutoTradeSystem.git
```

### npm 설치

이후 서비스를 실행하기 위해서는 npm이 필요했다.
오라클 리눅스에서는 아래의 명령어로 npm을 설치한다.

```bash
sudo dnf config-manager --set-enabled ol9_appstream
sudo dnf install nodejs
sudo dnf module enable nodejs:20
sudo dnf update nodejs
```

### build 및 배포

빌드를 위해 front 쪽 소스코드로 이동하여, dependency 설치 및 빌드를 진행한다.

```bash
cd CryptoAutoTradeSystem/frontend
sudo npm install
sudo npm run build
```

이후 배포를 위해 nginx를 설치해준다.

```bash
sudo dnf install -y nginx
sudo systemctl enable --now nginx.service
systemctl status nginx.service
```

이후 아래 명령어로 OS 방화벽을 해제하고 테스트 해본다.

```bash
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
curl http://$(hostname -i)
```

이후 배포하고자하는 build 파일에 대해 권한을 설정한다.

```bash
sudo chown -R nginx:nginx /srv/website/CryptoAutoTradeSystem/frontend/dist
sudo chcon -Rt httpd_sys_content_t /srv/website/CryptoAutoTradeSystem/frontend/dist
```

/etc/nginx/conf.d/default.conf 파일을 열고 아래와 같이 작성 후 저장한다.

```text
server {
  server_name   $(hostname -i) $IP;
  root           /srv/website/CryptoAutoTradeSystem/frontend/dist;
  index          index.html;
}
```

마지막으로 변경된 conf 파일로 다시 nginx를 기동하고 index 페이지가 잘 나오는지 확인하였다.

```bash
sudo systemctl restart nginx
curl http://$(hostname -i)
```

## VM security list 설정 및 테스트

- OCI 콘솔의 Networking → Virtual Cloud Networks → crypto-prd-default-vcn → Security → Create Security List
  - Name: crypto-prd-web-securitylist
  - Contents: 0.0.0.0/0 에 대해 TCP 통신 80 Port Ingress rule 추가

- OCI 콘솔의 crypto-prd-default-vcn → Subnets → crypto-prd-web-subnet → Security → Add Security List → crypto-prd-web-securitylist 선택 후 저장

## Custom Image 생성

