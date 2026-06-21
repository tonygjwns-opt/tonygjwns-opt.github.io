---
title: 파이썬 함정·성능 모음
date: 2026-06-21 09:00:00 +0900
categories: [컴공, 파이썬]
tags: [python, 함정, 성능, 코드리뷰]
toc: true
---

기초 문법은 검색하면 되지만, 직접 짜든 AI가 짜든 파이썬 코드를 검토할 때 "버그가 조용히 숨는 자리"는 정해져 있다. 그 자리들을 모았다.

## 1. 가변 객체와 공유 참조

같은 객체를 여러 번 가리키는 줄 모르고 한쪽을 고치면 전부 바뀐다. 파이썬 버그 1순위.

```python
grid = [[0]*n]*n                 # 함정: 같은 행 객체 n개 → grid[0][0]=1 이면 모든 행이 1
grid = [[0]*n for _ in range(n)] # 행마다 독립 생성

rows = [[]]*3                    # 함정: 빈 리스트 3개가 아니라 같은 리스트 3개
rows = [[] for _ in range(3)]    # rows[0].append(1) 이 전체에 안 퍼짐
```

기본값으로 준 리스트는 함수가 **정의될 때 한 번만** 만들어진다. 그래서 `bucket`을 안 넘기는 호출들은 매번 새 리스트가 아니라 **같은 리스트 하나를 공유**한다.

```python
def collect(item, bucket=[]):   # 함정: bucket=[] 는 정의 시 한 번만 생성
    bucket.append(item)
    return bucket

collect(1)   # 기대 [1],  실제 [1]
collect(2)   # 기대 [2],  실제 [1, 2]   ← 새 호출인데 이전 값이 남음
collect(3)   # 기대 [3],  실제 [1, 2, 3]
```

`collect(2)`는 "빈 바구니에서 2만 담아 `[2]`"를 기대하지만, 첫 호출이 쓰던 바구니가 함수에 눌러앉아 `[1, 2]`가 된다. 해결은 기본값을 `None`으로 두고 안에서 새로 만드는 것. (`dict`·`set` 기본값도 같다)

```python
def collect(item, bucket=None):
    if bucket is None: bucket = []
    bucket.append(item)
    return bucket
```

얕은 복사도 같은 함정이다.

```python
b = a[:]                    # 1차원은 슬라이싱이 곧 복사 — deepcopy보다 빠름
b = [row[:] for row in a]   # 2차원은 행마다 복사해야 독립
# a = b 는 복사가 아니라 같은 객체 참조 → 한쪽 수정이 양쪽에 반영
```

순회 중 원본 수정도 같은 부류다 — 리스트를 돌면서 그 리스트를 바꾸면 인덱스가 밀려 원소를 건너뛴다.

```python
for x in lst:
    if drop(x): lst.remove(x)            # 위험: 지우는 순간 뒤 원소가 당겨져 건너뜀
lst = [x for x in lst if not drop(x)]    # 권장: 새 리스트를 한 번에 (복사본 순회보다 빠름)
```

## 2. 값 vs 객체, 나눗셈, 참/거짓

```python
1 == 1.0    # True   (값 비교)
1 is 1.0    # False  (is 는 '같은 객체냐' = 메모리 정체성)
x is None   # None 비교만 is. 나머지 값 비교는 항상 ==
```

```python
-7 // 2     # -4   (// 는 음의 무한대 방향 floor)
int(-7/2)   # -3   (0 방향 절삭이 필요하면 int(a/b))
```

```python
if x:               # x 가 0·""·[]·None 이면 False → "원소 개수>0" 판정에 활용
None == None        # True
np.nan == np.nan    # False   (NaN 은 자기 자신과도 다름 → isna()/np.isnan())
```

## 3. 이터레이터는 한 번 쓰면 소진된다

```python
m = map(str, [1, 2, 3])
list(m)   # ['1','2','3']
list(m)   # []   ← 이미 다 흘려보냄
# map·filter·reversed·zip·제너레이터는 전부 일회성 스트림.
# 재사용·재배열하려면 먼저 list() 로 고정.
```

## 4. 제자리(in-place) 메서드의 반환값은 None

```python
y = lst.append(x)   # 함정: append 는 None 반환 → y 는 None
new = lst + [x]     # 새 리스트가 필요하면 이렇게
lst.sort()          # 원본을 정렬하고 None 반환
sorted(lst)         # 정렬된 새 리스트 반환 (원본 유지)
# reverse() ↔ reversed(lst)/lst[::-1] 도 '원본 변경 vs 새 객체' 같은 관계
```

## 5. 컬렉션 함정

```python
{}              # 빈 dict (빈 set 아님!)
set()           # 빈 set
list(set(a))    # 중복 제거. 단 a 의 원소가 list 면 unhashable → 에러
# dict 키·set 원소는 hashable(불변)만: 숫자·문자열·튜플 OK / 리스트·딕트 NO.
# 튜플도 안에 리스트를 품으면 unhashable.
```

## 6. 자료구조와 성능

조회·삽입 위치 하나로 O(1) 과 O(n) 이 갈린다.

```python
x in lst        # 리스트는 O(n) 선형탐색
x in s          # set·dict 는 O(1) (해시 기반) → 잦은 조회는 set/dict 로
lst.index(x)    # 역시 O(n)
lst.pop(0)      # 맨 앞 제거는 O(n) (뒤 원소를 한 칸씩 당김)
from collections import deque   # 양끝 O(1): popleft()/appendleft()
len(x)          # O(1)
```

대량 입출력일 때:

```python
import sys
input = sys.stdin.readline   # 내장 input() 보다 빠름 (.strip() 으로 개행 제거)
sys.stdout.write(...)        # print 가 매우 많을 때
```

## 7. 자주 쓰는 관용구

```python
print(*lst)              # 리스트를 공백으로 펼쳐 출력
[*s]                     # 문자열 → 문자 리스트
s[::-1]                  # 문자열·리스트 뒤집기
list(map(int, str(n)))   # 정수의 각 자릿수를 리스트로
x, y = y, x              # 임시변수 없이 스왑
INF = float('inf')       # 큰 초기값 (1e9 보다 안전한 무한대)
```

---

보충·교정 기록
- mutable default argument(기본값=[]) 함정을 `collect` 예제 + 설명으로 추가(호출 간 기본값 공유). 원본 노트엔 없었으나 AI 코드에 흔해 보강.
- 순회 중 원본 수정 함정 추가(복사본 순회 vs 컴프리헨션 재생성).
- 큰 초기값 1e9 → float('inf') 권장으로 보강.
- is None / 캐싱된 작은 int 한 줄 보강.
- PS 전용(PyPy3 제출·카운팅 정렬)은 범용 글이라 제외.
- ==/is·가변성·hashable 개념 배경은 다음 글(데이터모델)에서 다룸.
