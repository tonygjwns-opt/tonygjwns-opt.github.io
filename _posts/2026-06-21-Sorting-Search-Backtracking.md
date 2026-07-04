---
title: 정렬 · 탐색 · 완전탐색 · 백트래킹
date: 2026-06-21 21:00:00 +0900
categories: [컴공, 알고리즘]
tags: [알고리즘, 정렬, 이진탐색, 백트래킹]
toc: true
---

탐색은 답 후보를 뒤지는 일이다. 정렬은 그 전처리(이진탐색을 가능케 함), 완전탐색은 다 보기, 백트래킹은 다 보되 가지치기다.

## 정렬 — 기준(key) 설계

본질: 비교 기준(key)을 설계하는 문제. 정렬 자체는 내장으로 끝.

- 다중 기준은 튜플 key. 파이썬 `sort`는 **stable** → 순차 정렬로 우선순위를 쌓거나 입력순 보존이 공짜.

```python
li.sort(key=lambda x: (x[0], -x[1]))   # 1차 오름차순, 2차 내림차순
li.sort(); li.sort(key=len)            # stable: 앞 정렬 순서가 동점에서 보존됨
```

- 값 범위가 작으면 카운팅 정렬(값=인덱스인 배열에 개수)로 O(n+k).
- 값→순위 매핑(좌표 압축)은 `list.index()`가 O(n) → `dict{값:순위}`로 O(1).

## 정렬 알고리즘 (원리·복잡도)

실무는 내장 sort. 여기선 대표들의 원리와 복잡도만.

- **bubble**: 인접한 둘을 비교해 큰 걸 뒤로 swap. O(n²). 이미 정렬돼 있으면 swap이 적다.
- **insertion**: 왼쪽의 정렬된 부분에 다음 원소를 제자리에 끼워 넣는다. O(n²)이지만 거의 정렬된 데이터·작은 배열엔 매우 빠르다(그래서 Tim sort가 작은 조각에 활용).
- **shell**: insertion sort의 개선. 큰 간격(gap)으로 떨어진 원소끼리 먼저 insertion하고, gap을 줄여가며 반복해 마지막 gap=1은 보통 insertion이 된다. 큰 gap이 멀리 있는 원소를 미리 대략 제자리로 옮겨줘, 마지막 단계가 거의 정렬된 상태라 빨라진다. 복잡도는 gap 수열에 따라 O(n²)~O(n^1.25) 수준.
- **merge**: 1짜리가 될 때까지 반으로 쪼갠 뒤, 정렬된 두 조각을 작은 것부터 붙여 merge. 한쪽이 다 들어가면 남은 쪽은 통째로. Θ(n log n)(최선·최악·평균 동일), 공간 O(n).
- **quick**: pivot 기준 작은 것/큰 것으로 분할 후 재귀. pivot을 잘 골라야 균등 분할(랜덤, 또는 세 값의 중앙값). 평균 O(n log n), 최악 O(n²).
- **Tim sort**: 파이썬 `.sort()`/`sorted()`의 실제 알고리즘. merge sort를 실제 데이터에 맞게 튜닝한 것으로, 핵심 발상은 "실제 데이터엔 이미 정렬된 구간이 흔하니 그걸 공짜로 활용하자"이다.
  - **run 찾기**: 배열을 훑으며 이미 정렬된 연속 구간(run)을 찾는다. 내림차순 run이면 뒤집어 오름차순으로 만든다(stable 유지). 그래서 이미 정렬된 입력은 run 하나로 끝 → O(n).
  - **minrun으로 키우기**: run이 너무 짧으면 병합 횟수만 늘어 비효율. 그래서 최소 길이 minrun(보통 32~64)을 정하고, 짧은 run은 뒤 원소들을 binary insertion sort로 끌어와 minrun 길이까지 채운다.
  - **균형 잡힌 merge**: 만들어진 run들을 스택에 쌓되, 인접한 run들의 크기가 비슷하게 유지되도록 불변식을 지키며 즉시 병합한다. 비슷한 크기끼리 합쳐야 병합 트리가 균형 잡혀 O(n log n)이 보장된다.
  - **galloping(질주 모드)**: 두 run을 병합할 때 한쪽이 연속으로 계속 이기면(더 작으면), 하나씩 비교하는 대신 이진탐색으로 "몇 개가 연속으로 이기는지"를 점프해 찾아 한 번에 복사한다. 한쪽에 작은 값이 몰려 있을 때 비교 횟수를 크게 줄인다.
  - 복잡도: 이미 정렬돼 있으면 O(n), 평균·최악 O(n log n), 공간 O(n).

## 이진탐색 (binary search)

본질: linear는 처음부터 순서대로 O(n), binary는 **정렬된** 배열을 반씩 좁혀 O(log n).

```python
def lower_bound(a, target):      # a는 정렬돼 있어야 함
    left, right = 0, len(a)
    while left < right:
        mid = (left + right) // 2
        if a[mid] < target: left = mid + 1
        else:               right = mid
    return left                  # target 이 들어갈 위치. 존재 확인은 left<len(a) and a[left]==target
# 실무는 표준 bisect 모듈: bisect_left / bisect_right / insort
```

## 완전탐색 (brute force)

본질: 경우를 다 보되, **탐색 범위·시작점을 수학적으로 좁히는 게** 핵심.

- 조합은 인덱스 순서로 중복 없이(i<j<k), 목표 도달 시 즉시 break.
- 격자 탐색은 가능한 모든 시작 위치에서 창을 훑는다. 두 경우(흑/백 등)는 동시에 비교.
- 경계 입력(0, 음수, 1개, 빈 문자열)을 습관적으로 점검.
- 정형 규칙이 없으면 조건 만족까지 순차 증가시키며 세도 된다.

## 백트래킹 (backtracking)

본질: 가능한 트리를 DFS하되, 안 될 가지는 즉시 끊는다(pruning).

```python
def backtrack(path, start):
    if is_solution(path):
        record(path); return
    for i in range(start, n):
        path.append(i)          # 선택
        backtrack(path, i + 1)  # 재귀 (순열이면 visited로 제어)
        path.pop()              # 되돌리기
```

- 순열/조합 생성: 조합은 다음 재귀에 `start=i+1`, 순열은 `visited`로 사용한 수 제외.
- 상태는 최소로: N-Queen은 `board[row]=열` 1차원. 충돌은 같은 열 `board[j]==col`, 대각선 `abs(board[j]-col)==abs(j-row)`.
- 해를 찾으면 즉시 멈추고 위로 전파(flag+return). 확정된 부분(진행 위치)은 인자로 건너뛰어 가지친다.
