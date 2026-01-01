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
JENKINS_HOME=jenkins nohup java -jar jenkins.war --httpPort=8080 > jenkins.log 2>&1 &
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

이때 password 대신 PAT를 사용 해야 이후에 private repo에 접근할 때 문제가 없다.

Jenkins → Jenkins 관리 → System → GitHub → Add GitHub Server 에서 아래와 같이 정상적으로 연동 되는 것을 test하고 저장한다.

![jenkins_git_test]({{ juyoung-hong.github.io }}/assets/images/jenkins_git_test.jpg)

<br>

## Jenkins Webhook 테스트

Jenkins → 새로운 Item
  - name: github-fe-desktop-webhook
  - type: Freestyle project
  - 소스코드관리: Git
  - Repositories: private repository URL
  - Credentials: github user & pw
  - branches to build: */main
  - Triggers: GitHub hook trigger for GITScm polling

이후 깃허브의 연동하려고 했던 private repository의 Settings → Webhooks에 가면 Webhook이 생성되어 있다.

<br>

# Jenkins를 통한 배포

Jenkins → 새로운 Item
  - name: frontend-desktop-cicd
  - type: pipeline
  - GitHub project: private repository URL

```Jenkinsfile
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
                    sh "rm -rf Frontend-Desktop"
                    sh "git clone git@github.com:CryptoAutoTradingTeam/Frontend-Desktop.git"
                }
            }
        }
        stage('Build Container') {
            steps{
                dir('Frontend-Desktop'){
                    sh "pwd"
                    sh "docker container ls"
                    sh "docker run --privileged --rm tonistiigi/binfmt --install all"
                    sh "docker buildx rm arm64-builder || true"
                    sh "docker buildx create --name arm64-builder --driver docker-container --use"
                    sh "docker buildx inspect --bootstrap"
                    sh "docker buildx build --platform linux/arm64 -t frontend-desktop:v1.$BUILD_NUMBER --load ."
                    sh "docker buildx rm arm64-builder"
                    sh "docker images"
                }
            }
        }
        stage('Push to Container Registry') {
            steps {
                script {
                    sh "docker login ap-chuncheon-1.ocir.io"
                    sh "docker tag frontend-desktop:v1.$BUILD_NUMBER ap-chuncheon-1.ocir.io/axqyrowq4jay/crypto-prd-repo/frontend-desktop:latest"
                    sh "docker push ap-chuncheon-1.ocir.io/axqyrowq4jay/crypto-prd-repo/frontend-desktop"
                    sh "docker images"
                }
            }
        }
        stage('Deploy OKE') {
            steps{
                dir('Frontend-Desktop') {
                    sh "kubectl rollout restart deployment fe-desktop-deployment"
                    sh "kubectl get pods"
                    sh "echo 'done'"
                }
            }
        }
    }
}
```

위 설정을 저장한 이후 다시 github-fe-desktop-webhook로 돌아와서 구성 → 빌드 후 조치 → Build other projects
  - Projects to build: frontend-desktop-cicd
  - Trigger only if build is stable 을 선택하고 저장한다. 

<br>

## CI/CD 테스트

Gemini를 통해 토스쪽 화면을 보여주면서 로그인 화면을 좀 더 멋지게 변경해달라고 했다.

이후 해당 내용을 커밋하고, main branch에 병합해보았다.

jenkins를 통해서 clone하면 username을 요구한다거나, docker 단계에서 오류가 발생하는 문제가 있었으나 최종적으로 위의 파이프라인 커맨드로 아래와 같이 성공을 완료하였다.

정상적으로 파이프라인이 성공한 뒤, 로드밸런서 쪽으로 접속하니, 성공적으로 변경된 UI가 노출되는 것까지 확인하였다.

![jenkins_build_result]({{ juyoung-hong.github.io }}/assets/images/jenkins_build_result.jpg)