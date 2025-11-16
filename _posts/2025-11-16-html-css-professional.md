---
title: "HTML, CSS 고급 정리"
classes: wide
categories:
  - frontend
tags:
  - html
  - css
---

<br>

# Pseudo element

```css
.pseudo::first-letter {
  color: red;
}
```

- pseudo-class: 다른 상태 일때 (마우스를 위에 올리거나 ...)
- pseudo-element: 내부의 일부분에 대해서 스타일을 줄 때  (첫줄, 첫글자 등)
  - after \/ before: 내부에 있는 글자 중 맨 뒤 \/ 맨 앞에 무엇인가를 추가 할 때 사용함 (float clear 등)
  - file upload에 사용하는 input 태그 등 숨겨진 요소에 대한 스타일링에도 사용함

**pseudo 수식어** <br>
- 브라우저 마다 shadow-dom의 이름이 조금씩 다름 <br>
- webkit: chrome, safari, edge 등 적용 <br>
- moz: firefox에서 적용 <br>
- ms: internet explorer에서 적용 <br>
{: .notice--info}

# SASS

- CSS에 몇가지 문법을 더한 언어 (조건문, 반복문, 함수, 변수 등)
- 확장자: .scss, .sass (scss -> css 변환이 필요함)

**map 파일** <br>
- 디버깅 용도의 파일로 scss상의 오류에 대해서 확인 가능 <br>
{: .notice--info}

## 변수

### SASS

```scss
$main-color: #2a4cb2;
$default-size: 16px;

.background {
  background-color: $main-color;
  font-size: $default-size - 2px;
}
```

### CSS

```css
:root {
  --main-color: red;
}

.background {
  background-color: var(--main-color);
  font-size: calc(40px - 20px);
}
```

CSS를 사용하더라도 위의 문법으로 변수, 계산 등 가능함

## nesting 문법

```scss
.main-bg {
  h4 {
    font-size: 16px;
  }
  button {
    color: red;
  }
}
```

main-bg 안에 있는 h4, button 등 관련있는 class 들을 한번에 묶어서 사용할 수 있음

## extend 문법

- 중복이 발생하는 코드에 대해서 변수로 치환하고 재사용

```scss
%btn {
  width: 100px;
  height: 100px;
}

.btn-green {
  @extend %btn;
  color: green;
}
```

퍼센트 기호로 시작하는 임시 변수에 재사용할 속성을 정의한 후, 재사용 가능

## mixin 문법

```scss
@mixin head($fs, $attr){
  font-size:$fs;
  #{ $attr }: -1px;
}

h2{
  @include head(40px, letter-spacing)
}
```

- 긴 코드를 가변적으로 축약할 때 mixin 문법을 사용함

## use 문법

### SASS

```scss
@use 'default';

h3 {
  color: default.$main-color;
}
```

- 긴 코드를 다른파일에 미리 정의해 놓고, import 하고 싶은 경우 사용함 (여러 파일에 공통으로 사용하는 것들은 미리 따로 정의해놓고 가져와서 사용 가능)
- 다른 파일에 종속되는 default 파일은 컴파일 할 필요 없으므로 파일 명이 \_로 시작하도록 작명함

### CSS

```css
@import;
```

# 영상과 소리

## video

```html
<video controls>
  <source src="영상 경로.webm" type="video/webm">
  <source src="영상 경로.mp4" type="video/mp4">
</video>
```

source 태그를 사용하고, 용량이 더 적은 영상부터 위에 작성하면 위에서 부터 순차적으로 실행하며 브라우저에서 호환 가능한 영상을 선택하므로 보다 안정적으로 서비스 할 수 있음

- video 태그 속성
  - autoplay muted: 자동 재생
  - preload="auto": 미리 다운로드 받기 (none \/ auto \/ metadata)
  - poster: 썸네일 이미지
  - loop: 무한 반복 재생

## audio

```html
<audio controls>
  <source src="경로.mp3">
</audio>
```

- audio 태그 속성
  - muted: 무음모드
  - autoplay: 자동재생 (적용 안됨)
  - preload: 미리 로딩

# Animation

keyframes를 활용하여 좀 더 복잡한 애니메이션을 구현할 수 있음

```css
.ani-text:hover {
  animation-name: left-and-right;
  animation-duration: 1s;
}

@keyframes left-and-right{
  0%{
    transform: translateX(0px);
  }
  50% {
    transform: translateX(-100px);
  }
  75%{
    transform: translateX(100px);
  }
  100% {
    transform: translateX(0px);
  }
}
```

- transform 속성
  - rotate: 회전
  - translateX: x축 좌표 이동
  - translateY: y축 좌표 이동
  - scale: x배 확대
- animaion 속성
  - animation-delay: 딜레이 양
  - animation-iteration-count: 반복 횟수
  - animation-timing-function: 속도 조절

**transform과 성능** <br>
- 브라우저가 html, css를 렌더링 하는 순서는 다음과 같음: 1. Render Tree 만들기 2. Layout 잡기 3. Paint 칠하기 4. Composite 처리 <br>
- translateX가 아닌 margin을 조정해도 동작은 하지만, margin은 2번 단계에 해당하므로 변경되면 3, 4번을 다시 처리해야함 <br>
- 이에 반해 translate는 4번 단계에서 처리하므로 변경되더라도 4번만 다시 수행하면 되기때문에, translate로 애니메이션을 구현하는 것이 성능에서 유리함 <br>
- Composite 단계의 애니메이션 속성들은 다른 thread에서 처리하므로 속도면에서 더 유리할 수 있음 <br>
{: .notice--info}

# display: grid

조그만 부분에 규칙적인 레이아웃을 잡고자 할때 grid가 유용하게 사용될 수 있음

```html
<div class="grid-container">
  <div class="grid-nav"></div>
  <div class="grid-sidebar"></div>
</div>
```

```css
.grid-container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  grid-template-rows: 100px 100px;
  grid-gap: 10px;
  grid-template-areas:
    "header header header header"
    "side . . . "
    "side . . . "
}
.grid-container div {
  border: 1px solid black;
}
.grid-nav {
  grid-column: 1/ 4;
}
.grid-sidebar {
  grid-row: 2 / 4;
}
.grid-nav {
  grid-area: header;
}
.grid-sidebar {
  grid-area: side;
}
```

# position: sticky

스크롤을 내리더라도 화면에 고정할 수 있는 옵션 (조건부로 position: fixed가 적용될 수 있음)

```css
.image {
  position: sticky;
  top: 100px;
}
```