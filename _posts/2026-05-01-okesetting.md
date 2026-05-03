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

<br>

# Kubectl 설정

<br>

이제 아래 명령어를 가지고 kubectl을 설치합니다.

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

설치 이후 잘 설치되었는지 아래의 명령어로 확인해봅니다.

![install_kubectl]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/install_kubectl.jpeg)

```bash
kubectl version --output=yaml
```

<br>

## Kubeconfig 설정

<br>

이제 bastion에 설치한 kubectl을 통해서 OKE Cluster에 접근할 수 있도록 kubeconfig 파일을 구성합니다.

아래와 같이 OKE Cluster 상세 화면에서 Access Cluster 메뉴를 클릭합니다.

![access_cluster]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/access_cluster.jpeg)

![access_cluster2]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/access_cluster2.jpeg)

Local Access를 클릭한 이후 노출되는 아래의 명령어들을 입력하여 구성합니다.

```bash
oci -v
mkdir -p $HOME/.kube
oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.ap-chuncheon-1.aaa....
export KUBECONFIG=$HOME/.kube/config
```

![access_cluster3]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/access_cluster3.jpeg)

잘 설정이 되면 아래의 명령어로 현재 구성된 클러스터의 노드 정보를 가져올 수 있습니다.

```bash
kubectl get nodes
```

<br>

## Docker 설치

이제 아래 명령어를 가지고 docker를 설치해줍니다.

<br>

```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

![install_docker]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/install_docker.jpeg)

설치가 끝난 이후에는 아래 명령어로 설치가 잘 되었고 정상 실행중인지 확인합니다.

```bash
sudo systemctl status docker
```

![config_docker]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/config_docker.jpeg)

이후 ubuntu user가 super user 권한 없이도 docker 명령어를 수행 할 수 있도록 아래 명령어로 권한을 부여해주고, nginx 이미지를 다운로드 받습니다.

```bash
sudo usermod -aG docker ubuntu
newgrp docker
docker pull nginx
```

<br>

## 쿠버네티스 클러스터에 배포하기

<br>

### Worker Node Route Rule 수정

<br>

![set_NAT_rules]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/set_NAT_rules.jpeg)

worker node가 외부 인터넷에서 nginx 이미지를 다운로드해서 배포할 수 있도록 Worker Node Subnet에 NAT Gateway를 사용하는 Route Rule을 추가합니다.

<br>

### Worker Node Egress Rule 수정

<br>

![set_Egress_rule]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/set_Egress_rule.jpeg)

worker node가 외부 인터넷에서 nginx 이미지를 다운로드해서 배포할 수 있도록 Worker Node Subnet에 Egress Rule도 추가합니다.

<br>

### OKE 배포하기

<br>

이제 준비된 nginx 이미지를 OKE 클러스터에 배포해보겠습니다.

저는 아래와 같은 yaml파일을 작성해서 deployments 폴더 안에 nginx.yaml파일로 저장하였습니다.

무료로 loadbalancer를 사용할 수 있도록 shape은 flex로 하고 min, max 값은 무료 제한인 10으로 설정하였습니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: docker.io/library/nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "500m"
            memory: "512Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  annotations:
    service.beta.kubernetes.io/oci-load-balancer-shape: "flexible"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-min: "10"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-max: "10"
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
```

![deploy_kubernetes]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/deploy_kubernetes.jpeg)

이후 아래 명령어로 배포하고, 쿠버네티스 클러스터의 상태를 확인합니다.

모든 pod가 정상적으로 running 상태이고, service도 정상적으로 생성되었습니다.

```bash
kubectl apply -f deployments/nginx.yaml
kubectl get all
```

![create_lb]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/create_lb.jpeg)

이제 OCI 콘솔의 Loadbalancer를 확인하면 LB가 하나 생성되어 있는 것을 확인할 수 있습니다.

LB의 public IP는 service의 external IP와도 일치하고, 이 IP 주소를 브라우저에 입력하고 접속해보면, 아래와 같이 nginx가 잘 배포 되어 있는 것을 확인할 수 있습니다.

![access_nginx]({{ juyoung-hong.github.io }}/assets/images/2026-05-01-okesetting/access_nginx.jpeg)

다음으로는 jenkins로 CI/CD 구성을 하고 직접 만든 앱을 배포해보겠습니다.