---
title: "Java 기초 정리"
classes: wide
categories:
  - java
tags:
  - java
---

<br>

# Java 기초 문법

```java
public class DoorLockManager {
  String currentPassword;
  public boolean checkPassword(String password){
  // 내용
  }
}
```

- 접근 제어자 (access modifier) - public : 접근할 수 있는 범위를 제한함
- 클래스 이름 (class name) - DoorLockManager : 클래스의 이름
- 변수 (variable) - currentPassword : 클래스의 특성을 결정 짓는 "상태"
- 메소드 (method) - checkPassword : 값을 주고 결과를 넘겨주는 것
- 매개변수 (parameter) - password : 넘겨주는 값으로, 없어도 되고 여러개가 와도 됨
- 리턴 타입 (return type) - boolean : 자료형
- 주석 (comment) - // 내용 : 어떤 줄을 실행하지 않고자 할 때 사용함

**클래스와 메소드:** <br>
java의 메소드는 반드시 클래스 안에 포함되어야 함 <br>
클래스는 상태(state)와 행동(behavior)이 있어야 함 <br>
{: .notice--primary}

# Java 환경 설치 (MacOS + VScode)

- 환경 설정 관련 참고 블로그: [링크](https://www.varofla.com/2c6307e1-a558-4e45-920e-4c054d75be0a)

## Java 실행

```java
public static void main(String[] args){
  // main 함수
}
```

- 예약어 - static : 메소드를 static으로 선언하면 객체를 생성하지 않아도 호출할 수 있음.
- 리턴 타입 - void : 메소드가 돌려줄 것이 없을 때 사용함
- 메소드 명 - main : 대/소문자 구분도 하기 때문에 반드시 이 이름을 사용해야 함
- 매개 변수 - String [] args : main 메소드에 전달되는 매개 변수는 반드시 String [] args 여야함

**main 함수:** <br>
실행을 목적으로 하는 모든 java 클래스는 main() 메소드가 반드시 있어야 함 <br>
main 메소드는 반드시 위와 같이 선언되어야 함 <br>
{: .notice--primary}