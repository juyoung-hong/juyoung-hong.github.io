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

</br>

# 무료 도메인 구매

SSL 설정을 하기 위해 먼저 도메인이 필요했다.

클라우드 서비스와 마찬가지로 도메인에 비용을 지불할 수 없으니, 무료 도메인 제공하는 사이트를 찾아보았고, [내도메인.한국](https://xn--220b31d95hq8o.xn--3e0b707e/) 이라는 사이트를 발견하게 되었다.

.한국 또는 .kr 도메인을 무료로 제공한다고 해서 .kr 도메인을 잡아보려고 했다.

# Cloudflare

Cloudflare는 무료로 DNS, CDN, WAF, SSL 서비스를 제공한다.

마찬가지로 비용적인 이슈로 문제가 있는 나에게는 꼭 필요한 서비스이므로 https 서비스를 구현하는데 있어, Cloud flare를 사용하기로 하였다.

마찬가지로 회원가입을 먼저하고, 로그인을 해야하는데, 나는 구글 계정을 사용하였다.

로그인 후 이동한 홈화면에서 register new domain을 클릭하고, 이동한 페이지에서 위에서 취득한 도메인을 입력하였다.

원래는 무료 도메인을 따로 발급받고 Cloudflare에 연결하고자 하였으나, freenom은 거의 모든 도메인이 이미 사용중이었고, .kr로 끝나는 도메인은 cloudflare에서 지원하지 않는 이슈로 그냥 저렴한 도메인을 cloudflare에서 검색하여 구매하였다.

이후 구매한 domain에 대해 DNS 관리 페이지로 이동하였고, 아래 이미지와 같이 LB의 IP 주소를 A 레코드로 등록하였다.

![cloudflare-dns]({{ juyoung-hong.github.io }}/assets/images/cloudflare-dns.jpg)

## SSL 인증서 생성

이후 SSL/TLS 메뉴로 이동하여, Origin Certificate를 생성하였다.

RSA (2048) 옵션으로 대부분 옵션은 default로 설정하여 PEM 키를 발급받고, Origin Certificate 값과 Private Key는 따로 보관해 두었다.

## SSL 인증서 설치

이후 Cloudflare에서 발급받은 SSL 인증서를 OCI의 loadbalancer에 적용하고자 하였다.

오라클 클라우드 콘솔에 로그인하고 지금 배포되어있는 loadbalancer를 찾아 들어갔다.

이후 Certificates and ciphers 메뉴에 접근하여 하단의 Load balancer managed certificates 메뉴에서 Add certificate를 클릭한 후, 위에서 복사한 키 들을 붙여넣어 주었다.

![loadbalancer_ssl]({{ juyoung-hong.github.io }}/assets/images/loadbalancer_ssl.jpg)

## Listener 설정

이후 로드밸런서의 Listener 탭에 들어가서 443 포트로 리스닝하고 있는 리스너를 편집하여, HTTPS로 통신하게끔 하고 SSL 옵션을 켜고 방금 등록한 인증서를 선택해주었다.

이후에 구입한 도메인으로 접속을 시도하니 정상적으로 접속이 가능했다.

![loadbalancer_listener]({{ juyoung-hong.github.io }}/assets/images/loadbalancer_listener.jpg)

