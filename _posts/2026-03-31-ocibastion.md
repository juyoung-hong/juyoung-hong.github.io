---
title: "오라클 클라우드에 무료 bastion 서버 구성하기"
classes: wide
categories:
  - infra
tags:
  - oracle-cloud
  - bastion
---

<br>

# 오라클 클라우드에 무료 bastion 서버 구성하기

<br>

오늘은 지난 글에 이어서 오라클 클라우드에 배치할 클라우드 리소스들과 통신하기 위한 bastion 서버를 구성해보겠습니다.

<br>

## Bastion

<br>

Bastion Host는 외부 네트워크 (인터넷)에서 사설 네트워크(내부)로 접근할 때 거치는 중계 서버로써 외부 공격을 차단하는 요새 역할을 하며, SSH 접속을 단일화하여 보안을 강화하고, 내부 서버의 접근 제어를 수행합니다.

OCI에서는 별도의 Bastion 서버를 두지 않아도 Bastion 서비스를 이용하여 사설 망에 있는 서버에 접속할 수 있지만, 별도의 서버가 없다면 매번 Bastion 리소스를 생성해야하는 번거로움이 있고, 향후 CI/CD용 서버로도 사용할 예정이기 때문에 Bastion 서버를 구성해보겠습니다.

<br>

# 가상머신 생성

<br>

![ocicomputemenu]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_menu.jpeg)

OCI 전체 메뉴의 Compute를 클릭하고, Instance를 클릭하여 메뉴로 진입합니다.

![ocicomputemain]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_main.jpeg)

메인 메뉴에 진입하여 compartment가 맞게 설정되었는지 확인하고, "Create instance" 버튼을 클릭하여 가상머신 생성 페이지로 이동합니다.

![ocicomputecreate]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create.jpeg)

생성할 가상머신의 이름을 입력합니다. 네이밍 룰이 필요하며, 저는 직관적으로 {PRD/DEV}_{Application}으로 작성하였습니다.

![ocicomputecreate2]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create2.jpeg)

아래로 내리다 보면 Image가 있고 기본적으로 Oracle Linux가 선택되어 있습니다.

오라클 리눅스도 훌륭하지만 낮은 성능의 VM에서 메모리를 많이 잡아먹는 경향이 있기에 변경하고자 합니다.

Change Image 버튼을 클릭합니다.

![ocicomputecreate3]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create3.jpeg)

요즘 가상 많이 사용하는 Ubuntu를 선택한 뒤, 아래로 내려려 최신 이미지이면서 리소스를 조금만 사용할 수 있는 Ubuntu 24.04 Minimal을 선택하고 Select Image 버튼을 클릭합니다.

![ocicomputecreate4]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create4.jpeg)

마찬가지로 Shape를 변경하겠습니다. Change Shape 버튼을 클릭합니다.

![ocicomputecreate5]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create5.jpeg)

저는 항상 무료인 VM을 만들 예정이므로, Virtual machine을 선택하고, Specialty and previous generation을 선택하겠습니다.

Always Free-eligible 딱지가 붙어있는 VM.Standard.E2.1.Micro Shape를 선택하고, Select Shape 버튼을 클릭합니다.

![ocicomputecreate6]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create6.jpeg)

구성된 내용이 맞는지 확인하고, 별도로 Adbanced Option은 없이 Next를 클릭합니다.

![ocicomputecreate7]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create7.jpeg)

Secutiry Option을 선택할 수 있지만 잘 모르는 옵션이므로 선택하지 않고 Next를 클릭하여 진행하겠습니다.

![ocicomputecreate8]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create8.jpeg)

Networking 부분에서는 우선 VNIC 이름을 입력합니다. 네이밍룰은 {DEV/PRD}_{Application}_VNIC로 작성하였습니다.

Network 및 Subnet은 이전에 생성한 자원을 사용하게끔 설정하였고, Public Subnet인 DMZ_Subnet에 생성될 수 있도록 선택하였습니다.

<br>

## SSH Key 생성

<br>

SSH 접속에 사용할 Key는 새로 내려받아도 되지만 키를 소중히 관리해야하고, 가끔 Key에 문제가 생기는 경우가 있었던 것 같아서 로컬에서 만들어서 pub key를 업로드 하는 방식으로 이용하겠습니다.

아래의 명령어를 노트북의 터미널에서 실행하여 ssh key를 생성합니다.

```zsh
ssh-keygen -t rsa -b 2048 -f ~/.ssh/prd_oci
```

![sshkeygen]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/ssh-key-gen.jpeg)

private key, public key가 생성된 이후에는 마찬가지로 터미널에서 아래 명령어를 수행하여 생성된 pub 키를 출력하고, 출력된 키를 복사합니다 (ssh-rsa부터 .local 등 마지막까지)

```zsh
cat .ssh/prd_oci.pub
```

![ocicomputecreate9]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create9.jpeg)

다시 OCI 콘솔로 돌아와서, Paste public key를 선택한 뒤, 터미널에서 복사한 public key를 붙여넣기하고 Next 버튼을 클릭합니다.

![ocicomputecreate10]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create10.jpeg)

다음으로 부트 볼륨을 선택하는데, 이부분도 기본 옵션을 유지한채로 Next를 클릭합니다.

![ocicomputecreate11]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_compute_create11.jpeg)

마지막으로 잘못된 부분이 없는지 검토한 뒤, Create 버튼을 클릭하여 생성을 완료합니다.

이후 VM 상세 페이지로 이동하며 조금 기다리면 프로비저닝이 완료되고, Running 상태로 VM이 활성화 됩니다.

<br>

# Bastion 방화벽 설정

<br>

Bastion은 모든 서버의 관문 역할을 할 예정이므로 본인만 접속할 수 있도록 철저하게 방화벽을 등록합니다.

![ocifirewall]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_firewall.jpeg)

Networking 메뉴의 Virtual cloud networks에서 생성된 VCN (PRD_MLPLATFORM_VCN)을 선택한 뒤, Subnet에서 Bastion 서버가 위치한 서브넷인 DMZ_Subnet을 클릭합니다.

![ocisecurity]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_security.jpeg)

Subnet에서는 Security를 클릭하여, 기본적으로 구성된 security list를 클릭합니다.

![ocisecuritylist]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_securitylist.jpeg)

기본적으로 public subnet에서는 모든 곳에서 22번 포트로 통신이 가능하게끔 ingress rule이 설정되어 있습니다.

오른쪽 점세개를 클릭하여 Edit 버튼을 클릭하고 이부분은 저만 들어올 수 있도록 수정합니다.

![ocisecuritylist2]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_securitylist2.jpeg)

[내 아이피확인](https://ip.pe.kr/) 사이트에 접속해서 자택의 IP 주소를 확인합니다.

Source IP를 0.0.0.0/0 대신 위에서 확인한 IP주소/32로 입력하고, 이 규칙이 어떤 규칙인지 확인이 쉽도록 Description을 입력한 뒤, save changes 버튼을 클릭하여 저장합니다.

<br>

# Bastion 서버 SSH 접속

<br>

이제 다시 Bastion 서버로 돌아와서, 상세 페이지를 조금 내려보면 Bastion 서버에 할당된 Public IP Address를 확인할 수 있습니다.

![ocibastionmain]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_bastion_main.jpeg)

이 값을 확인하고 VSCODE 등 IDE에서 쉽게 접속할 수 있도록 원격 접속 도구에서 아래와 같이 입력합니다.

![vscodessh]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/vscode_ssh.jpeg)

VSCODE의 REMOTE-SSH 확장 프로그램은 ~/.ssh/config 파일로 이 값을 관리하며, 아래와 같이 bastion 관련 설정값을 입력 후 저장합니다.

```bash
Host bastion
  HostName 146.56.106.229
  User ubuntu
  IdentityFile /Users/jyhong/.ssh/prd_oci
```

![vscodessh2]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/vscode_ssh2.jpeg)

이제 REMOTE-SSH 실행시에 bastion을 목록에서 확인할 수 있고, 클릭하면 bastion 장비에 접속합니다.

최초 접속시 입력하는 창이 나온다면 yes 입력 후 엔터를 입력하면 VSCode Server를 bastion 서버에서 내려받고 잠시 뒤 서버에 연결됩니다.

![sshbastion]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/ssh-bastion.jpeg)

마지막으로 Bastion 서버가 추가됨으로써 아래와 같이 인프라가 구성되었습니다.

![ociinfra]({{ juyoung-hong.github.io }}/assets/images/2026-03-31-ocibastion/oci_infra_20260331.jpeg)