---
title: "오라클 클라우드로 무료 인프라 네트워크 구성하기"
classes: wide
categories:
  - infra
tags:
  - oracle-cloud
  - network
---

<br>

# 오라클 클라우드로 무료 인프라 네트워크 구성하기

<br>

오라클 클라우드는 개인에게도 상당히 많은 양의 리소스를 제공합니다.

클라우드 인프라에 대해서 학습하거나, 개인용 서버를 운영하기에 적합한 환경이므로 오라클 클라우드의 OKE를 구성하여 무료 관리형 쿠버네티스 인프라를 구성해보겠습니다.

<br>

## 오라클 클라우드 회원가입

<br>

![ocisignin]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_signin_20260329.jpeg)

가장 먼저 오라클 클라우드 홈페이지에서 회원가입을 진행합니다.

[오라클 클라우드 홈페이지](https://www.oracle.com/kr/cloud/)에 접속한 뒤, 화면 우측 상단의 사람 버튼을 클릭한 뒤 나오는 화면에서 "무료 클라우드 티어 가입" 버튼을 클릭하여 회원가입을 진행합니다.

회원가입시 리전을 선택해야 한다면, 물리적으로 위치가 가까운 서울, 춘천을 선택하거나 근처 일본까지 선택하시기를 추천드립니다. 

( 서울은 더이상 가상머신 생성 등이 불가하며, 춘천도 거의 가득찬 상태입니다. 오사카로 조금 멀리 선택하는 것이 가상머신을 만들때 조금 스트레스를 덜 받을 수 있습니다. )

<br>

### 리전

<br>

OCI에서 리전은 클라우드 서비스를 제공하는 물리적인 인프라의 위치입니다. (서울, 춘천, 도쿄, 오사카 등) 

당연히 물리적으로 거리가 가까울수록 네트워크 속도 지연이 발생할 가능성이 적기 때문에 서버에 접속할 사용자들과 물리적으로 가까운 리전을 선택하는 것이 좋습니다.

<br>

## 컴파트먼트 생성

<br>

회원가입을 잘 마쳤다면, 클라우드 서비스에 로그인하고 가장 먼저 컴파트먼트를 생성합니다.

<br>

### 테넌시

<br>

테넌시는 오라클 클라우드 계정에 할당되는 클라우드 자원 관리 공간으로, 주어진 테넌시 안에서 각종 클라우드 자원을 만들고 관리합니다.

보통 회사가 테넌시의 단위가 되며, 테넌시 내에서 사용자들을 만들고, 그룹화 하여 각 그룹에 대해 클라우드 자원을 관리할 수 있는 정책을 부여하여 사용합니다.

<br>

### 도메인

<br>

처음 클라우드 계정을 생성하게 되면, default 도메인이 자동으로 생성됩니다.

도메인은 유저와 그룹을 효과적으로 관리하기 위해, 업무 요구에 따라 추가적으로 만들어서 관리할 수 있습니다. 이번 실습에서는 실 사용자가 저밖에 없으니, 도메인은 별도로 만들지 않고, default 도메인을 사용하도록 하겠습니다.

<br>

### 컴파트먼트

<br>

컴파트먼트는 오라클 클라우드의 기능 중 하나로, 테넌시 내에서 클라우드 자원을 논리적인 그룹으로 나눌 수 있는 기능입니다.

하나의 테넌시 내에서 업무 구분 또는 조직 단위로 나누어 클라우드 자원을 관리하고자 하는 경우 컴파트먼트로 클라우드 자원을 나누어 정책으로 관리할 수 있고, 컴파트먼트는 계층 구조를 가질 수 있습니다.

예를 들어, 개발조직용 컴파트먼트와 운영조직용 컴파트먼트를 나누어 클라우드 자원을 관리할 수도 있습니다.

<br>

### 컴파트먼트 생성

<br>

![ocicompartmentsmenu]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_compartments_menu.jpeg)

로그인 이후 나타나는 화면에서 좌측 상단의 햄버거 버튼을 클릭하여 전체 메뉴를 확인합니다.

전체 메뉴에서 Identity & Security 메뉴를 클릭하면, Identity 관련 메뉴 하단의 Compartments 메뉴를 발견할 수 있습니다. 해당 메뉴를 클릭하여 이동합니다.

![ocicompartmentsmain]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_compartments_main.jpeg)

compartments 메뉴에 기본적으로 가상 상위 컴파트먼트 개념인 root 컴파트먼트가 만들어져 있습니다.

root 컴파트먼트는 반드시 삭제하면 안되고, 추가로 사용할 컴파트먼트를 만들겠습니다. Create compartment 화면을 클릭합니다.

![ocicompartmentscreate]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_compartments_create.jpeg)

1. 생성할 컴파트먼트의 이름을 작성합니다. 별도의 명명규칙이 있으면 좋을텐데 저는 "{dev/prd}_{application}_compartment"로 사용하겠습니다.
2. Description에는 컴파트먼트에 대한 설명을 작성하였습니다.
3. Parent comparment는 최상위 컴파트먼트였던 root 컴파트먼트를 선택 후 Create compartment 버튼을 클릭하여 생성합니다.

잠시 뒤, compartments 메뉴에서 새로고침을 진행하면 새로운 컴파트먼트가 생성되었음을 확인할 수 있습니다.

앞으로 생성할 ML Platform 관련 리소스들은 해당 컴파트먼트 하위에 생성하겠습니다.

![ocicompartmentsmain2]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_compartments_main2.jpeg)

이제 생성된 컴파트먼트에 유저를 등록하고, 유저 그룹을 만들고 정책을 적용할 수 있는 준비가 되었습니다.

그러나 현재 저는 혼자서 클라우드 자원 관리를 진행하고자 하니 일단 해당 과정은 생략하고 리소스를 만들어보도록 하겠습니다.

<br>

## 가상 클라우드 네트워크 구성

<br>

다음으로 진행할 일은 클라우드 자원들이 배치되어 서로 통신할 수 있도록 가상 클라우드 네트워크 (VCN)을 만드는 일입니다.

<br>

### 가상 클라우드 네트워크 (VCN)

<br>

가상 클라우드 네트워크 (Virtual Cloud Network)는 전통적인 물리 네트워크의 가상화된 버전입니다.

소프트웨어 정의 네트워크 (Software Defined Network)로 구현되며, 서브넷, 라우트 테이블, 게이트웨이 등의 네트워크 장치들도 소프트웨어적으로 정의된 형태로 구성됩니다.

사용자는 오라클 클라우드 데이터센터 내에서 사설 네트워크를 VCN 위에 구성할 수 있습니다.

<br>

### VCN 생성

<br>

![ocivcnmenu]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_vcn_menu.jpeg)

오라클 클라우드 전체 메뉴로 이동하여, 이번에는 Networking 탭에 있는 Virtual Cloud Networks 메뉴로 이동합니다.

![ocivcnmain]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_vcn_main.jpeg)

기본적으로 root compartment 하위의 리소스가 나타날텐데, Compartment 쪽을 클릭하여 아까 만든 컴파트먼트가 선택되도록합니다.

컴파트먼트가 잘 선택되었다면 Actions 버튼을 클릭하여 Start VCN Wizard를 클릭합니다.

![ocivcncreate]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_vcn_create.jpeg)

온프레미스 네트워크와 VPN 연결은 필요 없으므로 "Create VCN with Internet Connectivity"를 클릭하고 "Start VCN Wizard" 버튼을 클릭합니다.

![ocivcncreate2]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_vcn_create2.jpeg)

1. VCN의 이름을 정하는데, 저는 "{dev/prd}_{application}_VCN" 으로 생성하겠습니다.
2. Compartment는 위에서 만든 컴파트먼트를 선택해줍니다.
3. VCN IPv4 CIDR block은 원하는 사설망 대역이 있다면 변경 가능하지만, 저는 기본 대역으로 생성을 진행하겠습니다.
4. 이외에도 고급 설정을 적용할 수 있지만 저는 Next 버튼을 클릭하여 생성하려 합니다.

![ocivcncreate3]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_vcn_create3.jpeg)

마지막으로 원하는 구성이 맞는지 확인하고, 우측 하단의 Create 버튼을 클릭하여 VCN을 구성합니다.

![ocivcncreate4]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_vcn_create4.jpeg)

조금 기다리면 VCN 생성이 완료되고 우측 하단의 "View VCN" 을 클릭하면 생성된 VCN을 확인할 수 있습니다.

<br>

## Subnet 수정

<br>

![ocivcnsubnetedit]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_subnet_edit.jpeg)

생략해도 무방한 절차이지만, 기본적으로 구성된 서브넷과 관련 리소스들의 명명 규칙을 변경하고자 합니다.

![ocivcnsubnetedit2]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/oci_subnet_edit2.jpeg)

Public Subnet은 Load balancer와 bastion 장비 등을 둘 수 있도록 DMZ subnet으로 명명하겠습니다.

![mlplatformnetwork]({{ juyoung-hong.github.io }}/assets/images/2026-03-29-ocinetwork/mlpatform_network.jpeg)

최종적으로 위 이미지와 같은 네트워크가 구성되었습니다.

다음으로는 위 네트워크를 좀 더 수정한 뒤, OKE 클러스터를 배포해 보겠습니다.