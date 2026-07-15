---
title: 자료구조 — 선형 · 해시 · 트리 · 힙
date: 2026-06-22 11:00:00 +0900
categories: [컴공, 자료구조]
tags: [자료구조, 스택, 큐, 해시, 힙]
toc: true
---

자료구조 선택은 "어떤 연산이 자주 필요한가"로 결정된다.

## 개요

- 4대 기능: 입력 / 처리 / 유지(메모리·관계) / 검색.
- 선택 기준: ① 목적에 맞는 연산이 있는가 ② 메모리 방식(static 고정 vs dynamic 가변 — C에선 중요, 파이썬·자바는 자동) ③ 어느 쪽이 더 빠른가.

## 선형: 스택 · 큐 · 덱

- **스택(LIFO)**: 가장 최근 것부터 처리. 짝 맞추기·'직전으로 돌아가기'에 적합.
  - 괄호 검사: 열면 push, 닫으면 pop해 짝 확인, 끝에 비어야 성공.
  - 단조 스택: 아직 답 못 찾은 인덱스만 단조로 쌓아두면 '다음 큰 수' 류를 O(n)(안쪽 루프면 O(n²)).
- **큐·덱(FIFO)**: 앞에서 빼고 뒤에서 넣음. `list.pop(0)`은 O(n) → `deque`(popleft O(1)).
  - 순환·회전은 인덱스 modulo. '뒤집기' 같은 비싼 연산은 방향 flag로 미루고 출력 때만 반영. 크기 제한 있으면 overflow/underflow 주의.

```python
from collections import deque
dq = deque()
dq.append(x);     dq.pop()       # 뒤 (스택처럼)
dq.appendleft(x); dq.popleft()   # 앞 (큐처럼) — 양끝 모두 O(1)
```

## 선형: 링크드리스트

본질: Node(데이터 + 다음 노드 링크)를 엮은 선형 구조. 배열과 달리 메모리상 순서와 무관.

- **singly**: 한 방향. 중간 노드 제거 시 이전 노드의 next를 다음 노드로 이어준다.
- **doubly**: prev까지 보유. head/tail 제거는 포인터 1개, 중간 제거는 양옆 2개 갱신. tail로 역방향 순회.
- orphan node: 그 노드를 가리키는 참조가 없어진 노드(= 사실상 제거됨).
- 팁: singly에서 '끝에서 n번째'는 길이를 모르니 포인터 2개를 간격 두고 움직여 해결(추가 메모리 없이).
- (실무 파이썬은 list/deque로 충분 — 직접 구현할 일은 드물지만 개념은 알아둘 것.)

## 해시 (집합 · 맵)

본질: "있는지 / 몇 개인지"는 hash로 O(1). list의 in·index(O(n))를 피한다.

- 빈도수는 `dict{값:개수}`(또는 `Counter`). 키가 작은 정수뿐이면 `list[index]`를 카운터로(index가 hash 역할).
- 역방향 조회가 필요하면 양방향 매핑을 둘 다 만든다(이름→번호, 번호→이름).
- 집합 연산(교/합/차)은 내장. 대칭차 크기 = n + m − 2·|교집합|.

```python
from collections import Counter
Counter(a)          # 빈도 dict
set(a) & set(b)     # 교집합 (| 합, - 차, ^ 대칭차)
```

해시맵 내부: hash function이 key를 배열 인덱스로 압축(compression) → 저장·검색 O(1). 같은 key는 항상 같은 인덱스가 나오고, 일방향(역산 불가)이며 충돌쌍을 의도적으로 찾기 어렵다. 충돌(다른 key가 같은 인덱스) 해결:

- **separate chaining**: 그 인덱스에 linked list로 묶어 저장(key도 함께 저장해 구분). 잘 흩어지면 효율적.
- **open addressing(probing)**: 차 있으면 규칙대로 다음 빈 인덱스를 찾는다. 조회도 같은 규칙으로 추적.
- **clustering**: 충돌이 또 충돌을 부르는 뭉침 현상(성능 저하).

## 트리 · 힙

- **Tree**: node가 0개 이상의 child를 가리키는 위계 구조(폴더 구조). child 없으면 leaf, 모든 node는 parent 하나뿐. 순회는 DFS/BFS.
- **BST(이진탐색트리)**: child 2개 이하 + 왼쪽 < parent < 오른쪽 → 탐색에 유리(균형 시 O(log n)). 편향되면 O(n)이라 실무는 자가균형 트리(red-black 등)를 쓴다.
- **Heap**: parent-child 대소 규칙이 일정한 트리. min-heap은 root가 최소, child ≥ parent. 최대/최소 추적·우선순위 큐에.
  - 삽입: 왼→오로 채우고, 규칙 깨지면 부모와 swap해 올림(heapify up).
  - 삭제(root 제거): root와 마지막 원소 swap → 제거 → 규칙 맞게 내림(heapify down).
  - 보통 배열로 구현. root를 index 1로 두면 parent=i//2, left=i·2, right=i·2+1.

```python
import heapq
h = []
heapq.heappush(h, x); heapq.heappop(h)   # 최소 힙, O(log n)
# 최대 힙은 값을 음수로 넣거나 (-priority, item) 튜플로
```
