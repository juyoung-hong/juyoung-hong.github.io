---
title: "HTML, CSS 중급 정리"
classes: wide
categories:
  - html
tags:
  - html
  - css
---

<br>

# 웹폰트

## PC에 설치된 font 첨부

```css
body {
  font-family: 'gulim', 'dotum';
}
```

안정성을 위해서 시스템에 설치되어있는 폰트를 여러개 작성하여 CSS 파일을 작성할 수 있음

## custom font 첨부

```css
@font-face{
  font-family: '폰트명';
  src : url(../font/gulim.ttf);
}

body {
  font-family: '폰트명';
}
```

가지고 있는 폰트 파일에 대해 작명하여, 설치되어 있지 않아도 css 파일에 사용할 수 있음

**한글 폰트:** <br>
기본적으로 여러개의 폰트를 등록할 수 있으나, 한글 폰트는 파일 용량이 매우 크므로 사이트 로딩 시간이 늘어나기도 함 - 따라서 한글 폰트는 최대 1-2개 쓰는 것을 권장함 <br>
용량을 줄이기 위해서는 web용으로 개발된 .woff 파일을 사용하는 것이 좋음 (ttf의 3분의 1수준) <br>
{: .notice--info}

- font-weight: 폰트 굵기 조정 (100 - 900), 굵기를 조정하는 것보다는 굵은 폰트를 따로 등록해서 사용하는 것이 좋음
- transform: rotate(0.03deg): 윈도우에서는 폰트를 아주 미세하게 회전시키면 부드러운 폰트를 얻을 수 있음

**Google Fonts:** <br>
- Google Fonts에서 제공하는 link 또는 import를 사용하면, 내가 폰트를 제공하는 것이 아닌 Google이 제공하므로 좀더 빠른 속도로 폰트를 제공할 수 있음 <br>
{: .notice--info}

# Flex-box

```css
.flex-container{
  display:flex;
}
```

```html
<div class="flex-container">
  <div class="flex-item">box1</div>
</div>
```

여러개의 박스들을 가로로 배치하고 싶을때, float나 inline block을 사용하는 방법 외에 display: flex를 부모에게 주면 자동으로 가로배치할 수 있음

- justify-content: 좌우 정렬 방법 (center, flex-end (우측 정렬), flex-start(좌측 정렬))
- align-items: 상하 정렬 방법 (center, flex-end (아래 정렬), flex-start(위 정렬))
- flex-direction: 배치 방법 (column(세로 배치), row(가로 배치))
- flex-wrap: wrap 폭이 넘치게 되면 아래로 보내고 싶을 때
- flex-grow: 박스 크기를 비율(배수)로 설정 가능

# Head 태그

1. CSS 파일: 링크 태그를 사용하여, CSS 파일 첨부가 가능함
2. style 태그: style 태그를 열어서 CSS를 작성할 수 있음
3. title: 사이트 제목 (브라우저의 탭에 뜨는 이름)
4. meta 태그:
  - charset="UTF-8"
  - name="구글 검색시 제목" content="검색에 도움을 주는 키워드"
  - name="viewport", width="device-width", initial-scale=1
5. open graph: 페이스북이나 카카오톡에 링크를 공유하면 생기는 박스를 커스터마이징 하는 도구
6. favicon: 웹사이트의 제목 옆에 뜨는 아이콘을 커스터마이징 (ico 파일이 가장 호환성이 좋음: 32x32 사이즈)

# 반응형 레이아웃

## 크기 단위

- vw: viewport width (브라우저 폭에 비례)
- vh: viewport height (브라우저 높이에 비례)
- rem: 기본 폰트사이즈에 비례 (html 태그 폰트 사이즈(16px)의 10배), 모든 곳에 rem 단위로 크기를 지정하면, 기본 font-size가 커질때 모두 같이 커질 수 있음 - 요즘은 잘 사용하지 않음
- em: 내 폰트사이즈의 X배

## 반응형 레이아웃

1. head 태그에 아래 내용 추가 필요

```html
<meta name="vieport" content="width=device-width, initial-scale=1.0">
```

2. CSS 파일에 아래 내용 추가

media query 문법: 브라우저 폭이 1200px 이하인 경우, 아래의 클래스명을 적용한다는 뜻 (여러개 사용하는 경우 가장 밑에 있는 것을 우선적으로 적용함)

media query는 항상 CSS의 가장 아래에 있어야 잘 작동함

부트스트랩 갖은 것들을 사용하면 반응형 레이아웃, 컴포넌트를 매우 빠르게 만들 수 있음

```css
@media screen and (max-width: 1200px){
  .main-title{
    font-size: 30px;
  }
}
```

**break point:** <br>
- break point 기준 px 값은 다른 사람들이 많이 사용하는 것으로 사용하는 것을 권장함 <br>
- 너무 많아지면 관리하기가 어려우므로 최대 5개 이내를 권장함 <br>
- 반응형 레이아웃을 만들때는 PC용을 먼저 만들면 쉬움 <br>
{: .notice--info}

# Font Awesome

- 이미지 대신 라이브러리를 사용하여 쉽게 아이콘 사용 가능

## 설치 방법

1. CDN
  - cdnjs 검색 후 all.min.css 파일을 html 파일 head 태그에 첨부하면 끝
  - 장점: 간편하고, 속도가 매우 빠름
  - 단점: 해당 사이트가 다운되면 우리 사이트도 다운될 수 있음

2. 직접 파일 다운로드 후 head 태그에 첨부

## 사용 방법

Font Awesome 사이트의 Icons 메뉴에서 아이콘을 검색하여 선택하고, html 예제 코드를 복사 후 첨부

해당 아이콘은 글자이기 때문에 font-size 등 style을 조정하면 아이콘의 스타일을 조정할 수 있음

# Transition 속성

1. 시작 스타일 제작
2. 최종 스타일 제작
3. 언제 변하는지 이벤트 확인
4. Transition으로 애니메이션 효과 주기

시작 스타일에 transition 속성 추가

```css
.start-style {
  opacity: 0;
  transition: all 1s;
}
.end-style:hover {
  opacity: 1;
}
```

마우스를 클릭했을 때 등은 css로 구현할 수 없으니, java script 등을 사용하는 것이 필요함

**transition 속도 조정:** <br>
- transition-timing-function을 통해서 애니메이션의 전환 속도를 조정할 수 있음 <br>
{: .notice--info}
