---
title: "비트코인 자동매매 프로젝트1 - 마이크로서비스 컨테이너 이미지 생성"
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

지난 글에서는 OKE Cluster를 public loadbalancer 와 나머지 리소스들은 private network에 위치하게끔 구성하고 nginx 이미지를 테스트로 배포해보았다.

이번에는 nginx 말고 앞으로 개발하고자 하는 소스코드를 컨테이너 이미지로 만들어서 배포하고자 한다.

<br>

# Github 세팅

OKE에 배포할 CI/CD 용 머신도 프리티어의 한계로 좋은 장비는 사용하지 못할 것 같다.

그리하여 소스코드도 모노레포로 구성하는 것보다 멀티레포로 구성하여 수정사항이 있는 독립적인 서비스만 배포하는 것이 유리하다고 판단했고, 레포지토리가 많아질 것을 생각하여 Organization을 만들어 한꺼번에 관리하고자 한다.

<br>

## Organization 생성

다음 [링크](https://velog.io/@gmlstjq123/Github-Organization-%EB%A7%8C%EB%93%A4%EA%B8%B0)의 글을 참고하여 무료로 "CryptoAutoTradingTeam" 라는 Organization을 생성하였다.

<br>

## Repo 생성

우선은 간단한 화면만 React로 만들어서 front-end 서비스만 배포할 생각으로 "Frontend-Desktop" 이라는 Repository를 생성하였다.

<br>

# 소스 코드 생성

빈 repo를 노트북에 clone 해와서 vite를 통해서 react + typescript 환경을 만들었다.

```zsh
npm create vite@latest frontend-desktop
```

이후 Gemini를 활용해서 실제로 동작하지는 않는 로그인 화면을 빠르게 구현했다.

```tsx
// src/components/LoginForm.tsx

import React, { useState, type FormEvent } from 'react';
import './LoginForm.css'; // 스타일 파일 import

const LoginForm: React.FC = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    
    // TODO: 실제 백엔드 연동 로직 대신 콘솔 출력
    console.log('로그인 시도:', { username, password });
    alert(`로그인 시도: ${username}`);
  };

  return (
    // 전체 화면 컨테이너 (배경 및 중앙 정렬)
    <div className="login-page-container">
        <form className="login-form-box" onSubmit={handleSubmit}>
            
            <h2 className="title">사용자 로그인</h2>
            
            {/* 아이디 입력 그룹 */}
            <div className="input-group">
                <label htmlFor="username" className="label">아이디</label>
                <input
                    type="text"
                    id="username"
                    value={username}
                    onChange={(e) => setUsername(e.target.value)}
                    required
                    className="input-field"
                    placeholder="아이디 입력"
                />
            </div>
            
            {/* 비밀번호 입력 그룹 */}
            <div className="input-group">
                <label htmlFor="password" className="label">비밀번호</label>
                <input
                    type="password"
                    id="password"
                    value={password}
                    onChange={(e) => setPassword(e.target.value)}
                    required
                    className="input-field"
                    placeholder="비밀번호 입력"
                />
            </div>
            
            {/* 로그인 버튼 */}
            <button type="submit" className="login-button">
                로그인
            </button>
        </form>
    </div>
  );
};

export default LoginForm;
```

```tsx
/* src/components/LoginForm.css */

/* 1. 전체 페이지 스타일: 배경 및 중앙 정렬 */
.login-page-container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    /* 이미지와 비슷한 배경 (밝은 회색 계열) */
    background-color: #f0f2f5; 
    padding: 20px;
}

/* 2. 폼 박스 스타일 */
.login-form-box {
    background: white;
    padding: 40px 50px; /* 데스크톱에 맞춰 패딩 증가 */
    border-radius: 8px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1), 0 5px 10px rgba(0, 0, 0, 0.05); /* 깊은 그림자 */
    width: 100%;
    max-width: 400px; /* 데스크톱에 적합한 최대 너비 */
    text-align: center;
}

.title {
    font-size: 24px;
    font-weight: 600;
    color: #333;
    margin-bottom: 30px;
}

/* 3. 입력 필드 그룹 스타일 */
.input-group {
    margin-bottom: 20px;
    text-align: left;
    position: relative; /* 비밀번호 동그라미 위치 조정을 위해 */
}

.label {
    display: block;
    font-size: 14px;
    color: #555;
    margin-bottom: 5px;
}

.input-field {
    width: 100%;
    padding: 10px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    box-sizing: border-box; 
    font-size: 16px;
    transition: border-color 0.2s;
}

.input-field:focus {
    border-color: #007bff;
    outline: none;
}

/* 4. 버튼 스타일 */
.login-button {
    width: 100%;
    padding: 12px;
    background-color: #007bff; /* 파란색 버튼 */
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
    margin-top: 10px;
    transition: background-color 0.2s;
}

.login-button:hover {
    background-color: #0056b3;
}
```

App.tsx, App.css 등 기타 다른 파일들도 정리하고 아래 명령어로 실행해보니 정상적으로 실행이 가능하였다.

```zsh
npm run dev
```

<br>

# 소스 코드 배포

<br>

## docker file 생성

<br>

정말 간단한 로그인 페이지이지만 현재 만들어진 클러스터에 이 내용을 배포하려고 한다.

또 다시 Gemini를 통해서 docker file과 nginx의 conf 파일을 빠르게 만들었다.

```nginx
# nginx.conf

server {
    listen 80;
    server_name  localhost;

    root   /usr/share/nginx/html;
    index  index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # API 호출이 백엔드로 전달되어야 한다면, 여기에 프록시 설정을 추가합니다.
    # location /api {
    #     proxy_pass http://<백엔드_서비스_주소>;
    #     proxy_set_header Host $host;
    #     proxy_set_header X-Real-IP $remote_addr;
    # }

    # 에러 페이지 설정 (선택 사항)
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

```dockerfile
# Docker file 
#==========================================================
# STAGE 1: 빌드 스테이지 (Build Stage)
# ==========================================================
# Node.js LTS 버전의 Alpine 리눅스 이미지를 사용합니다.
FROM node:20-alpine AS builder

# 작업 디렉토리 설정
WORKDIR /app

# package.json 및 lock 파일을 복사하고 의존성을 설치합니다.
# 이 과정이 분리되어야 node_modules가 변경되지 않는 한 캐시를 사용할 수 있습니다.
COPY package*.json ./
RUN npm install

# 나머지 소스 코드 복사
COPY . .

# Vite 앱 빌드
# 결과물은 /app/dist 에 생성됩니다.
RUN npm run build


# ==========================================================
# STAGE 2: 실행 스테이지 (Run Stage) - Nginx 기반
# ==========================================================
# Nginx의 공식 stable Alpine 이미지를 사용합니다. (매우 작음)
FROM nginx:stable-alpine

# OKE 배포에 적합하도록 Nginx 설정을 덮어쓰기 위해 설정 파일을 복사합니다.
# 이 파일은 아래 2.1 단계에서 작성합니다.
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 빌드 스테이지에서 생성된 정적 파일(dist 내용)을 Nginx의 웹 루트로 복사합니다.
COPY --from=builder /app/dist /usr/share/nginx/html

# Nginx 기본 포트 80 노출
EXPOSE 80

# 컨테이너 시작 시 Nginx 실행
CMD ["nginx", "-g", "daemon off;"]
```

이후 아래 명령어로 컨테이너를 build 하고 실행해보았고 역시 잘 실행되는 것을 확인했다.

```zsh
docker build -t frontend-desktop:v1.0 .
docker run --name frontend-desktop -p 8080:80 frontend-desktop:v1.0
```

![frontend_desktop_index]({{ juyoung-hong.github.io }}/assets/images/frontend_desktop_index.jpg)

<br>

## OCI 컨테이너 레지스트리 생성

OCI의 컨테이너 레지스트리는 사용하는데는 비용이 들지 않고, 저장하는데 오브젝트 스토리지와 같은 요율로 비용을 청구한다고 한다.

정확한 내용은 찾기 어려웠지만 OCI 프리티어는 오브젝트 스토리지 10GB까지가 무료이므로 일단 한번 사용해보려고 한다. (위에서 만든 도커이미지는 약 76.4MB 정도로 10 GB를 넘으려면 한참 필요하다)

- OCI 콘솔의 Developer Services → Container Registry → Create Repository
  - access: private
  - repo name: crypto-prd-repo/frontend-desktop

## OCI 인증 토큰 생성

- OCI 콘솔 우측 상단 My Profile → Tokens and Keys → Auth Tokens → Generate Token
  - Description: crypto-prd-auth-token

## docker 설치

기본적으로 oracle linux에는 docker가 설치되어 있지 않아 로그인이 불가능했다.

따라서 아래 명령어로 먼저 bastion에 docker를 설치해줬다.

```zsh
sudo dnf update -y
sudo yum install yum-utils dnf-utils zip unzip -y
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf remove -y runc
dnf install -y docker-ce --nobest
systemctl enable docker.service
systemctl start docker.service
```

## 컨테이너 레지스트리 로그인

먼저 아래의 명령어로 테넌시 네임스페이스를 확인한다. PW는 위에서 만든 토큰이다.

사용자 정보는 OCI 웹콘솔의 Profile을 클릭하면 나오는 My profile 화면에서 확인할 수 있다.

```zsh
oci os ns get
```

이후 아래 명령어로 컨테이너 레지스트리에 로그인한다.
유저네임을 물어보면 <테넌시 네임스페이스>/<사용자> 로 답한다.

private으로 컨테이너 레지스트리를 만들었기 때문에 public 환경에서는 접근할 수 없고, bastion 등 oci private network에 위치한 VM 통해서만 접근이 가능하다.

```zsh
docker login ap-chuncheon-1.ocir.co
```
