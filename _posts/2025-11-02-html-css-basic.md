---
title: "HTML, CSS 기초 정리"
classes: wide
categories:
  - html
tags:
  - html
  - css
---

<br>

# HTML

- HTML: 일종의 마크업(자료의 구조를 표현하기 위한) 언어

## HTML 파일의 템플릿

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Document</title>
  </head>
<body>
  <!-- body -->
</body>
</html>
```

- DOCTYPE: 문서의 버전과 형식을 명시하여 올바르게 렌더링 되도록 함
- html: html문서의 루트 요소를 정의함
- head: 문자 인코딩, 타이틀, 설명, 스타일시트, 스크립트 등 문서 정보(메타데이터)에 관련된 다양한 요소를 포함함.
- body: HTML 문서의 내용(contents)을 나타내는 태그

**올바른 작성법:** <br>
글자의 용도와 존재 목적을 바로 바로 파악할 수 있어야하므로 반드시 태그 안에 글자를 작성해야함<br>
{: .notice--info}

## 자주 사용하는 태그

- h: 제목을 표현하며, 숫자는 글자의 크기를 나타냄
- p: 본문의 약자로 글자를 표현하고 싶을때 사용함
- img: 이미지를 표현하고 싶을때 사용함
- button: 버튼
- ul: 순서가 없는 리스트, 하위 항목을 li 태그로 나타냄
- ol: 순서가 있는 리스트, 마찬가지로 하위 항목은 li 태그로 나타냄
- a: 링크에 사용하며, href 속성을 사용하여 링크를 지정함

## 스타일

style 속성은 여러개의 속성을 입력할 수 있음

### 자주 사용하는 스타일 속성

**가운데 정렬**

```css
display: block;
margin-left: auto;
margin-right: auto;
```

**글자 스타일**

```css
font-size: 16px;
font-family: 'gulim';
color: red;
letter-spacing: 1;
text-align: center;
font-weight: 100
```

**일부 글자에만 스타일을 주고 싶은 경우:** <br>
span 태그 안에 넣어서 태그에 스타일 속성을 주는 방식으로 사용할 수 있음<br>
{: .notice--info}

# CSS 파일

위에서의 스타일 속성들을 별도의 css 파일에 정의하여 사용할 수 있음

별도의 css 파일을 만든뒤, html 문서의 link 태그로 css 파일을 연계함

css 파일에는 스타일 속성을 작성 후 중괄호로 감싼 뒤 이름을 작성하고(이름은 .으로 시작함), 적용하고자 하는 html 태그에서 class 속성으로 이름을 입력함

**style.css**

```css
.title {
    font-size: 20px; 
    text-align: center; 
    font-family: AppleGothic
}
```

**index.html**

```html
<p class="title">스타일 적용</p>
```

**클래스 작명:** <br>
클래스 이름에 중복이 생기지 않도록 유니크하게 작성해야하며, 스타일은 보통 class로 정의함 <br>
{: .notice--info}

## div

박스를 집어 넣고 싶을 때 div 태그를 사용함

```css
width: 100px;
height: 100px;
background-color: cadetblue;
margin: 30px;
padding: 40px;
border: 4px solid black;
border-radius: 5px;
margin-left: auto;
margin-right: auto;
display: block;
```

- margin: 상하 좌우 여백
- padding: 박스 안쪽 여백
- border: 테두리 선의 두께, 종류, 색상
- border-radius: 박스의 테두리를 둥글게 만드는 정도
- display: block으로 설정시 가로행 전체 영역을 차지함

**레이아웃:** <br>
레이아웃 전체를 감싸는 박스를 만들어 두면 그 안의 박스가 삐져나오지 않도록 하므로 유용함 <br>
div는 기본적으로 display:block 속성을 가지므로 여러 박스를 한줄에 배치하기 위해서 float 속성을 사용하면 편함 <br>
float 사용 후 해제하기 위해서 다음 요소에는 clear: both와 float: none을 주는 것이 좋음 - 이 속성을 가진 별도의 div박스 추가하여 사용함 <br>
{: .notice--info}

## selector 문법

css 이름 뒤 공백(>) 후 태그로 selector 문법 적용 

- \> : 직계자식에만 적용함
- 공백 : 모든 자식에게 적용함
- input[type=text]: input 태그인데, type이 text인 것만 적용
- 콤마로 태그(클래스명)를 구분하면, 여러개의 태그에 적용 가능
- nth-child(숫자): 순서를 기준으로 selector를 적용하고 싶은 경우

**navbar 클래스 안에 있는 모든 li 태그에 적용**

```css
.navbar li {
  display: inline-block;
}
```

**레이아웃:** <br>
셀렉터를 너무 길게 작성하면 가독성이 떨어지므로, 어디에 적용되었는지 알기 쉽게 셀럭터의 적용 범위는 좁게 잡는 것이 좋음<br>
{: .notice--info}

## background

- 배경이미지 삽입시, background css파일에 정의 후, div 태그로 표현함
- 배경이미지는 기본적으로 꽉차지 않으면 반복됨 (background-repeat 속성으로 조정)
- filter 속성으로 배경이미지의 보정도 가능함

```css
.main-background {
  width: 100%;
  height: 500px;
  background-image: url(파일경로);
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
}
```

**margin collapse:** <br>
div박스가 2개 이상인데, 테두리가 겹치는 경우 margin도 합쳐지게 되어 함께 동작함 <br>
이 경우, 부모 박스에 padding을 주는 등 위쪽 테두리가 겹치지 않도록 변경하면 정상동작함 <br>
{: .notice--primary}

## position (좌표 이동)

- position 속성을 이용하면, margin을 주는 방법 외에 해당 아이템의 좌표 이동이 가능함 (float 처럼 공중에 떠있게 됨)
- position: fixed를 사용하면, 고정메뉴를 만들 수 있음
- position: absolute를 사용하면, 부모 요소 기준으로 고정점을 만들 수 있음

```css
.main-button {
  position: relative;
  top: 100px;
}
```

## z-index

- 공중에 떠있는 요소들이 많은 경우, 어떤 요소가 맨 앞에 와야하는지를 결정함
  - z-index (정수) 값이 높을 수록 앞에 오게 됨 

## 반응형 layout

- width 등에 고정된 pixel 값 대신 %를 사용하고, min-width, max-width 등의 속성을 함께 사용하면 반응형 layout을 만들기 좋음

**width:** <br>
padding, border 값들은 width와 관계 없음에 주의 <br>
box-sizing: border-box 를 주면, padding, border 등을 모두 포함한 영역에 적용하도록 (실제 눈에 보이는 영역) 할 수 있음 <br>
box-sizing: border-box, body - margin:0 등을 CSS 파일 최상단에 저장해두면 작업하기 쉬움 (normalize.css - 브라우저간 호환성 해결 코드) <br>
{: .notice--info}

## form

- action: form에 작성한 내용이 어디로 전달될지, method: 어떤 방식으로 전달될지 정의
- type: password, email, text 등 타입 정의, value: 기본 값, placeholder: 배경 글자
- select & option: drop-down 메뉴를 만들 수 있음
- textarea: 드래그 해서 원하는 크기로 만들 수 있음
- label 태그: input 태그의 id와 label 태그의 for 속성을 맞춰주면 글자를 선택해도 체크되는 효과가 생김

## table

- tr: 가로줄 만들 때 사용
- td: 세로줄 만들 때 사용 (th: 셀안에 있는 값이 굵어짐)
- tr을 먼저 만들고, tr 하위에 td를 만들어서 사용함
- thead: 제목행에 대한 분류 및 스타일링을 위해 사용
- tbody: 일반 행은 여기에 작성함
- 셀간 간격을 제거하고 싶은 경우, border-collapse: collapse로 줌
- colspan 속성: 셀 merge

**vertical-align:** <br>
inline/inline-block 요소간의 세로 정렬을 수행할때 사용함 (ex. 붙어있는 글자들간의 사이즈 크기가 있는 경우) <br>
{: .notice--info}

## pseudo-class

- cursor 속성: 마우스를 갖다 댈때 포인터 모양 변경
- hover: 마우스를 갖다 댈 때 스타일 속성 변경
- active: 클릭하는 중에 변경되는 스타일 속성
- focus: 커서가 찍혀있을때 스타일
- link: 방문하기 전 링크의 스타일 속성
- visited: 방문한 후 링크의 스타일 속성

```css
.btn:hover{
  background-color: chocolate;
}
```

**순서:** <br>
동시에 적용할때는 hover, focus, active 순서로 사용해야함 <br>
{: .notice--info}

## 클래스 작명 법

- (Object Oriented CSS) 공통 클래스(뼈대)와 개별 적용할 클래스(색상)를 각각 만들어서 클래스를 두개 적용하면 좋음: CSS 양이 줄어들고, 유지보수가 편리해짐
- (Block__Element--Modifier) 클래스 명을 작명할때 "chunk__역할--세부특징" 으로 작명하면 고민을 덜 할수 있음

**요즘:** <br>
요즘 React, Vue 등을 활용하여 개발하게 되면 Component 단위로 개발하게 되고, 하나의 CSS 파일에 전체 내용을 담지 않으므로 이러한 네이밍 컨벤션이 별로 의미가 없을 수 있음 <br>
{: .notice--info}