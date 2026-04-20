---
title: "간단한 React 앱 구성 후 OKE에 배포하기"
classes: wide
categories:
  - devops
tags:
  - github
  - organization
  - react
---

<br>

# Github Organization 구성하기

<br>

오늘은 지난 글에서 만들었던 OKE에 배포할 간단한 어플리케이션을 만들어 보겠습니다.

Github는 무료로 git에 대한 원격 저장소를 제공하며, 특히 Organization 기능을 활용하면 여러개의 Repo들을 하나로 묶어서 관리하는 등 좀 더 효율적으로 협업할 수 있습니다.

이번에는 무료로 사용할 수 있는 github organization을 생성하여 소스코드 형상 관리 시스템을 세팅해보겠습니다.

<br>

## 새로운 Organization 생성하기

<br>

![new_organization]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/new_organization.jpeg)

github 계정으로 로그인 한 뒤 화면 우측 상단의 "+" 버튼을 클릭하고, "New organization"을 클릭합니다.

![new_organization2]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/new_organization2.jpeg)

무료로 사용할 수 있도록 왼쪽의 "Create a free organization"을 클릭합니다.

![new_organization3]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/new_organization3.jpeg)

Organization Name을 기존의 다른 조직과 겹치지 않도록 입력하고, contact email 주소를 입력한 뒤, "My personnel account"로 선택 후 아래의 "로봇이 아닙니다"를 풀고 약관에 동의하면 Organization을 생성할 수 있습니다.

![new_organization4]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/new_organization4.jpeg)

초대할 동료가 있다면 초대하고, Complete setup 버튼을 클릭하면 Organization이 생성됩니다.

<br>

## 새로운 Repo 생성하기

<br>

![new_repo]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/new_repo.jpeg)

Organization이 생성되면 Repositories 탭을 클릭한 후 화면에서 오른쪽 상단의 "New repository" 버튼을 클릭하여 Repo를 생성합니다.

![new_repo2]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/new_repo2.jpeg)

이후로는 일반적으로 github에서 repo를 만드는 것과 동일하게 repository에 사용할 이름 등을 입력하고 Create Repository를 클릭하여 생성합니다.

![clone_repo]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/clone_repo.jpeg)

```zsh
git clone {repository url}
```

repository 생성 완료후에는 생성된 repository에서 작업할 수 있도록 터미널을 열어, 위 명령어로 local에 repo를 클론해오고 IDE로 작업할 폴더를 열어서 준비합니다.

![open_repo]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/open_repo.jpeg)

저는 IDE로 google의 anti-gravity를 사용하겠습니다.

<br>

## React project 세팅

<br>

프론트엔드쪽 관련해서는 잘 모르지만 react 관련 앱들이 대부분 주류로 사용되는 것 같습니다.

올해 코딩애플님의 react 강의를 들었었고, 강의 내용과 같이 기본적인 프로젝트 세팅을 진행해 보겠습니다.

제 PC에는 node.js 및 npm이 설치되어 있으므로 아래 명령어로 Vite 프로젝트를 시작해 보겠습니다.

![create_vite_project]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/create_vite_project.jpeg)

```zsh
npm create vite@latest .
```

IDE로 작업 폴더를 열고, Node.js가 설치되어있다면 위 명령어로 현재 폴더에 react + typescript 개발 환경을 구성합니다.

![git_commit]({{ juyoung-hong.github.io }}/assets/images/2026-04-20-github_organization/git_commit.jpeg)

```zsh
git add *
git commit -m "Initial Commit"
git push origin
```

위 명령어로 지금까지 구성된 내용을 커밋하고 remote repository에 반영합니다.

