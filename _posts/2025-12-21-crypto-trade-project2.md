---
title: "비트코인 자동매매 프로젝트2 - Jenkins CI-CD 구성"
classes: wide
categories:
  - cryptotrade
tags:
  - project
  - cryptotrade
  - ci-cd
---

<br>

# 개요

지난 글에서는 구성된 OKE 클러스터에 내가 개발하고자 하는 내용을 컨테이너 이미지로 만들어서 배포했다.

이 과정을 각각 서버에 들어가서 이미지를 빌드하고, push하고, 배포 하는 과정을 직접 했었는데 이런 CI/CD 과정을 자동화 하고자 한다.

<br>

# CI/CD 파이프라인

일반적으로는 아래와 같은 파이프라인으로 CI/CD가 구성된다.

<br>

## 지속적 통합 (Continuous Integration)

- 빌드: 코드 작성 후 Git에 커밋하면 자동으로 빌드가 실행되며, 소스 코드 컴파일 및 라이브러리를 가져와서 실행가능한 애플리케이션을 생성한다.
- 테스트: 빌드된 애플리케이션에 대해 자동화된 테스트를 실행해서 버그를 찾고, 기능적 무결성을 확인한다.
- 통합: 빌드와 테스트를 통과하면 개발브랜치에서 메인 브랜치로의 통합이 자동으로 수행된다.

<br>

## 지속적 전달 (Continuous Delivery)

- 릴리스: 애플리케이션을 리포지터리에 제공한다.

<br>

## 지속적 배포 (Continuous Deployment)

- 배포: 코드를 운영환경에 배포하며, 이 단계에서 무중단 배포, 롤백 등의 기능을 구현할 수 있다.

<br>

# Jenkins 구성

젠킨스는 구축, 테스트, 배포와 관련된 거의 모든 종류의 작업을 자동화하는데 사용할 수 있으므로 젠킨스를 사용하고자 하며, CI/CD 용도의 머신이 따로 있으면 좋겠지만 한정된 리소스 문제로 우선 bastion을 CI/CD에 사용하고자 한다.

<br>

## JDK 설치

젠킨스는 자바 기반도구이므로 먼저 OpenJDK를 설치해야 한다.

```bash
sudo yum install java-17-openjdk java-17-openjdk-devel -y
java -version
```

이후 war 파일로 설치하는 방법에 따라 아래 명령어를 수행하였다.

```bash
wget https://get.jenkins.io/war-stable/2.528.3/jenkins.war
JENKINS_HOME=jenkins java -jar jenkins.war --httpPort=8080 &
```

이후에 bastion 8080 포트로 접속할 수 있도록 OCI의 security list에 8080 포트를 추가하고 아래 명령어로 OS 방화벽도 해제한다.

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

이후에 http://<bastion public IP>:8080 으로 접속하고, 아래 그림과 같은 이미지가 뜨면 터미널에 있는 최초 비밀번호로 로그인한다.

![jenkins_initial]({{ juyoung-hong.github.io }}/assets/images/jenkins_initial.jpg)

<br>

## Jenkins 설치

이후 옵션 중 Install suggested plugins를 클릭해서 설치를 진행하였다.

Admin 계정을 하나 생성한 뒤 URI는 건드리지 않고 설정을 마무리 하였다.

<br>

# Git Hub 설정

bastion은 성능이 그렇게 좋지 않아, 개발은 평소에 노트북으로 하고자 한다.

Git Hub와 SSH 연결이 되도록 ssh 퍼블릭 키를 아래 명령으로 생성하고 복사 했다.

```bash
ssh-keygen -t ed25519 -C <이메일주소>
cat ~/.ssh/id_ed25519.pub
```

이후 Github에 접속해서, Settings → SSH and GPG keys → SSH keys → New SSH key 에 복사한 public key를 붙여 넣는다.

![git_ssh_key]({{ juyoung-hong.github.io }}/assets/images/git_ssh_key.jpg)

<br>

이후에 개발환경 SSH config 파일에 깃허브 호스트 정보를 등록한다.

```bash
vi ~/.ssh/config
```

```text
Host github.com
  IdentityFile ~/.ssh/id_ed25519
  User git
```

이후 접속을 시도해 본다.

```bash
ssh -T git@github.com
```

"Hi juyoung-hong! You've successfully authenticated, but GitHub does not provide shell access." 문구와 함께 정상적으로 통신이 가능함을 확인하였다.

<br>

## Github와 Jenkins 연동

깃허브와 젠킨스의 연동은 깃허브에서 토큰을 만들고 젠킨스에 크리덴셜로 등록하는 방식으로 진행한다.

Github → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic)

이제 젠킨스에 크리덴셜을 만든다.

Jenkins → Jenkins 관리 → Credentials → Add credentials → New Credentials에 추가한다.
  - Kind: Secret text
  - Scope: Global
  - Secret: 깃허브 토큰
  - ID: GITHUBTOKEN

한번더 Username with password로 생성해서 github 계정 정보를 저장한다.