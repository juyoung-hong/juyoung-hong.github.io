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
기본 생성자 (default constructor): 매개변수가 없는 생성자 (생략 가능) → new 라는 예약어로 객체를 생성할때, 생성자가 호출됨 <br>
클래스만으로는 일을 할 수 없고, 객체를 생성해야만 일을 시킬 수 있음 <br>
{: .notice--primary}

**객체 명:** <br>
객체의 이름은 한단어 혹은 두단어로 되도록 간단하게 짓는 것이 좋음 <br>
길다면 소문자로 시작하고, 다음 단어의 첫문자만 대문자로 지정함
{: .notice--info}

## 변수

java는 총 4가지의 변수를 가짐

```java
public class VariableTypes {
  int instanceVariable;
  static int classVariable;
  public void method(int parameter){
    int localVariable;
  }
}
```

- 지역 변수 (local variables) : 중괄호 내에서 선언된 변수 (지역 변수를 선언한 중괄호 내에서만 유효함)
- 매개 변수 (parameters) : 메소드에 넘겨주는 변수 (메소드가 호출될 때 생명이 시작되며, 메소드가 끝나면 소멸됨)
- 인스턴스 변수 (instance variables) : static이라는 예약어가 없이 메소드의 밖에, 클래스의 안에 선언된 변수 (객체가 생성될 때 생명이 시작되고, 그 객체를 참조하는 다른 객체가 없으면 소멸 됨)
- 클래스 변수 (class variables) : static이라는 예약어가 있는 메소드 밖에 클래스 안에 선언된 변수 (클래스가 처음 호출될 때 생명이 시작되고, 자바 프로그램이 끝날 때 소멸됨)

**Garbage collector:** <br>
C 또는 C++에서는 할당(allocation)을 한 변수에 어떤 값을 지정하면, 필요없다고 개발자가 지정하고, 바로 메모리에서 지울 수 있음 <br>
그러나 java는 가비지 콜렉터가 알아서 메모리를 청소하므로 필요없으니 메모리에서 지워달라고 할 수 없음 <br>
{: .notice--info}

**변수 이름 규칙:** <br>
보통은 첫 문자는 소문자로 시작하는 단어로, 두 번째 단어부터 첫 문자만 대문자로 시작하면 됨 <br>
상수(절대 변하지 않는 값)의 경우는 모두 대문자로 지정하며, 단어와 단어 사이에는 _로 구분함 <br>
{: .notice--primary}

## 자료형

- 기본 자료형 (Primitive data type) : new를 사용하지 않고 바로 초기화가 가능한 경우 (추가로 개발자가 만들어낼 수 없음)
  - 정수형 : byte(signed 8bit), short, int, long, char(unsigned)
  - 소수형 : float(32 bit), double(64 bit)
  - 기타 : boolean
- 참조 자료형 (Reference data type) : new라는 예약어를 사용해서 초기화 하는 경우 (String은 예외)

**Long Type:** <br>
기본적으로 java에서는 숫자를 int로 생각하므로, long 타입의 숫자는 숫자 가장뒤에 L을 붙여줘야 함 <br>
{: .notice--info}

**소수점:** <br>
float, double은 값의 범위를 넘어서만 정확성을 보장하지 못하므로 돈계산시에 사용해서는 안됨 → java.math.BigDecimal 클래스를 사용해야만 함 <br>
일반적으로는 소수점 처리시 double을 많이 사용하며, 소수점 자리수가 적은 경우 float를 사용함 <br>
{: .notice--info}

```java
// char를 지정하는 방법
char value = 'a'; // 홑따옴표 안에 문자를 바로 대입
char value = '\u0097'; // \u와 16진수 값을 대입
char value = 999; // 해당 값의 유니코드 번호를 지정
```

**기본 자료형의 기본 값:** <br>
지역 변수로 기본 자료형을 사용하는 경우에는 반드시 값을 지정해야 함 <br>
만약 그 변수의 값이 기본 값이라고 하더라도, 명시적으로 기본값을 지정해 주는 것이 좋음 <br>
char를 제외한 모든 숫자의 기본값은 0이며, char는 \u0000 (공백)이 기본값으로 사용됨 <br>
{: .notice--primary}

## 연산자

### 연산자 (피 연산자가 2개인 경우)

- +(additive operator) : 더하기
- -(subtraction operator) : 빼기
- *(multiplication operator) : 곱하기
- /(division operator) : 나누기 (계산하는 두 값이 정수형이더라도 결과가 소수형이면 알아서 변환해주지 않으므로 주의해야함)
- %(remainder operator) : 나머지 

### 단항 연산자 (피 연산자가 1개인 경우)

- +(Unary plus operator) : 단항 플러스 연산자 (변수 x 1을 의미: 숫자가 양수임을 명시적으로 보여줄때 사용)
- -(Unary minus operator) : 단항 마이너스 연산자 (변수 x -1을 의미)
- ++(Increment operator) : 증가 연산자 (변수 + 1을 의미: 변수의 앞/뒤 순서에 유의)
- --(Decrement operator) : 감소 연산자 (변수 - 1을 의미: 변수의 앞/뒤 순서에 유의)
- !(Logical complement operator) : 논리 부정 연산자
- ~(tilde) : 비트 값의 0을 1로, 1을 0으로 바꾸는 연산자

### 삼항 연산자

- 조건식 ? true일때 값 : false일때 값 (Conditional operator) : if 문장을 간단하게 처리함 (변수에 값을 할당할 때 사용함)

**비교 연산자:** <br>
참조 자료형에 대한 비교는 그 주소값이 같은지 확인함 <br>
&&은 좌측 연산이 false이면, 우측 연산을 수행하지 않으므로 &와 차이가 있음: 꼭 필요한 경우를 제외하고는 &&와 ||을 사용할 것을 권장 <br>
{: .notice--primary}

**형 변환:** <br>
서로 다른 타입 사이에 변환하는 작업 (casting) <br>
boolean type은 숫자로 변활할 수 없기 때문에 형 변환이 불가능함 <br>
기본 자료형과 참조 자료형 사이의 형변환도 기본적으로 불가능함 <br>
형변환을 할 때 범위가 더 큰 타입으로 변환하면 아무 문제가 없지만, 범위가 작은 타입으로 변환하는 경우는 꼭 생각해보고 변환해야함 <br>
{: .notice--primary}

## 조건문

```java
if(boolean값1){
  처리문장1;
} else if(boolean값2) {
  처리문장2;
} else {
  처리문장3;
}
```

가독성을 높이기 위해서 단지 하나의 문장만 실행하더라도 중괄호를 열고 닫는 것이 좋음

if문의 조건에 여러가지 조건을 한번에 따지기 위해서는 &&나 \|\|을 활용하면 됨

if 문은 두가지 이상의 값을 비교하거나, true/false 여부를 확인하고자 할때 많이 사용하며, 하나의 값이 여러 범위에 걸쳐서 비교되어야 하는 경우에는 하나의 값으로 분기하여 비교하는 switch 문을 사용하는 것이 좋음

```java
switch(비교대상변수){
  case 점검값1:
    처리문장1;
  break;
  default:
    기본처리문장;
  break;
}
```

- 비교 대상 변수: long을 제외한 정수형과 몇가지 특별한 타입만 들어갈 수 있음
- 점검 값: 중괄호 안에는 case 문이 오거나 default가 나와야 하며, 각 case를 마무리하고 싶다면 반드시 break를 추가해야함 (default는 앞에 있는 조건에 맞지 않는 경우에 수행 됨)

**switch:** <br>
switch에서는 한번 조건을 만족시키면, break가 올때까지 어떤 case가 오든지 상관하지 않고 계속 실행하므로 반드시 case에 대한 처리가 끝나면 break를 붙여주는 습관을 들여줘야 함 <br>
default는 원하지 않는 결과가 나올 수 있으므로 맨 마지막에 넣는 것을 권장함 <br>
숫자 비교시에는 적은 숫자부터 증가시켜 나가는 것이 좋음 <br>
{: .notice--primary}

## 반복문

### while문

```java
while(boolean조건){
  처리문장;
}
```

- break: 현재 수행중인 중괄호에서 빠져나오기 위해 사용
- continue: 그 뒤에 있는 문장은 건너 뛰고, boolean 조건 점검 부분으로 다시 가라는 의미

```java
do{
  처리문장;
} while(boolean조건);
```

do-while문은 적어도 한번은 반복 문장이 실행 됨: 한번은 꼭 실행시키고 싶을 때 사용함

### for문

while문의 경우 잘못 사용하면 무한 루프에 빠지기 쉬우므로, for 루프를 선호하는 개발자들이 많음

```java
for (초기화; 종료조건; 조건값 증가){
  반복문장;
}
```

**label:** <br>
for 루프를 두개 이상 쓰거나, while 루프를 두개 이상 사용할 경우, 바깥쪽 루프의 시작점으로 이동하려고 할 때 label을 사용함 <br>
{: .notice--info}

## 배열

- 배열: 한가지 타입에 대해서 하나의 변수에 여러개의 데이터를 넣을 수 있음 (배열의 index는 0부터 시작함)
- 배열은 지역변수라고 하더라도 배열의 크기만 정해주면, 초기값을 초기화 하지않아도 기본값이 할당된 채로 사용할 수 있음
- 얼마나 자주 사용하는지, 어디에서 사용하는지를 확인하여 메소드에서 선언할지 또는 클래스의 인스턴스 변수로 선언할지 결정하면 됨 (static 변수를 사용하면 객체를 생성할때마다 인스턴스 변수를 새로 만들지 않음)

```java
int [] lottoNumbers = new int[7]; // 대괄호는 타입과 변수 사이에 위치해도 됨 (많이 사용하는 방식)
int lottoNumbers[]; // 변수명 뒤에 위치해도 됨
String [] strings=null; // 배열과 같은 참조 자료형을 선언할때 명시적으로 무소유의 상태를 선언할 수도 있음 (사용시에는 반드시 초기화 후 사용해야함)
int lottoNumbers = {5, 12, 23, 25, 38, 41, 2}; // 중괄호를 사용하는 경우에는 한번에 변수선언 및 초기화가 이루어져야함 (보통 절대 변경되지 않는 값을 지정할때 이렇게 사용함)
```

**toString:** <br>
참조 자료형은 public String toString()이라는 메소드를 만들어줘야만 원하는 내용이 출력됨 <br>
그렇지 않으면 "타입이름@고유번호"로 내용이 출력됨 <br>
{: .notice--info}

### 2차원 배열

```java
int [][] twoDim = new int[2][3]; // twoDim[0]은 int가 아닌 배열임, twoDim[0][0]이 int임.
twoDim = new int[2][]; // 1차원 크기만 지정하고, 2차원 크기를 지정하지 않을수도 있음. 이와같이 선언하면 2차원 배열의 공간의 크기를 서로 다르게 지정할 수 있음
twoDim[0] = new int[3];
twoDim[1] = new int[2];
int [][] twoDim = { {1, 2, 3}, {4, 5, 6} }; // 2차원 배열 선언 및 초기화
```

### 배열의 길이

```java
oneDim.length // 배열의 길이는 .length로 알 수 있음
twoDim[0].length
twoDim[1].length // 2차원 배열의 길이는 각 1차원 배열에 .length로 알 수 있음
```

- 배열의 값을 출력하고자 하는 경우, for 또는 while 문을 사용하여 수행함
- .length를 사용하면 배열의 길이를 하드코딩하지 않고 유연하게 가져올 수 있으나, for 문 안에 직접 넣는 것은 성능적 측면에 효과적이지 않으므로 아래와 같이 수행하는 것이 권장됨

```java
int twoDimLength = twoDim.length;
for (int oneLoop=0;oneLoop<twoDimLength;oneLoop++){
  int twoDimOneLength=twoDim[oneLoop].length;
  for(int twoLoop=0;twoLoop<twoDimOneLength;twoLoop++){
    System.out.println("twoDim["+oneLoop+"]["+twoLoop+"]="+twoDim[oneLoop][twoLoop]);
  }
}
```

### Collection을 위한 for Loop

```java
for(타입이름 임시변수명: 반복대상객체){
  // 반복문
}
for (int[] dimArray:twoDim){
  for(int data:dimArray){
    System.out.println(data);
  }
}// 이렇게 사용하면 편리하지만, index를 알 수 없는 단점이 있어 위치를 확인하고자 하는 경우에는 임시 변수를 둬야함 (세미콜론이 두개 있는 for루프 사용)
```

### 자바 실행시 원하는 값을 넘겨주기

클래스 이름 뒤에 공백으로 분리한 문자열을 나열하면, 이 문자열들이 args라는 배열에 전달됨

```bash
java ArrayMain a b c d
```
