---
title: "오라클 쿠버네티스 접속 환경 설정하기"
classes: wide
categories:
  - infra
tags:
  - oracle-cloud
  - oke
  - kubernetes
---

<br>

# 오라클 쿠버네티스 접속 환경 설정하기

<br>

오늘은 지난 글에 이어서 오라클 클라우드에 접속할 수 있는 환경을 Bastion에 설정해 보겠습니다.

오라클 쿠버네티스 클러스터와 Bastion이 만들어져 있는 상태여야 하므로 이전 글과 함께 봐주시면 감사하겠습니다.

- 오라클 클라우드로 무료 인프라 네트워크 구성하기: [링크](https://juyoung-hong.github.io/infra/ocinetwork/)
- 오라클 클라우드에 무료 bastion 서버 구성하기: [링크](https://juyoung-hong.github.io/infra/ocibastion/)

<br>

# OCI CLI 설정

<br>

현재 Bastion과 OKE가 만들어져 있는 상태에서 생성된 OKE에 접근할 수 있도록 Bastion에 OCI CLI를 설치하고 설정을 진행하겠습니다.

<br>

## Bation swap 메모리 설정

<br>

Always Free 사이즈인 VM.Standard.E2.1.Micro 로 만들어진 Bastion은 메모리가 1GB 밖에 안되기 때문에 조금만 무거운 작업을 하면 자주 먹통이 되곤합니다.

![set_swap_memory]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/set_swap_memory.jpeg)

작업을 수행하는 과정에 이러한 현상이 발생하지 않도록 아래 명령어를 통해 스토리지의 일부를 메모리로 활용할 수 있도록 해줍니다.

<br>

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=4096
sudo chown root:root /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

<br>

Bastion에서 파일을 손쉽게 수정할 수 있도록 아래 명령어로 Vim을 설치합니다.

<br>

```bash
sudo apt-get update
sudo apt-get install vim
```

<br>

Vim 설치 이후, VM이 재기동된 이후에도 이러한 스왑메모리 설정이 유지될 수 있도록 /etc/fstab 파일을 열어서 아래와 같이 수정합니다.

![edit_fstab]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/edit_fstab.jpeg)

<br>

```bash
sudo vi /etc/fstab
/swapfile swap swap defaults 0 0
```

<br>

마지막으로 아래 명령어를 통해, 잘 설정되었는지 확인합니다.

![check_swap_memory]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/check_swap_memory.jpeg)

<br>

```bash
sudo swapon -s
```

<br>

## OCI CLI 설치

<br>

이제 본격적으로 OCI CLI를 설치하기 위해 아래 명령어를 이용하여 OCI CLI를 다운로드 받습니다.

Default 옵션을 사용하기 위해 이후 몇가지 문의에 대해서는 엔터를 입력하여 넘어갔습니다.

버전을 확인하면 잘 설치된것을 확인할 수 있습니다.

![install_oci_cli]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/install_oci_cli.jpeg)

<br>

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
source ./.bashrc
oci --version
```

<br>

## OCI CLI 설정

<br>

이제 클라우드 자원 제어를 위한 API Key pair를 생성합니다.

테넌시의 OCID와 사용자의 OCID가 필요한데, 아래와 같이 확인할 수 있습니다.

<br>

### 테넌시 OCID 확인

<br>

![tenancy_ocid]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/tenancy_ocid.jpeg)

<br>

### User OCID 확인

<br>

![user_ocid]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/user_ocid.jpeg)

<br>

![setup_config]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/setup_config.jpeg)

이제 아래 명령어를 가지고 설정을 진행합니다.

```bash
oci setup config
```

이제 생성된 API public Key를 OCI 사용자 API Key에 등록합니다.

![cat_ssh_publickey]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/cat_ssh_publickey.jpeg)

먼저 생성된 oci api key 중 public key를 복사합니다.

```bash
cat ~/.oci/oci_api_key_public.pem
```

이후 User OCID를 확인했던 아래 메뉴로 이동하여 복사한 API Key를 붙여넣기합니다.

![ociconsole_apikey]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/ociconsole_apikey.jpeg)

![add_apikey]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/add_apikey.jpeg)

![check_ocicli]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/check_ocicli.jpeg)

API Key 등록이 끝난 후에는 아래 명령어로 OCI CLI가 정상 동작하는지 확인합니다.

```bash
oci os ns get
oci iam region list --output table
```