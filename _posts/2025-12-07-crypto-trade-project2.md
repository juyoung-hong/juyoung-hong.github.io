---
title: "비트코인 자동매매 프로젝트2 - 소스코드 구성 (Git Hub)"
classes: wide
categories:
  - cryptotrade
tags:
  - project
  - cryptotrade
  - git
---

<br>

# GitHub

구성된 인프라 위에 소스코드를 구성하고자 한다.
먼저 Github를 원격 리포지토리로 사용하여 전체 코드를 관리하고자 하니 레포를 먼저 만들겠다.

<br>

## Organization 생성

현재도 그렇지만 나중에도 협업할 가능성이 있고, 무엇보다 관련된 리포지토리들을 묶어서 관리하고 싶은 마음에 Organization을 먼저 생성하였다.

Free 버전으로 CryptoAutoTradingTeam 을 생성하였고, Organization의 생성 방법은 구글링이나 다른 블로그 글을 찾아보면 상세히 설명되어 있다.

<br>

## Repository 생성

이후에 생성된 Organization에 CryptoAutoTradeSystem 라는 Public Repo를 만들었다.

이제 이 비어있는 Repo를 clone 해와서 local 개발환경을 세팅할거다.

<br>

# 개발 환경 구성

노트북 터미널을 열고 아래의 명령어를 수행하여 원격 repository를 clone 해 왔다.

```zsh
git clone https://github.com/CryptoAutoTradingTeam/CryptoAutoTradeSystem.git
```

빈 repository가 clone 된 이후에는 vscode로 작업 폴더를 열어 작업을 시작한다.

<br>

## 작업 폴더 구성

작업 폴더 구성하는 것에 대해 조금 살펴보니, 여기서부터 시스템 아키텍처가 크게 중요해지는 것 같다.

MSA, 헥사고날 등 아키텍처에 따라서, 그리고 모노레포, 멀티레포 등에 따라 폴더 구조가 달라지게 되는데, 여기서 공부해야할게 너무 많아서 막혀버리게 되었다.

결국 발전은 나중에 천천히 시키기로 하고, 지금 당장 쉽게 적용할 수 있는 폴더 구조로 프론트와 백만 나누고 진행하기로 했다.

그리고 프론트엔드쪽은 이전에 배운 것처럼 vite 라는 번들링 툴로 구성하려고 한다.

<br>

## FastAPI 프로젝트 생성

<br>

### 가상 환경 생성

이전에는 pyenv를 사용하여 많이 버전관리를 했었는데, 요즘은 uv라는 도구를 많이 사용한다고 한다.

속도도 빠르고, 패키치 설치, 관리, 빌드, 배포 모두 가능하다고 해서 이번 기회에 사용해보고자 한다.

python은 현재 시점 기준으로 가장 안정적인 최신 버전이 3.12 버전인것 같아서 3.12를 사용한다. 

개발 도중 버전 변경이 필요한 경우에는 그때 바꿔서 사용하고자 한다.

<br>

#### uv 설치

```zsh
curl -LsSf https://astral.sh/uv/install.sh | sh
uv init backend --python 3.12
cd backend
source .venv/bin/activate
```

<br>

#### FastAPI 설치

```zsh
uv add "fastapi[all]"
uv sync
```

#### now 서비스 작성

이후 server 폴더를 추가하고 간단한 지금 시간을 가져오는 코드를 작성해보았다.

```python
from datetime import datetime

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = ["http://localhost:3000"]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/api/now")
async def get_current_time():
    now = datetime.now().strftime("%Y년 %m월 %d일 %H:%M:%S")
    return {"now": now}
```

아래 명령어를 가지고 샘플 코드를 실행하였고, 정상 동작함을 확인하였다.

```zsh
uv run fastapi dev server/app.py
curl -X 'GET' \
  'http://127.0.0.1:8000/api/now' \
  -H 'accept: application/json'
```
<br>

## React 프로젝트 생성

가상 환경이 터미널에 적용된 것을 확인하고 react project를 생성한다.

```zsh
cd ..
npm create vite@latest
```

홈 작업 디렉토리에서 JavaScript + React compiler 옵션을 선택하고 frontend 프로젝트를 생성하였다.

<br>

### 소스 코드 수정

가장 간단히 서비스가 잘 동작할 수 있는지 확인할 수 있도록 App.jsx를 아래와 같이 수정하였다.

```jsx
import { useState } from 'react'

function App() {
  const backendUrl = 'http://localhost:8000/api/now';
  const [datetime, setDateTime] = useState("backend not connected!")

  fetch(backendUrl)
    .then(response => {
      if (!response.ok) {
        throw new Error('Backend response was not ok');
      }
      return response.json();
    })
    .then(data => {
      setDateTime(data.now);
    })
    .catch(error => {
      console.error("백엔드 연결 오류:", error);
    });

  return (
    <p>Hello, {datetime} </p>
  )
}

export default App
```

이후 main.jsx에서도 css 등 가져오는 부분이 없도록 아래와 같이 변경하였다.

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

<br>

#### 실행

이후 아래 명령어로 실행하면 아래 이미지와 같이 정상 동작함을 확인하였다.

```zsh
npm run dev -- --port 3000
```

![hello_now]({{ juyoung-hong.github.io }}/assets/images/hello_now.jpg)

<br>

# 운영 환경 이식

이제 개발 환경에서의 테스트가 끝났으니, OCI에 만들어둔 VM으로 해당 코드를 옮기려 한다.

<br>

## git push

.gitignore에 pyc 및 pycache 관련 내용을 입력하고 commit 후 github로 push 한다.

커밋 메시지 규칙은 아래의 내용을 따르기로 한다.

- [https://newkimjiwon.tistory.com/199](https://newkimjiwon.tistory.com/199)

github flow 등 branch 전략도 필요하지만 추후에 정리하고 추가하는 것으로 한다.

## 결과 폴더 구조

```text
├── backend
│   ├── main.py
│   ├── pyproject.toml
│   ├── README.md
│   ├── server
│   │   ├── __init__.py
│   │   └── app.py
│   └── uv.lock
├── frontend
│   ├── eslint.config.js
│   ├── index.html
│   ├── node_modules
│   ├── package-lock.json
│   ├── package.json
│   ├── public
│   ├── README.md
│   ├── src
│   │   ├── App.css
│   │   ├── App.jsx
│   │   ├── assets
│   │   ├── index.css
│   │   └── main.jsx
│   └── vite.config.js
└── README.md
```

## 운영 환경 구성

이제 원격 서버 환경으로 접속한 뒤 git pull로 소스코드를 내려 받고, 실행하고자 했다.

그러나 "VM.Standard.E2.1.Micro" 스펙의 VM으로는 git을 설치하다가 VM이 먹통이 되어버렸다.

그래서 개발 서버 구성시에 부족했던 부분들을 보완해서 다시 운영 환경 인프라를 구성하고 테스트 해봐야 할 것 같다.

