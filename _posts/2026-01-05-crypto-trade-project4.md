---
title: "비트코인 자동매매 프로젝트4 - front & back 연계"
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

지난 글에서는 backend 서비스 (인증, 인가) 기능을 만들고, DB, CI/CD를 붙였다.

이제 기존에 만들어둔 frontend 서비스에 backend 서비스를 붙여서 로그인 기능을 동작하게 하려고 한다.

<br>

# 로그인 기능 구현

먼저 Gemini를 사용해서, 로그인이 성공하면 이동할 대시보드 페이지를 구현했다.

왼쪽 상단에는 로고와 텍스트틀 넣고, 우측 상단에는 "홈" 메뉴와 로그아웃 버튼을 구현했다.

로그아웃을 하게 되면, 최초 접속했던 로그인 화면으로 이동하게끔 하였다.

우측 탭은 평소에는 접혀있고, 누르면 펼쳐지게끔 하고 "내 자산" 메뉴와 "거래소 등록" 메뉴 두개만 만들어 두었다.

기능은 없고 이런 UI만 구현해놓은채로 지난번 구현했던 backend 서비스의 log-in API와 연동하였다.

local에서 docker로 backend를 띄워둔채로 UI 테스트를 몇차례 진행하고, main에 병합하여 배포하였다.

![dashboard_ui]({{ juyoung-hong.github.io }}/assets/images/dashboard_ui.jpg)

