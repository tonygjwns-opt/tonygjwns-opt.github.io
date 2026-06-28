---
title: 파이썬 데이터모델·함수·OOP
date: 2026-06-21 11:00:00 +0900
categories: [컴공, 파이썬]
tags: [python, oop, 함수형, 데이터모델]
toc: true
---

기초 문법 너머, 파이썬 코드를 "왜 이렇게 동작하나" 수준으로 읽기 위한 모델. 변수·함수·클래스가 내부에서 어떻게 객체로 다뤄지는지를 모으면 남의(또는 AI의) 코드가 보인다.

## 변수는 값이 아니라 이름이다

파이썬 변수는 객체를 담는 상자가 아니라 객체에 붙인 **이름표**(reference)다. `y = x`는 x가 가리키던 그 객체에 이름표 y를 하나 더 붙이는 것(복사 아님).

여기서 "그럼 y를 바꾸면 x도 바뀌나?"가 헷갈리는데, 답은 **무엇을 하느냐**에 달렸다. 두 동작을 갈라야 한다.

- 재바인딩 `y = 새값` : 이름표 y를 떼서 **다른 객체**에 붙인다. x의 이름표는 그대로 → x 안 바뀜. (항상)
- 변형(mutate) `y.append(...)`, `y[0]=...` : **객체 자체의 내용**을 고친다. 같은 객체에 x도 붙어 있으니 x로 봐도 바뀜.

```python
# (1) immutable(int·str·tuple): 변형 자체가 불가능 → 재바인딩만 가능 → x 절대 안 바뀜
x = 5
y = x
y = y + 1        # 재바인딩(6 이라는 새 int)
# x = 5,  y = 6

# (2) mutable 인데 '재바인딩': 역시 x 영향 없음
x = [1, 2]
y = x
y = y + [3]      # 새 리스트로 갈아끼움
# x = [1, 2],  y = [1, 2, 3]

# (3) mutable 을 '변형': 이때만 x 도 바뀜 (같은 객체를 고쳤으니까)
x = [1, 2]
y = x
y.append(3)      # 같은 리스트를 제자리 수정
# x = [1, 2, 3],  y = [1, 2, 3]      ← y is x 는 True
```

정리: "y 바꿨는데 x도 바뀜"은 (3)처럼 **mutable을 변형할 때만** 일어난다. `int·str·tuple`은 변형 자체가 안 되니 이런 일이 없다. 즉 구분 축은 **자료형이 아니라 재바인딩이냐 변형이냐**다(자료형은 "변형이 가능한지"만 결정한다).

이름 탐색 순서는 LEGB: Local → Enclosing → Global → Builtin.

```python
a == b   # 값이 같은가
a is b   # 같은 객체인가(메모리 정체성) — None 비교에만 쓰고 나머지는 ==
False == 0     # True   (bool 은 int 의 하위 타입)
False == ""    # False
```

## 가변성과 hashable

mutable = 만든 뒤 내용을 바꿀 수 있는지. `list·dict·set`은 mutable, `int·str·tuple`은 immutable. 내부가 전부 immutable이면 hashable → `dict` 키·`set` 원소로 쓸 수 있다(해시값이 안 변해야 하므로).

```python
{(1, 2): "ok"}   # 튜플 키 OK
{[1, 2]: "no"}   # 리스트는 unhashable → 에러
# tuple 은 top-level 만 immutable — 안에 list 를 품으면 그 튜플도 unhashable
```

## 컬렉션의 공통 골격

추상적으로 컬렉션은 세 성질로 본다: Sized(`len`), Iterable(하나씩 꺼냄), Container(`in` 판정). 분류는 순서·매핑 여부로:

```python
# Set      순서·중복 없음. 집합연산 a|b  a&b  a-b  a<=b(부분집합). discard()는 없어도 에러 X
# Sequence 순서 있음(list·tuple·str). 인덱싱·슬라이싱
# Mapping  key→value(dict). get(k, default)로 KeyError 회피
{}              # 빈 dict
set()           # 빈 set
frozenset([1])  # immutable set → 이건 hashable
```

## 참/거짓과 None

```python
if x:   # None·0·0.0·""·[]·{} 은 False, 나머지는 True (truthiness)
```

None은 "값이 없음"을 나타내는 유일 객체(False와도 다름). 함수가 명시적 `return` 없이 끝나면 None을 반환한다.

## 함수: 객체이고, 인자는 객체 참조로 넘어간다

함수도 id를 가진 객체다 → 변수에 담고, 인자로 넘기고, 반환할 수 있다.

```python
f = compute              # 같은 함수 객체 (f is compute → True)
list(map(len, words))    # 함수를 인자로
```

함수 인자도 위와 **똑같다** — `매개변수 = 넘긴 값`이라는 이름표 붙이기일 뿐이다. 그래서 위 (1)(2)(3)이 그대로 적용된다.

```python
def append_one(lst): lst.append(1)   # (3) 변형 → 호출자의 리스트도 바뀜 (같은 객체)
def rebind(lst):     lst = lst + [1] # (2) 재바인딩 → 함수 안에서만 새 객체, 호출자는 그대로

data = [1, 2]
append_one(data)   # data == [1, 2, 1]   ← 바뀜
rebind(data)       # data == [1, 2, 1]   ← rebind 호출은 아무 효과 없음
# 호출자를 안 건드리려면 함수 안에서 lst = lst[:] 로 복사 후 작업
```

파이썬에선 "pass by value냐 reference냐" 논쟁이 무의미하다 — 항상 같은 객체 참조가 넘어가고, 결과를 가르는 건 함수가 그 객체를 **변형**했는지 **재바인딩**했는지다.

```python
def f(a, b, *args, **kwargs):
    # *args   : 초과 위치인자 → 튜플
    # **kwargs: 초과 키워드인자 → 딕트
    ...
f(1, 2, *[3, 4], **{"k": 5})         # 펼쳐서 전달
# 기본 인자는 default 없는 매개변수 뒤에. 기본값이 mutable이면 함정(→ 함정·성능 글)
```

## 람다·고차함수·제너레이터

```python
key = lambda x: x[1]     # 한 줄 함수 (정렬 key 등 일회용)
list(map(f, it))         # [f(x) for x in it] 보다 메모리 적음(lazy)
list(filter(pred, it))   # pred 가 참인 것만
```

iterator = 값을 하나씩 내주는 스트림. generator = 필요할 때만 계산(lazy)하고 상태를 기억.

```python
gen = (f(x) for x in data)   # () 는 제너레이터(lazy), [] 는 리스트(즉시 다 만듦)
next(gen)                    # 다음 값 하나

def fib_upto(n):             # yield: 멈췄다 재개, 상태 유지
    a, b = 0, 1
    while a < n:
        yield a
        a, b = b, a + b
```

## 데코레이터: 함수를 감싸는 함수

함수를 받아 함수를 돌려준다. 로깅·캐싱·타이밍 같은 공통 작업을 본체 수정 없이 덧입힌다.

```python
import functools
def log_args(fn):
    @functools.wraps(fn)          # wrapper 가 원래 함수의 이름·docstring 을 유지
    def wrapper(*args, **kwargs):
        print(args, kwargs)
        return fn(*args, **kwargs)
    return wrapper

@log_args                         # compute = log_args(compute) 와 동일
def compute(...): ...
```

## OOP: 클래스와 인스턴스

class = 틀(blueprint), instance = 찍어낸 객체. 메서드 첫 인자 `self`는 인스턴스 자신.

```python
class Point:
    def __init__(self, x, y):     # 생성자 — 인스턴스 변수는 여기서 다 정의하는 게 국룰
        self.x = x; self.y = y
p = Point(1, 2)
p.move()  # == Point.move(p) — 메서드는 self 에 인스턴스가 묶인 함수
```

`self` 없이 클래스 본문에 둔 변수는 **클래스 변수**라 모든 인스턴스가 공유한다. 조회는 인스턴스 → 클래스 → 상위클래스 순으로 올라간다.

```python
class Dog:
    species = "canis"             # 클래스 변수 (공유)
    def __init__(self, name):
        self.name = name          # 인스턴스 변수 (개별)
```

## property / classmethod / staticmethod

```python
class Home:
    def __init__(self, area): self.area = area
    @property                     # 메서드를 속성처럼: home.price (괄호 없이)
    def price(self): return self.area * 1000
    @classmethod                  # __init__ 외의 또 다른 생성 경로
    def from_dict(cls, d): return cls(d["area"])
    @staticmethod                 # 인스턴스와 무관한 함수 (잘 안 씀)
    def unit(): return "만원"
```

## 매직 메서드와 __slots__

언더스코어 둘로 감싼 메서드를 정의하면 내장 연산과 연동된다(다형성).

```python
class Vec:
    def __init__(self, v): self.v = v
    def __len__(self): return len(self.v)         # len(obj)
    def __repr__(self): return f"Vec({self.v})"   # print/디버깅에 매우 유용
    def __contains__(self, x): return x in self.v # x in obj
    def __call__(self, i): return self.v[i]       # obj(i) — 인스턴스를 함수처럼
```

`__call__`은 클래스로 데코레이터를 만들 때 핵심. `__slots__`로 허용 속성을 못박으면 임의 속성 추가를 막고 메모리를 아낀다.

```python
class P:
    __slots__ = ["x", "y"]        # x, y 외 속성 추가 차단
```

## 상속과 예외

```python
class VIP(Person):
    def __init__(self, name, title):
        super().__init__(name)    # 부모 생성자 호출
        self.title = title
```

다중 상속은 되지만 가독성이 떨어진다. 추상 클래스는 `abc` 모듈로 "이 메서드는 반드시 구현"을 강제한다.

```python
try:
    risky()
except ValueError as e:           # 구체적으로 잡는다
    handle(e)
else:                             # 예외가 없을 때만
    ...
finally:                          # 있든 없든 항상
    cleanup()
```

맨 `except`로 다 잡으면 `KeyboardInterrupt`(Ctrl+C)·`SystemExit`까지 삼켜 위험 → 예외는 구체적으로 지정한다. 파이썬은 "허락보다 용서"(일단 실행하고 예외로 처리) 스타일.
