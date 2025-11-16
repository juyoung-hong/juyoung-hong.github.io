---
title: "HTML, CSS 고급 정리"
classes: wide
categories:
  - html
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
