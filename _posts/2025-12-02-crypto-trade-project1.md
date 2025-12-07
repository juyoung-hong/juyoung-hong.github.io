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

<br>

# 오라클 클라우드

오라클 클라우드는 개인적으로 3년전쯤 부터 사용하고 있다.

프리티어를 사용하면 회원가입만 해도 VM, DB, LB, VCN 등 다른 클라우드에서는 비용을 지불해야하는 많은 리소스들을 무료로 사용할 수 있기 때문이다.

집에 남아도는 노트북을 서버 컴퓨터로 활용해도 되지만, 이 간단한 시스템에 이것저것 덧붙여 보고 싶은 것도 많고 클라우드 관련해서도 공부하고 싶은 내용들이 많아서 오라클 클라우드를 활용하고자 한다.

<br>

## Compartment

오라클 클라우드에서는 컴파트먼트라는 편리한 기능을 지원한다.

테넌시 내에 있는 클라우드 자원을 다시 논리적인 그룹으로 나눠서 사용할 수 있고, 클라우드 자원을 묶어서 정책을 적용하는 방식으로 관리할 수 있다.

개발 컴파트먼트와 운영 컴파트 먼트를 나누어, 각  사용자별로 읽기와 쓰기 권한을 나누어 부여 하는 것도 방법이다.

<br>

### Compartment 생성

- OCI 콘솔의 Identity & Security → Identity → Compartments → Create compartment

개발용 리소스는 만들수 없을 것 같지만, 우선 컴파트먼트는 개발과 운영을 나누고자 한다.

컴파트먼트 등 클라우드 자원에 대한 네이밍 룰도 필요하겠지만 토이 프로젝트는 간단히 만들겠다.

root directory의 하위에 'crypto-dev-compartment' 와 'crypto-prd-compartment' 를 생성하였다

![create_compartment]({{ juyoung-hong.github.io }}/assets/images/create_compartment.jpg)

<br>

## 권한 관리

오라클 클라우드에서는 유저를 그룹으로 묶고, 그룹에 정책을 통해 권한을 부여한다.

권한 부여시 범위를 컴파트먼트 또는 테넌시로 제한할 수 있다.

유저와 그룹은 다시 도메인이라고 하는 더 포괄적인 개념으로 관리된다.

최초 계정 생성시 Default 도메인이 자동으로 생성되며, 추가로 도메인을 생성해서 관리할 수 있다.

<br>

### Domain 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-dev-compartment 컴파트먼트 선택 → Create Domain

Free Domain으로 crypto-dev-default-domain 을 아래와 같이 추가하였다.

Remote-DR 관련 설정은 서울 리전에서만 가능한 듯 보였는데, 나는 일단 DR 설정을 끄고 춘천 리전에서 생성하였다.

![create_domain]({{ juyoung-hong.github.io }}/assets/images/create_domain.jpg)

<br>

### Group 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-dev-compartment → crypto-dev-default-domain → User management → Group → Create group

crypto-dev-default-developer-group로 개발 환경의 개발자 그룹을 만들었고, 이 그룹에는 별도 유저는 할당하지 않았다.

![create_group]({{ juyoung-hong.github.io }}/assets/images/create_group.jpg)

<br>

### User 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-dev-compartment → crypto-dev-default-domain → Users → Create

개발자 계정을 만들기 위해서, 이름은 동일하지만 메일 주소는 다른것으로 개발자용 계정을 생성하였고, crypto-dev-default-developer-group 그룹에 해당 유저를 할당하였다.

![create_user]({{ juyoung-hong.github.io }}/assets/images/create_user.jpg)

<br>

### 권한 정책 생성

- OCI 콘솔의 Identity & Security → Identity → Policies → Create Policy

개발자들에게 dev 컴파트먼트에 대한 리소스 사용 권한을 주고자 아래와 같이 설정하였다.

- Policy 명: crypto-dev-default-developer-group-policy
- Policy (mannual):

```bash
allow group crypto-dev-default-developer-group to use all-resources in compartment crypto-dev-compartment
```

![create_policy]({{ juyoung-hong.github.io }}/assets/images/create_policy.jpg)

<br>

## 가상 클라우드 네트워크

VCN은 리전 레벨의 자원으로 여러개의 가용 도메일에 걸쳐 구성되므로, 복수의 가용 도메인을 가진 리전에서 VCN을 생성하면, 고가용성 및 이중화를 구현할 수 있다.

### 가상 클라우드 네트워크 생성

- OCI 콘솔의 Networking → Virtual Cloud Networks → Actions → Start VCN Wizard → VCN with internet connectivity

- VCN 명: crypto-dev-default-vcn

![create_vcn]({{ juyoung-hong.github.io }}/assets/images/create_vcn.jpg)

### 서브넷

VCN을 더 잘게 분할한 하위 네트워크 자원으로 각 서브넷은 다른 CIDR 블록 범위를 가지며, 겹치지 않도록 구성된다.

### 라우트 테이블

VCN에서 IP 패킷의 전달 경로를 결정하는데 사용되는 자원으로 통신이 원활하려면 서브넷에 적절한 라우트 규칙을 설정해야 한다.

### 시큐리티 리스트

VCN 내에서 Ingress 및 Egress 트래픽에 대한 보안 규칙을 정의하는 자원이며, 같은 서브넷 안에 속한 모든 인스턴스는 동일한 보안 규칙을 적용받는다.

- Stateful 규칙: 수신 규칙과 관계 없이 트래픽이 원래 호스트로 돌아갈 수 있음 (요청을 보낸 호스트로 응답이 돌아갈 수 있도록 허용하는 경우)
- Stateless 규칙: 트래픽의 수신과 송신간의 관련성이 없어 응답을 추적하지 않는 경우 사용 (특정 포트로 들어오는 트래픽에 대해서 응답을 차단하고 싶은 경우)

### 인터넷 게이트웨이

VCN 내의 인스턴스가 인터넷과 통신하는 입구 역할을 하며, 이를 위해서는 퍼블릭 서브넷 내의 자원들이 public IP를 가져야 한다.

### NAT 게이트웨이

public IP 주소가 없는 클라우드 자원이 인터넷으로 나가는 접근을 가능하게 하는 가상 라우터이다. 

private 서브넷 내의 자원들이 인터넷 접속이 필요할때 사용한다.

### 서비스 게이트웨이

OCI 내의 다양한 오라클 클라우드 서비스에 대한 private 접근을 허용하는 게이트 웨이이다.

데이터를 인터넷에 노출시키지 않고 VCN내의 자원들이 오라클 클라우드 서비스에 안전하게 비공개 접근하게 한다.

### Dynamic Routing Gateway

여러 리전의 VCN 또는 온프레미스 네트워크를 private 네트워크로 연결할 때 사용하는 가상 라우터이다.

## 가상 머신

- 이미지: 인스턴스의 초기 상태 (운영체제 + 소프트웨어)
- Shape: 서비스에 할당되는 자원의 단위
- 블록 볼륨: 가상 디스크로 필요에 따라 분리 및 연결이 가능
- 부트 볼륨: 컴퓨트 인스턴스의 부팅 이미지를 저장하는 블록 스토리지

### SSH 키 페어 생성

1. 로컬 컴퓨터에서 ssh key 생성 (.ssh 하위의 oci_crypto_dev로 생성함)

```zsh
ssh-keygen -t rsa -b 2048 -f /Users/jyhong/.ssh/oci_crypto_dev
```

2. OCI 콘솔의 Compute → Instances → Create Instance

- instance name: crypto-dev-web-ap01
- Image: Oracle Linux9
- Shape: VM.Standard.E2.1.Micro
- Primary VNIC: 위에서 생성한 VCN 및 public subnet 선택 후 자동 할당
- SSH: 위에서 만든 ssh pub키 업로드
- 부트볼륨: 자동 선택

3. SSH 접속

VS 코드의 remote 관련 config에 아래 내용을 저장하고 원격으로 접속한다.

```yaml
Host crypto-dev-web-ap01
  HostName IP주소
  User opc
  IdentityFile /Users/jyhong/.ssh/oci_crypto_dev
```

아래 이미지와 같이 정상적으로 접속이 가능함을 확인했다.

![connect_vm]({{ juyoung-hong.github.io }}/assets/images/connect_vm.jpg)

## 결과

최종적으로 아래 이미지와 같은 구조로 클라우드 리소스 생성을 완료했다.

![oci_20251207]({{ juyoung-hong.github.io }}/assets/images/oci_20251207.jpg)
