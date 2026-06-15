---
title: "Coding Test 공부 - Java"
classes: wide
categories:
  - codingtest
tags:
  - java
---

<br>


# 배열의 선언 및 초기화

```java
int[] answer = new int[2];
answer[0] = a;
answer[1] = b;

# 또는
int[] answer = {};
answer = new int[] {a, b};

# 또는
int[] answer = new int[] {a, b};
```

# 배열의 평균값

```java
import java.util.Arrays;

Arrays.stream(numbers).average()
  .orElse(0);
```

- Arrays.stream(numbers).average():
  - numbers의 평균을 구합니다.
- .orElse(0):
  - null인 경우, 0을 리턴합니다.

# LongStream

```java
import java.util.stream.LongStream;

LongStream.range(1, num_list.length + 1)
    .mapToInt(i -> num_list[(int) (num_list.length - i)])
    .toArray();
```

- LongStream.range(1, num_list.length + 1):
  - 1부터 배열의 길이(num_list.length)까지의 숫자 시퀀스를 생성합니다.
- .mapToInt(i -> num_list[(int) (num_list.length - i)]):
  - 생성된 각 숫자 i를 사용하여 배열의 인덱스에 접근합니다.
- .toArray():
  - 위의 과정을 통해 수집된 요소들을 다시 int[] 배열 형태로 변환하여 반환합니다.

# String

```java
String answer = "";
for (int i=0; i < my_string.length(); i++){
    answer += my_string.charAt(my_string.length()-(i+1));
}
```

- String은 []로 접근할 수 없고, ~번째 글자는 charAt() 메서드를 이용함
- length도 멤버가 아닌 메서드로 호출해야함
- StringBuilder를 이용하면 append, reverse가 쉬움

```java
import java.util.*;

StringBuilder(myString).reverse().toString();
```