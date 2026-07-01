---
title: 재귀 · 분할정복 · 복잡도
date: 2026-06-21 20:00:00 +0900
categories: [컴공, 알고리즘]
tags: [알고리즘, 재귀, 분할정복, 복잡도]
toc: true
---

재귀는 "큰 문제를 같은 꼴의 작은 문제로 환원"하는 사고틀, 분할정복은 그 대표 응용, Big O는 그 비용을 재는 언어다.

## 재귀 (recursion)

본질: 큰 문제를 같은 꼴의 작은 문제로 환원한다. **종료조건(base)과 환원식**이 전부.

- base는 가장 작은 경계까지(n==1). 한 단계 위에서 끊으면 최소 입력에서 깨진다.
- 동작 원리: call stack(LIFO — 마지막 호출이 먼저 반환). base 없으면 무한 → stack overflow.
- 같은 부분문제를 중복 호출하면 폭발한다: 단순 재귀 피보나치는 O(φ^n) → 메모이제이션이나 DP로 O(n).
- 하노이류: n-1 옮기고 → 큰 1개 옮기고 → n-1 옮기기. 이동 수 2^n−1 (T(n)=2T(n-1)+1).

```python
from functools import lru_cache

@lru_cache(None)          # 중복 부분문제 캐싱 → 지수 재귀를 선형으로
def fib(n):
    if n < 2: return n    # base
    return fib(n - 1) + fib(n - 2)
```

## 분할정복 (divide & conquer)

본질: 같은 구조의 부분으로 쪼개 재귀하고 **합친다**. 균등 분할이면 깊이가 log라 빠르다.

- 균일하면 멈추고, 섞였으면 분할(쿼드=4분할, 종이=9분할). 균일성은 "전부 같은가"로 단순화.
- 분할 좌표는 이중 for로 오프셋을 곱해 일반화.
- 큰 거듭제곱을 O(log)로: 지수를 반으로 쪼갠다. a^b = (a^(b/2))², 홀수면 ×a, 매 단계 모듈러.

```python
def power(a, b, mod):            # 빠른 거듭제곱 O(log b)
    if b == 0: return 1
    half = power(a, b // 2, mod)
    half = half * half % mod
    return half * a % mod if b % 2 else half
```

(정렬의 merge·quick, 최단거리 등이 다 분할정복 계열 — 정렬 글 참고.)

## 복잡도 (Big O)

본질: 입력 크기 N에 대한 **연산 수 증가율**. 최고차항만 남기고 계수·하위항은 버린다 (3N²+4N → O(N²)).

- big O(최악) / big Ω(최선) / big Θ(상·하한 동일).
- 같은 연산도 자료구조에 따라 다르다: 값 존재 확인 = list O(n) vs set/dict(hash) O(1).
- 공간 복잡도에도 적용. 입력 자체 크기는 제외하고 추가로 쓴 공간만 센다 (제자리 수정 O(1), 새 배열 O(n)).
- 재귀 비용은 점화식으로 잰다: 분할정복 T(n)=2T(n/2)+O(n) → O(n log n) (마스터 정리).
