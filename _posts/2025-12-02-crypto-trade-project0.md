---
title: "비트코인 자동매매 프로젝트0 - 요구사항 정리 및 인프라 구성"
classes: wide
categories:
  - cryptotrade
tags:
  - project
  - cryptotrade
  - requests
---

<br>

# 개요

공부한 것들을 습득하려면 실습만큼 좋은건 없는 것 같다.

기존에 사용하던 비트코인 자동 매매 시스템을 더 깨끗한 구조와 기술 스택으로 웹기반 시스템으로 다시 만들어보고자 한다.

<br>

# 요구사항

1. 비용: 나혼자 사용할거기 때문에 가장 저렴하게 최대한 많은 무료 서비스를 이용하고자 한다. 이런관점에서 최대한 오라클 클라우드의 무료서비스를 활용할 것이다.

2. 소프트웨어 아키텍처: 여러가지로 구현을 시도해 봤고, 아직 답은 결정나지 않았지만 아래의 독립적인 서비스들이 필요할 것 같다.
  - 거래소와 인터페이스 하는 모듈
  - 데이터를 수집하거나 전처리하고 전달하는 모듈
  - 거래 전략을 가지고 시그널을 생성하는 모듈
  - 생성한 전략을 테스트할 수 있는 백테스팅 모듈

3. 기타 필요한 기능: 위의 기능들이 핵심이 될 것 같고 그 외에도, nginx, apigw, load balancing, APM, 메신저 알림 서비스 등이 추가로 필요할 것 같다.

4. 인프라 아키텍처: 대충 세어도 서비스가 7-8개 이상이 될 것 같은데, 아무리 경량화한 이미지를 사용하더라도, 각 서비스를 독립적으로 구성해서 3tier 아키텍처로 이중화 구성했을때는 오라클 클라우드 프리티어로는 CPU, RAM등 자원이 부족했다. 그래서 아쉽지만 이중화는 포기하고 하나의 VM안에 전부 다 때려넣기로 결정했다.

<br>

최종적으로는 오라클 클라우드의 OKE 서비스를 사용하여, 아래와 같은 구성으로 무료 인프라를 운영해보려고 한다.

- 4 OCPU, 24GB RAM, 200GB 스토리지를 최대한 잘 활용하여 서비스를 구성해봐야겠다.

![OKE_pricing]({{ juyoung-hong.github.io }}/assets/images/OKE_pricing.jpg)

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

root directory의 하위에 'crypto-dev-compartment'과 'crypto-prd-compartment'를 생성하였다. 

![create_compartment]({{ juyoung-hong.github.io }}/assets/images/create_compartment.jpg)

<br>

## 권한 관리

오라클 클라우드에서는 유저를 그룹으로 묶고, 그룹에 정책을 통해 권한을 부여한다.

권한 부여시 범위를 컴파트먼트 또는 테넌시로 제한할 수 있다.

유저와 그룹은 다시 도메인이라고 하는 더 포괄적인 개념으로 관리된다.

최초 계정 생성시 Default 도메인이 자동으로 생성되며, 추가로 도메인을 생성해서 관리할 수 있다.

<br>

### Domain 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-prd-compartment 컴파트먼트 선택 → Create Domain

Free Domain으로 crypto-prd-default-domain 을 아래와 같이 추가하였다.

Remote-DR 관련 설정은 서울 리전에서만 가능한 듯 보였는데, 나는 일단 DR 설정을 끄고 춘천 리전에서 생성하였다.

아래 이미지는 dev로 되어 있지만, 무료 티어 리소스의 한계로 OCI에서는 실제 운영할 인프라 관련해서만 설정하였다.

![create_domain]({{ juyoung-hong.github.io }}/assets/images/create_domain.jpg)

<br>

### Group 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-dev-compartment → crypto-prd-default-domain → User management → Group → Create group

crypto-prd-default-developer-group로 개발 환경의 개발자 그룹을 만들었고, 이 그룹에는 별도 유저는 할당하지 않았다.

![create_group]({{ juyoung-hong.github.io }}/assets/images/create_group.jpg)

<br>

### User 생성

- OCI 콘솔의 Identity & Security → Identity → Domains → crypto-prd-compartment → crypto-prd-default-domain → Users → Create

개발자 계정을 만들기 위해서, 이름은 동일하지만 메일 주소는 다른것으로 개발자용 계정을 생성하였고, crypto-prd-default-developer-group 그룹에 해당 유저를 할당하였다.

마찬가지로 이미지와 달리 prd에 대해 만들었고, 이후 함께 개발할 사람이 있다면 초대할 예정이다.

![create_user]({{ juyoung-hong.github.io }}/assets/images/create_user.jpg)

<br>

### 권한 정책 생성

- OCI 콘솔의 Identity & Security → Identity → Policies → Create Policy

개발자들에게 prd 컴파트먼트에 대한 리소스 사용 권한을 주고자 아래와 같이 설정하였다.

- Policy 명: crypto-prd-default-developer-group-policy
- Policy (mannual):

```bash
allow group crypto-prd-default-developer-group to use all-resources in compartment crypto-prd-compartment
```

![create_policy]({{ juyoung-hong.github.io }}/assets/images/create_policy.jpg)

<br>

## 가상 클라우드 네트워크

VCN은 리전 레벨의 자원으로 여러개의 가용 도메일에 걸쳐 구성되므로, 복수의 가용 도메인을 가진 리전에서 VCN을 생성하면, 고가용성 및 이중화를 구현할 수 있다.

<br>

### 가상 클라우드 네트워크 생성

- OCI 콘솔의 Networking → Virtual Cloud Networks → Actions → Start VCN Wizard → VCN with internet connectivity

- VCN 명: crypto-prd-default-vcn

![create_vcn]({{ juyoung-hong.github.io }}/assets/images/create_vcn.jpg)

<br>

### 서브넷

VCN을 더 잘게 분할한 하위 네트워크 자원으로 각 서브넷은 다른 CIDR 블록 범위를 가지며, 겹치지 않도록 구성된다.

<br>

### 라우트 테이블

VCN에서 IP 패킷의 전달 경로를 결정하는데 사용되는 자원으로 통신이 원활하려면 서브넷에 적절한 라우트 규칙을 설정해야 한다.

<br>

### 시큐리티 리스트

VCN 내에서 Ingress 및 Egress 트래픽에 대한 보안 규칙을 정의하는 자원이며, 같은 서브넷 안에 속한 모든 인스턴스는 동일한 보안 규칙을 적용받는다.

- Stateful 규칙: 수신 규칙과 관계 없이 트래픽이 원래 호스트로 돌아갈 수 있음 (요청을 보낸 호스트로 응답이 돌아갈 수 있도록 허용하는 경우)
- Stateless 규칙: 트래픽의 수신과 송신간의 관련성이 없어 응답을 추적하지 않는 경우 사용 (특정 포트로 들어오는 트래픽에 대해서 응답을 차단하고 싶은 경우)

<br>

### 인터넷 게이트웨이

VCN 내의 인스턴스가 인터넷과 통신하는 입구 역할을 하며, 이를 위해서는 퍼블릭 서브넷 내의 자원들이 public IP를 가져야 한다.

<br>

### NAT 게이트웨이

public IP 주소가 없는 클라우드 자원이 인터넷으로 나가는 접근을 가능하게 하는 가상 라우터이다. 

private 서브넷 내의 자원들이 인터넷 접속이 필요할때 사용한다.

<br>

### 서비스 게이트웨이

OCI 내의 다양한 오라클 클라우드 서비스에 대한 private 접근을 허용하는 게이트 웨이이다.

데이터를 인터넷에 노출시키지 않고 VCN내의 자원들이 오라클 클라우드 서비스에 안전하게 비공개 접근하게 한다.

<br>

### Dynamic Routing Gateway

여러 리전의 VCN 또는 온프레미스 네트워크를 private 네트워크로 연결할 때 
사용하는 가상 라우터이다.

<br>

### 네트워크 구성 

다음 [링크](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengnetworkconfigexample.htm#example-oci-cni-privatek8sapi_privateworkers_publiclb)의 오라클 도큐먼트를 참고하여, Cluster with OCI CNI Plugin, Private Kubernetes API Endpoint, Private Worker Nodes, and Public Load Balancers를 구성하였다.

<br>

#### 라우트 테이블 생성

Example 문서에 따라 각 서브넷 별로 할당할 IP 주소 대역대를 미리 지정해 놓고 각 역할에 맞추어 라우트 테이블, 시큐리티 리스트를 생성하고, 마지막으로 subnet을 생성하였다.

문서와 다른 점은 각 pods, workernodes 등이 인터넷에 접속할 수 있도록 NAT Gateway를 라우트 테이블에 추가해줬고, 시큐리티 리스트에도 Egress rule을 추가해줬다.

먼저 아래의 라우트 테이블을 생성하였다.

- routetable-KubernetesAPIendpoint
- routetable-workernodes
- routetable-pods
- routetable-serviceloadbalancers

<br>

#### Security List 생성

각 서브넷에 적용할 Security List를 생성하였다.
- seclist-KubernetesAPIendpoint
- seclist-workernodes
- seclist-pods
- seclist-loadbalancers
- seclist-Bastion

<br>

#### 서브넷 생성

이후 아래와 같이 서브넷을 생성하였다.

- (private) KubernetesAPIendpoint
- (private) workernodes
- (private) pods
- (public) loadbalancers
- (private) bastion

<br>

## OCI Container Engine for Kubernetes

마이크로서비스아키텍처는 컨테이너를 사용하여 구성한다.

컨테이너는 애플리케이션을 쉽게 패키징할 수 있으므로 이식성을 높여주고, 클라우드 환경에서도 쉽게 실행할 수 있다.

여기에 쿠버네티스와 같은 컨테이너 오케스트레이션 도구를 함께 사용하면, 배포, 스케일링, 로드 밸런싱, 컨테이너 오토스케일링 등의 작업을 자동화 할 수 있고, 서비스 메시 아키텍처를 구현할 수 있는 도구와 함께 사용하면, 트래픽 관리, 보안, 모니터링을 더 쉽게 할 수 있다.

OKE는 오라클이 제공하는 광리형 쿠버네티스 서비스로 오라클이 쿠버네티스를 자동으로 설치, 구성, 관리, 업그레이드하고, 클러스터를 제공한다.

<br>

### OKE 생성

<br>

#### SSH Key 생성

로컬 컴퓨터에서 ssh key 생성 (.ssh 하위의 oci_crypto_prd로 생성함)

```zsh
ssh-keygen -t rsa -b 2048 -f /Users/jyhong/.ssh/oci_crypto_prd
```

<br>

#### OKE Cluster 생성

- OCI 콘솔의 Developer Services → Kubernetes Clusters(OKE) → Create Cluster → Custom Create
  - Name: crypto-prd-cluster
  - Kubernetes Version: 1.34.1
  - VCN native pod networking
  - network for cluster: KubernetesAPIendpoint
  - network for load balancer: loadbalancers
  - node Name: crypto-prd-cluster-nodepool
  - node Type: Managed
  - node place: workernodes
  - node shape: VM.Standard.A1.Flex
  - node OCPU: 4 OCPU
  - node Memory: 24 GB
  - node Image: aarch64-2025.08.31-0 (Oracle Linux 8.10)
  - node count: 1 (OKE지만 자원 활용을 최대한 하기위해 1개만 생성하였다.)
  - Pod communication: pods

<br>

### Bastion 구성

이후에 private network에 위치한 oke cluster에 접근할 수 있도록 bastion 용도의 가상머신을 생성하였다.

<br>

#### 가상머신 생성

- OCI 콘솔의 Compute → Instances → Create Instance
  - instance name: bastion
  - Image: Oracle Linux8
  - Shape: VM.Standard.E2.1.Micro
  - Primary VNIC: 위에서 생성한 VCN 및 bastion subnet 선택 후 자동 할당
  - SSH: 위에서 만든 ssh pub키 업로드
  - 부트볼륨: 자동 선택

<br>

#### 메모리 swap 설정

VM.Standard.E2.1.Micro는 CPU 사양도 낮지만 메모리가 1GB 밖에 안되어, 아주 작은 작업만 하더라도 접속 불가 상태가 되어버려 거의 쓸수가 없다.

원활하게 사용하기 위해서 disk를 swap 메모리로 사용할 수 있도록 아래와 같이 설정한다.

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=4096
sudo chown root:root /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

VM이 재기동 될때 자동으로 swap 설정이 적용되도록 /etc/fstab 파일에 다음 내용을 추가한다

```text
/swapfile swap swap defaults 0 0
```

아래와 같이 제대로 설정되었는지 확인한다.

```bash
sudo swapon -s
```

<br>

#### OCI CLI 설치

위에서 만든 bastion 가상머신에 접속하여, 아래와 같은 명령어로 OCI CLI를 설치한다.

```bash
sudo dnf -y install oraclelinux-developer-release-el8
sudo dnf install python36-oci-cli
```

설치된 이후에는 API Key를 생성한다.

```bash
oci setup config
```

만들어진 API Key pair 중 퍼블릭키를 OCI의 사용자 정보에 등록한다.

- OCI 콘솔의 My Profile → Tokens and keys → API keys → Add API Key

이후에는 아래와 같은 명령으로 문제없이 잘 되는지 테스트한다.

```zsh
oci os ns get
oci iam region list --output table
```
<br>

#### kubectl 설치

쿠버네티스 클러스터를 제어하기 위해 kubernetes.io 홈페이지에서 아래와 같은 명령을 찾아설치한다.

```zsh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

<br>

#### kubeconfig 파일 생성 및 클러스터 접속

- OCI웹콘솔의 OKE 상세 화면 → Containers → Clusters → Cluster details → Access Cluster → local access에 있는 명령을 수행한다

```bash
mkdir -p $HOME/.kube
oci ce cluster create-kubeconfig --cluster-id <cluster id> --file $HOME/.kube/config --region <region> --token-version 2.0.0  --kube-endpoint PRIVATE_ENDPOINT
export KUBECONFIG=$HOME/.kube/config
```

이후 재시작 하더라도 계속 접근할 수 있도록 ~/.bash_profile에 아래 문구를 추가한다.

```bash
export KUBECONFIG=$HOME/.kube/config
```

아래와 같이 정상적으로 config가 생성된 것을 확인할 수 있고, 클러스터 구성을 조회할 수 있다.

```bash
more ~/.kube/config
kubectl cluster-info
kubectl config view
kubectl get nodes
```

<br>

### nginx 배포 테스트

본격적으로 서비스를 배포하기 전에 간단히 nginx를 배포해본다.

bastion에 접속하여 아래와 같이 deployment.yaml 파일을 만든다.

```bash
vi deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  annotations:
    oci.oraclecloud.com/load-balancer-type: "lb"
    service.beta.kubernetes.io/oci-load-balancer-shape: "flexible"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-min: "10"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-max: "10"
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: docker.io/library/nginx:1.14.2
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f deployment.yaml
kubectal get all
```

정상적으로 2개의 pod가 동작하는 것을 확인할 수 있다.

<br>

#### 서비스 배포 및 테스트

마지막으로 public load balancer를 배포하여, 직접 브라우저에서 테스트 해본다.

OCI에서 flexible LB 10Mbps까지는 무료로 사용할 수 있으므로 기존 deployment.yaml 파일을 아래와 같이 수정한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: docker.io/library/nginx:1.14.2
        ports:
        - containerPort: 80
---
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
  annotations:
    oci.oraclecloud.com/load-balancer-type: "lb"
    service.beta.kubernetes.io/oci-load-balancer-shape: "flexible"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-min: "10"
    service.beta.kubernetes.io/oci-load-balancer-shape-flex-max: "10"
spec:
  selector:
    app: nginx
  type: LoadBalancer
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: https
    port: 443
    targetPort: 80
```

이후 아래 명령으로 적용한다.

```bash
kubectl apply -f deployment.yaml
kubectl get all
```

조금 기다리면 OCI 콘솔에는 로드밸런서가 생성되어 있고, 생성된 public lb의 IP 주소로 접근하면 아래와 같이 nginx가 잘 구동되어 있다.

![nginx_test]({{ juyoung-hong.github.io }}/assets/images/nginx_test.jpg)