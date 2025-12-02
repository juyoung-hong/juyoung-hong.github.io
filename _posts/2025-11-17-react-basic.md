---
title: "React 기초 정리"
classes: wide
categories:
  - frontend
tags:
  - react
---

<br>

# React 사용 이유

HTML, CSS, JavaScript 웹개발을 편리하게 도와주는 라이브러리

- Single Page Application : 앱처럼 부드럽게 동작하는 html 페이지를 만들고 싶을 때 사용함
- HTML을 함수나 object에 담아서 사용할 수 있으므로 재사용이 편리해짐
- React Native를 사용하면, 같은 문법으로 앱도 개발이 가능함

# 개발환경 세팅 및 프로젝트 생성

## 개발환경 세팅

1. Node.js LTS 버전 설치 (M1: ARM 버전으로 다운로드)
  - 본래 js는 브라우저에서만 실행가능하지만, Node.js를 통해서 PC에서 구동 가능
  - Node.js를 설치하면 npm(자바스크립트 라이브러리 관리 도구)이 함께 설치됨

## 프로젝트 생성

1. 작업 폴더 생성
2. 터미널 오픈 후 아래 명령어 수행
  - vite: 빌드\/번들링 툴 : 소스코드의 사이즈 압축 가능, 자바스크립트 변환 가능, 빠른 미리보기 지원

```zsh
npm create vite@latest # 리액트 프로젝트 생성
```

3. src 폴더 안에 코드 작성

```zsh
npm run dev # 미리보기
```

4. css 파일에 들어있는 기본적으로 들어가 있는 내용은 전부 지우고 아래 코드로 대입

```css
body {
  margin: 0;
}
div {
  box-sizing: border-box;
}
```

5. App.jsx 파일에서도 return 값에 div 태그 하나만 남기고 시작

## 프로젝트 구조

- App.jsx: 기본적으로 코드는 여기에 작성 (App.jsx -> main.jsx -> index.html 로 반영됨)
- public: 이미지, 데이터 등 파일 보관
- package.json: 내가 설치한 라이브러리를 보관하는 파일, npm 명령어 설정도 가능
- node_modules: 설치한 라이브러리들이 모여있는 폴더

# 레이아웃 만들기

## JSX 문법

1. class를 넣을땐 className으로 작성

```jsx
<div className="App"></div>
```

2. 데이터 바인딩에는 중괄호 사용

```jsx
let post = '제목';
<h4>{ post }</h4>
```

3. style 넣을때는 style={}로 사용, '-'이 포함된 경우 두 단어는 붙여쓰고 Camel Case 적용

```jsx
<h4 style={ {color: 'red'} }>{ post}</h4>
```

4. 태그 2개 이상 return 금지

5. 데이터를 잠시 저장하는 경우에는 변수 또는 state를 사용

  - 데이터 바인딩 된 값이 변경되면 변수는 반영되지 않지만, state는 변경을 감지하여, html을 자동 재 렌더링함
  - 자주 변경 될 것 같은 html 부분은 state로 만드는 것이 좋음

```jsx
import { useState } from 'react';

let [data, setdata] = useState('data');
```

* Destructuring 문법:

```jsx
let [a, b] = [1, 2];
```

6. OnClick 이벤트 핸들러 안에 함수 이름 입력

```jsx
<span onClick = { ()=>{ setdata(data+1) }}></span>
```

**array\/object 수정** <br>
- array 또는 object의 데이터를 수정할때 원본은 보존하고 copy 본을 만들어서 수정하는 것이 좋음<br>
- setdata함수는 기존 state와 변경할 state를 비교해서 같으면 변경해주지 않으며, array 또는 object에는 주소값만 저장됨, 데이터는 변경되었으나 주소가 변경되지 않아 동작하지 않을 수 있으므로 아래와 같이 코드 작성이 필요함<br>
{: .notice--info}

```jsx
<span onClick = { ()=>{ 
  let copy = [...data]
  copy[0] = 1;
  setdata(copy); 
  }}></span>
```

# Component

1. function 만들기
2. return() 안에 html 담기
3. <함수명><\/함수명> 사용하기

```jsx
// 1개 return 에는 1개의 div만 담고, 여러개 묶는 경우 fragment(<></>) 사용
function Modal(){
  return(
    <div calssName="modal">
    </div>
  )
}

// 사용
<Modal></Modal>
```

## Component로 만들면 좋은 것

1. 반복적인 html을 축약할 때
2. 큰 페이지들
3. 자주 변경되는 것들

state를 가져다 사용할때 컴포넌트는 문제가 발생할 수 있으므로 모든 것들을 컴포넌트로 만드는 것은 좋지 않음

## 동적인 UI 만드는 방법

1. html css로 미리 디자인 완성
2. UI의 현재 상태를 state로 저장
3. state에 따라 UI가 어떻게 보일지를 저장

```jsx
let [modal, setModal] = useState(false);

{
  modal == ture ? <Modal/> : null
}
```

# 반복문

## map

```jsx
[1,2,3].map(function(a, i){
  return <div key={i}>a</div>
})
```

반복문으로 html 생성하면 key={숫자}를 추가해줘야함

## for

```jsx
var array = [];
for (var i = 0; i < 3; i++>){
  array.push(<div>안녕</div>)
}
return(
  <div>
    { array }
  </div>
)
```

for 문법은 JSX 안에서 사용할 수 없으므로  바깥에서 사용해야함

# Props

- props: 부모 component에서 자식 component로 state를 전송할 수 있음 (state를 만드는 곳은 최상위 컴포넌트에서 만들어야함)
- Component의 수가 많아지면, props를 사용하기 귀찮아질 수 있음

```jsx
<Modal state={state}/>

function Modal(props){
  return(
    <div className="modal">
      <h4>{props.state}</h4>
    </div>
  )
}
```

# Input 태그

- onChange, onMouseover 등 이벤트 핸들러는 매우 많이 있음
- input에 입력한 값을 가져오려면 e 사용 (e.target.value)
- 이벤트가 상위 요소로 퍼지는 이벤트 버블링을 막고 싶으면 e.stopPropagation 사용
- state 변경 함수는 비동기 처리됨

# Class Component

- constructor, super, render 정의 필요
- state 변경시에는 this.setState({변경할 state}): 차이점만 변경함

```jsx
class Modal extends React.Component {
  constructor(props){
    super(props);
    this.state = {
      name : 'kim'
    }
  }
  render(){
    return (
      <div>이름 {this.state.name} </div>
    )
  }
}
```

# Build

- html 파일을 만들기 위해서 아래와 같은 명령어 실행 필요

```bash
npm run build
```