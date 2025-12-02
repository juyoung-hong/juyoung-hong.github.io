---
title: "비트코인 자동매매 프로젝트1 - 인프라 구성 (오라클 클라우드)"
classes: wide
categories:
  - cryptotrade
tags:
  - project
  - cryptotrade
  - cloud
---

<br>

# 개요

취업을 하기 전 나는 데이터 분석, 머신러닝 용도로 python을 주로 활용했다.

그 과정에서 클라우드, DB, 크롤링 등 배우고 시도해보긴 했지만 깊이 알지는 못하는 상태에서 회사에 입사하게 되었다.

회사에서는 이때까지 내가 공부해왔던 기술보다는 웹개발 쪽 관련된 일들과 인프라 관련된 일들이 매우 많았다.

여러가지 부족함을 메꾸기 위해 공부를 해보는게 좋을 것 같다고 생각했고, 공부한 내용은 역시 직접 해봐야 더 감이 잘 오는 것 같다.

어떤 주제로 실습을 해볼까 고민하던 찰나에 마침 사용중인 python auto trading 코드를 더 깨끗한 구조와 최신 기술스택으로 웹 시스템으로 바꿔보고자 한다.

다른 사람들의 의견대로 한번에 대단한걸 만들수는 없으니, 가장 간단한 구조부터 차근차근 만들어 나가보고자 한다.

# 오라클 클라우드

오라클 클라우드는 개인적으로 3년전쯤 부터 사용하고 있다.

프리티어를 사용하면 회원가입만 해도 VM, DB, LB, VCN 등 다른 클라우드에서는 비용을 지불해야하는 많은 리소스들을 무료로 사용할 수 있기 때문이다.

집에 남아도는 노트북을 서버 컴퓨터로 활용해도 되지만, 이 간단한 시스템에 이것저것 덧붙여 보고 싶은 것도 많고 클라우드 관련해서도 공부하고 싶은 내용들이 많아서 오라클 클라우드를 활용하고자 한다.

## Compartment

오라클 클라우드에서는 컴파트먼트라는 편리한 기능을 지원한다.

테넌시 내에 있는 클라우드 자원을 다시 논리적인 그룹으로 나눠서 사용할 수 있고, 클라우드 자원을 묶어서 정책을 적용하는 방식으로 관리할 수 있다.

개발 컴파트먼트와 운영 컴파트 먼트를 나누어, 각  사용자별로 읽기와 쓰기 권한을 나누어 부여 하는 것도 방법이다.

### Compartment 생성

- OCI 콘솔의 Identity & Security → Identity → Compartments → Create compartment

개발용 리소스는 만들수 없을 것 같지만, 우선 컴파트먼트는 개발과 운영을 나누고자 한다.

컴파트먼트 등 클라우드 자원에 대한 네이밍 룰도 필요하겠지만 토이 프로젝트는 간단히 만들겠다.

root directory의 하위에 'crypto-dev-compartment' 와 'crypto-prd-compartment' 를 생성하였다

![create_compartment]({{ juyoung-hong.github.io }}/assets/images/create_compartment.jpg)

## 권한 관리

오라클 클라우드에서는 유저를 그룹으로 묶고, 그룹에 정책을 통해 권한을 부여한다.

권한 부여시 범위를 컴파트먼트 또는 테넌시로 제한할 수 있다.

유저와 그룹은 다시 도메인이라고 하는 더 포괄적인 개념으로 관리된다.

최초 계정 생성시 Default 도메인이 자동으로 생성되며, 추가로 도메인을 생성해서 관리할 수 있다.

### Domain 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-dev-compartment 컴파트먼트 선택 → Create Domain

Free Domain으로 crypto-dev-default-domain 을 아래와 같이 추가하였다.

Remote-DR 관련 설정은 서울 리전에서만 가능한 듯 보였는데, 나는 일단 DR 설정을 끄고 춘천 리전에서 생성하였다.

![create_domain]({{ juyoung-hong.github.io }}/assets/images/create_domain.jpg)

### Group 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-dev-compartment → crypto-dev-default-domain → User management → Group → Create group

crypto-dev-default-developer-group로 개발 환경의 개발자 그룹을 만들었고, 이 그룹에는 별도 유저는 할당하지 않았다.

![create_group]({{ juyoung-hong.github.io }}/assets/images/create_group.jpg)

### User 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-dev-compartment → crypto-dev-default-domain → Users → Create

개발자 계정을 만들기 위해서, 이름은 동일하지만 메일 주소는 다른것으로 개발자용 계정을 생성하였고, crypto-dev-default-developer-group 그룹에 해당 유저를 할당하였다.

![create_user]({{ juyoung-hong.github.io }}/assets/images/create_user.jpg)

### 권한 정책 생성

