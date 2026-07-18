---
title: C++ — 리뷰용 핵심 포인터 · 참조 · 메모리 (파이썬 사용자용)
date: 2026-06-22 15:00:00 +0900
categories: [컴공, C]
tags: [cpp, 포인터, 메모리, 코드리뷰]
toc: true
---

C++ 특유의 요소와 함정만 정리해 둔다.

## 구조·타입 (뼈대)

- 모든 프로그램은 `main`이 필요. `#include`로 라이브러리(`<iostream>`·`<vector>`·`<string>`). 컴파일 `g++ a.cpp -o a.out` → 실행 `./a.out`.
- 입출력: `std::cout << x << "\n";` / `std::cin >> x;`.
- 정적 타입: `int`·`double`·`char`·`bool`(※ `bool`, boolean 아님)·`string`. `const`로 상수, 캐스팅 `(int)x`.

## 참조(&)와 포인터(*) — 파이썬엔 없는 핵심

C++는 변수가 실제 메모리 자리를 갖고, 그 자리를 직접 다루는 두 방법이 있다.

**참조(reference) `&`** — 기존 변수의 별명. 바꾸면 원본도 바뀌고, 한 번 묶이면 다른 걸 못 가리킨다.

```cpp
int a = 5;
int &r = a;      // r은 a의 별명
r = 9;           // a도 9가 됨
void swap(int &i, int &j) { ... }   // &로 받아야 실제 원본이 바뀜(값 전달은 사본이라 안 바뀜)
```

**포인터(pointer) `*`** — 값이 아니라 '메모리 주소'를 저장한다.

```cpp
int a = 5;
int* p = &a;      // &a = a의 주소. p는 그 주소를 담음
int x = *p;       // *p = 그 주소가 가리키는 값(역참조) = 5
int* q = nullptr; // 아무것도 안 가리킴
```

`&`는 선언에 쓰면 참조, 식에서 쓰면 '주소 연산자'(그 변수의 주소)다. 참조가 더 신식이고 대개 낫지만 포인터도 알아둬야 한다.

## 메모리 (new / delete)

C++는 GC(자동 회수)가 없어 동적 메모리를 직접 관리한다.

```cpp
int* arr = new int[n];   // 힙에 동적 할당
// ... 사용 ...
delete[] arr;            // 반드시 해제 — 안 하면 메모리 누수
```

`new`로 잡았으면 반드시 `delete`(배열은 `delete[]`). 소멸자(`~City() {}`)는 객체가 scope를 벗어날 때 자동 실행되어 정리를 맡는다.

## 함수·클래스 (선언 / 정의 분리)

- 함수는 **호출 전에 선언이 보여야** 한다. main 아래 정의하려면 위에 프로토타입: `void eat(int);`.
- 헤더 분리는 **필수가 아니다** — 한 파일이면 그 `.cpp` 안에 선언·정의를 다 넣어도 된다. 여러 `.cpp`로 나누거나 라이브러리로 낼 때만, 선언을 `.hpp`에 모으고 정의는 `.cpp`에 둔다. 쓰는 쪽은 `#include "fns.hpp"`, 컴파일 `g++ main.cpp fns.cpp`. (template·default 인자는 hpp에 둔다.)
- `overload`(같은 이름+다른 매개변수), `template<typename T>`(어떤 타입이든), default 인자(뒤쪽 매개변수부터), `inline`(작은 함수를 호출 자리에 삽입해 호출 비용 절약).
- class: `class City { int pop; public: void add(); };` — **끝에 `;` 필수**, 기본 접근은 private. 초기화 리스트 생성자 `City(int p): pop(p) {}`.

## 벡터 vs 배열

- 배열: 고정 크기. `int nums[4];` · `int nums[] = {1,2,3};` (대괄호가 변수 뒤 — 자바 `int[] a`와 반대).
- vector(가변, `<vector>`): `std::vector<int> a;` → `a.push_back(x)`(끝 추가)·`a[i]`·`size()`·`pop_back()`.

## 자주 틀리는 함정

- `=`와 `==` 혼동: `if (x = true)`는 컴파일은 되지만 대입이라 버그(찾기 어려움).
- `switch`는 정수만, `case`마다 `break`.
- 인플레이스 수정은 참조로 받아야 한다: `void f(std::string &text)`. 값으로 받으면 사본이라 원본이 안 바뀜.
- 함수 안에서 만든 배열을 반환하면 위험(스코프 벗어나면 사라짐) → 동적 할당(`new`)이나 vector 반환.
- 객체 벡터를 iterator로 돌 때 `(*it)->size()`처럼 먼저 역참조.
