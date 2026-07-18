---
title: 자바 — 리뷰용 핵심과 함정 (파이썬 사용자용)
date: 2026-06-22 14:00:00 +0900
categories: [컴공, 자바]
tags: [java, 함정, 코드리뷰]
toc: true
---

나는 파이썬을 주로 쓰지만 혹시 자바를 봐야 할 수 있어, 자바 특유의 함정 요소만 정리한다.

## 구조·타입 (뼈대)

- 모든 코드는 class 안에, 실행은 `main`에서 시작. 컴파일 언어(`javac Name.java` → `java Name`).
- 문장 끝은 `;`, 블록은 `{}`(파이썬 들여쓰기 대신).
- 정적 타입: 선언할 때 타입을 지정한다. `int`(±약 21억)·`double`·`boolean`(소문자 true/false)·`char`(작은따옴표)·`String`(큰따옴표). `final` = 상수(재할당 불가).
- `int / int`는 정수 나눗셈(`7 / 2 == 3`) — 한쪽이 double이라야 실수.
- `'a'`는 char, `"a"`는 String (파이썬은 둘이 같음).

## == vs .equals (자바 최대 함정)

기본형(int·double 등)은 `==`가 값 비교지만, **객체(String 등)는 `==`가 참조(주소) 비교**다. 내용이 같은지는 `.equals()`.

```java
String a = "hi", b = new String("hi");
a == b          // false (서로 다른 객체)
a.equals(b)     // true  (내용은 같음)
```

문자열 비교에 `==`를 쓰는 건 대표적 버그다. 또 `double`은 부동소수 표현 오차로 `==`가 실패할 수 있다: `0.1 + 0.2 == 0.3` → false. 그래서 실수 비교는 허용 오차(epsilon)를 둔다: `Math.abs(a - b) < 1e-9`. 돈처럼 정확한 십진 계산이 필요하면 `BigDecimal`을 쓴다. (이 부동소수 문제는 자바만이 아니라 파이썬 등 대부분 언어 공통이다.)

## 배열 vs ArrayList

- **배열**: 고정 크기. `int[] a = new int[4];` 또는 `{1,2,3}`. 길이는 `a.length`(필드라 `()` 없음). 그냥 출력하면 주소 → `Arrays.toString(a)`, 내용 비교는 `Arrays.equals(a, b)`.
- **ArrayList**: 가변. `ArrayList<Integer> a = new ArrayList<>();` 정수는 래퍼 타입 `Integer`(autoboxing). `add(x)`·`get(i)`·`set(i,x)`·`size()`로 다룸(`a[i]` 인덱싱 안 됨).
  - 함정: `remove(int)`은 **인덱스** 제거, `remove(Integer)`는 **값** 제거. 값으로 지우려면 `remove(Integer.valueOf(x))`.

## 컬렉션·객체 기타

- Map/Set: `HashMap<K,V>`·`HashSet<E>`. 선언은 반드시 구현체로 `new HashMap<>()`. `put/get/containsKey`·`add`.
- 순회 중 제거는 `Iterator`(`it.remove()`)로 안전하게(for로 돌며 지우면 인덱스가 어긋남).
- String은 불변 — 메서드(`substring`·`toUpperCase`·`replace`)는 원본을 안 바꾸고 새 값을 반환하니 받아야 한다.
- 객체를 그냥 출력하면 `클래스명@주소` → `toString()` 오버라이드(파이썬 `__str__`에 해당).
- `static`: 객체 없이 클래스에 속함(`Math.min`, `Integer.parseInt`).
- `int`는 null 불가, `Integer`는 null 가능(값 없음 표현).

## 자주 틀리는 함정

- `switch`는 `case`마다 `break`가 없으면 아래로 줄줄이 흘러간다(fall-through).
- 거듭제곱 연산자가 없다 → `Math.pow(a, b)`(double 반환). `Math.*`는 반환값을 받아야 한다.
- `split(" ")`는 연속 공백에서 빈 토큰이 생김. `split`에 `|` 등 메타문자는 `\\|`로 이스케이프.
- for-each는 String에 직접 못 씀 → `toCharArray()`. `"x" * 3` 같은 문자열 반복도 안 됨.
- 증감 `x++` / `++x` 존재(파이썬엔 없음). 삼항 `a = cond ? x : y;`.
