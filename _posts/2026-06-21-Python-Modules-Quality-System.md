---
title: 파이썬 모듈·코드품질·시스템
date: 2026-06-21 13:00:00 +0900
categories: [컴공, 파이썬]
tags: [python, 모듈, 테스트, 시스템]
toc: true
---

언어 내부(데이터모델·OOP) 다음은 "코드를 어떻게 묶고, 점검하고, 바깥과 잇는가"다. 프로젝트 구조와 import·테스트를 읽기 위한 모음.

## 파이썬은 인터프리터 언어다

컴파일 언어(C 등)는 컴파일러로 미리 머신코드로 바꿔 CPU에 의존한다. 파이썬은 한 줄씩 해석(interpreted)해 CPU 독립적이라 어디서나 돌지만 그만큼 느리다. 대신 표준 라이브러리 상당수가 C로 작성돼 빠르다.

```python
import array
array.array('i', [1, 2, 3])   # 정수만 담는 배열 — list 보다 메모리 효율적(표준 라이브러리 예)
```

## 모듈·import·시스템 연동

코드는 작은 모듈로 쪼개 묶고, 필요한 것만 import해서 쓴다. 그리고 모든 코드가 파이썬일 필요도 없다 — 더 큰 시스템의 한 조각이다.

```python
import random            # 모듈 전체
from math import gcd     # 일부만
import numpy as np       # 별칭(builtin·긴 이름 회피)
import sys; sys.path     # 모듈 탐색 경로 목록
```

`import x`는 `sys.path`를 뒤져 모듈을 찾고 **처음 한 번만** 실행·캐시한다(다시 import해도 재실행 안 함). 프로젝트마다 의존성·버전이 다르면 충돌 → **가상환경**으로 격리하고 `pip`로 설치한다(`requirements.txt`에 버전 고정).

시스템·외부 프로그램 호출도 표준 모듈로 한다:

```python
import sys
sys.argv                       # 명령행 인자 리스트 (argv[0] = 스크립트 파일명). 옵션·타입은 argparse 로 보완

import subprocess
subprocess.run(["ls", "-l"])   # 다른 언어·외부 프로그램 실행
```

## 추상 클래스로 인터페이스 강제 (abc)

추상 클래스 = 직접 객체로 못 만들고, 상속받은 자식이 특정 메서드를 **반드시 구현하게** 강제하는 틀. 여러 구현이 같은 메서드 이름(인터페이스)을 갖도록 못박을 때 쓴다.

```python
from abc import ABC, abstractmethod

class Strategy(ABC):               # 공통 틀(부모). Strategy() 로 직접 생성은 불가
    @abstractmethod
    def signal(self, price): ...   # 자식이 반드시 구현해야 하는 메서드

class Momentum(Strategy):
    def signal(self, price):       # 구현했으므로 OK
        return price > price.mean()

Momentum().signal(p)   # 정상 동작
Strategy()             # 에러: 추상 메서드라 인스턴스화 불가
# class Broken(Strategy): pass   →   Broken() 도 에러(signal 을 안 만듦)
```

왜 쓰나: 엔진이 전략 여러 개를 똑같이 `.signal()`로 호출할 수 있게 "이 인터페이스를 따르라"고 못박는 것. 백테스팅 엔진의 전략 베이스 클래스가 대표 예다.

## 코드 품질: 스타일·문서

PEP = 파이썬 개선 제안(규약). PEP8은 스타일(이름 규칙·들여쓰기·공백), PEP257은 docstring 규약. linter가 PEP8 준수를 자동 점검한다.

```python
def discount(price, rate):
    """할인가를 반환한다. rate 는 0~1."""   # docstring → help()·autodoc 으로 노출
    return price * (1 - rate)

def f(x: float, n: int) -> float:           # type hint: 동적 타입의 모호함을 줄임(강제는 아님)
    return x ** n
```

긴 문자열은 괄호로 묶어 여러 줄로 이어붙인다:

```python
msg = ("아주 긴 "
       "문자열도 "
       "이렇게 이어진다")
```

## 테스트

테스트 = 코드가 의도대로 도는지 자동 확인. pure function(입력·출력에만 의존, 부수효과 없음)이 테스트하기 쉽다.

```python
# 1) unittest: 함수를 예시 입력에 넣고 결과를 단언(assert)
import unittest
class TestSub(unittest.TestCase):
    def test_basic(self):
        self.assertEqual(subtract(54, 13), 41)
# 실행: python -m unittest

# 2) doctest: docstring 에 적은 예시를 그대로 실행해 검사
def subtract(a, b):
    """
    >>> subtract(54, 13)
    41
    """
    return a - b

import doctest
doctest.testmod()   # docstring 의 >>> 예시를 모두 실행 → 결과가 다르면 실패로 보고(맞으면 조용)
```
