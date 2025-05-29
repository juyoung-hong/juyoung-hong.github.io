---
title: "Java 기초 정리"
classes: wide
categories:
  - java
tags:
  - java
---

<br>

# Java 환경 설치 (MacOS + VScode)

- 환경 설정 관련 참고 블로그: [링크](https://www.varofla.com/2c6307e1-a558-4e45-920e-4c054d75be0a)


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

## Java 실행

```java
public static void main(String[] args){
  // main 함수
}
```

- 예약어 - static : 메소드를 static으로 선언하면 객체를 생성하지 않아도 호출할 수 있음.
- 리턴 타입 - void : 메소드가 돌려줄 것이 없을 때 사용함
- 메소드 명 - main : 대/소문자 구분도 하기 때문에 반드시 이 이름을 사용해야 함
- 매개 변수 - String [] args : main 메소드에 전달되는 매개 변수는 반드시 String [] args 여야하며 변수명인 args만이 바뀔 수 있음

**main 함수:** <br>
실행을 목적으로 하는 모든 java 클래스는 main() 메소드가 반드시 있어야 함 <br>
main 메소드는 반드시 위와 같이 선언되어야 함 <br>
{: .notice--primary}

```zsh
# 컴파일 (test.class 파일 생성)
javac test.java
# 실행
java test.java
```

**주석 (comment):** <br>
//: 한줄 주석처리 <br>
/*  */: 블록 주석 <br>
/**  */: 문서용 주석 (클래스/메소드 선언 바로 앞에 있으면 문서용으로 인식됨) <br>
{: .notice--primary}

## 메소드

```java
public static void main(String [] args){
  System.out.println("Java");
}
```

- 제어자 (modifier) - public static : 메소드의 특성을 결정함
- 리턴 타입 (return type) - void : 메소드가 끝났을 때 돌려주는 타입
- 메소드 이름 (method name) - main : 메소드 이름 (소괄호 앞)
- 매개 변수 목록 (parameter list) - String [] args : 매개 변수 목록 (소괄호 안)
- 에외 목록 (exception list) : 소괄호의 끝과 중괄호 시작의 사이에 선언
- 메소드 내용 (method body) : 중괄호 안에 있는 내용들

## 클래스와 객체

```java
public class SmartPhone{
  boolean power;
  float height;
  float width;

  public SmartPhone{
    // 생성자 (constructor)
  }
  public void turnUp(){
    power = true; // 전원 켬
  }
  public void turnOff(){
    power = false; // 전원 끔
  }
}

SmartPhone iPhone = new SmartPhone;
SmartPhone galaxy = new SmartPhone;
```

**클래스와 객체:** <br>
각각의 실제 사물: 객체(Object), 인스턴스(instance) <br>
기본 생성자 (default constructor): 매개변수가 없는 생성자 (생략 가능) <br>
클래스만으로는 일을 할 수 없고, 객체를 생성해야만 일을 시킬 수 있음 <br>
{: .notice--primary}

**객체 명:** <br>
객체의 이름은 한단어 혹은 두단어로 되도록 간단하게 짓는 것이 좋음
길다면 소문자로 시작하고, 다음 단어의 첫문자만 대문자로 지정함
{: .notice--info}