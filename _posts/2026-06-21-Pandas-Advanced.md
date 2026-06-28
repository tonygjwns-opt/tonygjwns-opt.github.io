---
title: pandas 심화 — 인덱스·집계·병합·시계열
date: 2026-06-21 17:00:00 +0900
categories: [컴공, 파이썬]
tags: [python, pandas, groupby, 시계열]
toc: true
---

기초(생성·조회·정제) 다음 단계 — 표를 **재구조화하고, 묶어서 집계하고, 합치고, 시계열로 다루는** 작업. 퀀트 데이터 작업의 대부분이 여기 있다.

## 인덱스 다루기

```python
df.set_index('date')            # 한 컬럼을 인덱스로 (고유값이어야 깔끔)
df.sort_index(ascending=False)  # 인덱스 기준 정렬
df.sort_values(by='code')       # 컬럼 값 기준 정렬
df.reset_index(drop=True)       # 인덱스를 0,1,2.. 로 되돌림(drop=True면 옛 인덱스 버림)
df.columns = df.columns.swaplevel(0, 1)   # 멀티인덱스 레벨 교환
```

## 컬럼·행 조작

```python
df.rename(columns={'name': 'Name', 'age': 'Age'})   # 일부만 변경(없는 key 는 무시)
df.columns = ['a', 'b', 'c']                         # 전체 교체(길이 일치해야)
df.columns = df.columns.str.lower()                  # 소문자화

del df['a']                  # 원본에서 컬럼 제거
df.drop(columns=['B', 'C'])  # 제거된 새 df 반환(원본 유지). axis=1 도 동일
df.drop([0, 1])              # 행 제거(인덱스로)
```

## 중복·고유값

```python
df.duplicated()                       # 행별 중복 여부(True=중복)
df.drop_duplicates(subset=['item'])   # 특정 컬럼 기준 중복 제거
df['col'].unique()                    # 고유값 배열
df['col'].nunique()                   # 고유값 개수
df['col'].value_counts()              # 값별 빈도
```

## groupby (집계)

"쪼개기(split) → 적용(apply) → 합치기(combine)". groupby 뒤엔 반드시 집계를 붙인다.

```python
df.groupby('student')['grade'].mean()
df.groupby(['c1', 'c2'])['c3'].sum().reset_index()    # 여러 키, 인덱스 정리
df.groupby('cat')['id'].count().reset_index().rename(columns={'id': 'n'})
df.groupby('cat')['x'].agg(['mean', 'std', 'count'])  # 여러 집계 한 번에
```

## 형태 변환 (long ↔ wide)

같은 데이터도 두 가지 모양으로 둘 수 있다.

- **long(긴 형태)**: 한 행 = 관측 하나. 컬럼 수는 적고 행이 길다. DB·seaborn 그래프에 편함.
- **wide(넓은 형태)**: 한 항목이 한 열. 사람이 보기 좋은 피벗표. 시계열 정렬·상관계산에 편함.

`pivot`은 long → wide, `melt`는 wide → long. 같은 데이터를 오가는 변환이다.

```python
# long 원본: 한 행에 (날짜, 종목, 종가) 하나씩
#   date        ticker  close
#   2020-01-01  AAPL    100
#   2020-01-01  MSFT    200
#   2020-01-02  AAPL    101
#   2020-01-02  MSFT    201

df.pivot(index='date', columns='ticker', values='close')
# → wide: 종목을 '열'로 펼치고, close 를 칸에 채움
#   ticker      AAPL  MSFT
#   date
#   2020-01-01  100   200
#   2020-01-02  101   201
#   ※ index·columns 조합이 중복되면 에러 → 평균 등으로 합치는 pivot_table 사용
```

```python
# melt 는 반대 방향: 흩어진 열들을 (이름, 값) 두 컬럼으로 접음
df.melt(id_vars=['date'], var_name='ticker', value_name='close')
# wide 의 AAPL·MSFT 열이 다시 ticker/close 두 컬럼의 여러 행으로 돌아감(long 복원)
```

`crosstab`은 두 범주의 **조합별 개수**를 세는 표(빈도 교차표):

```python
pd.crosstab(df['gender'], df['passed'])
# → 각 (gender, passed) 조합이 몇 번 나오는지
#   passed   False  True
#   gender
#   F        2      5
#   M        3      4
```

## 병합

```python
pd.concat([df1, df2])           # 위아래로 이어붙이기(컬럼 같을 때)
pd.concat([df1, df2], axis=1)   # 좌우로 붙이기
pd.merge(a, b, on='id')         # 공통 컬럼 기준 결합
pd.merge(a, b, left_on='cust_id', right_on='id', suffixes=['_o', '_c'])  # 키 이름 다를 때
# how: inner(기본, 양쪽 매칭) / outer(전부) / left(왼쪽 전부 + 매칭되는 오른쪽)
```

## 시계열

퀀트에서 가장 자주 쓰는 부분.

```python
df['t'] = pd.to_datetime(df['t'])   # 문자열 → datetime
df = df.set_index('t')               # 시계열 인덱스화 → 날짜 슬라이싱·리샘플 가능
df.loc['2020-01':'2020-03']          # 기간 슬라이싱

df.resample('M').last()              # 월말값(다운샘플 + 집계)
df.resample('W').ohlc()              # 주간 시·고·저·종가
df.resample('3D').mean()             # 3일 간격 평균
# freq: D 일 / W 주 / M 월말 / MS 월초 / Q 분기 / B 영업일

pd.date_range('2020-01-01', '2020-12-31', freq='B')   # 영업일 날짜 배열
```

## 빠른 plot

```python
df.groupby('hour')['views'].count().plot(figsize=(12, 7))
df.plot(kind='bar', stacked=True)
df.plot(kind='scatter', x='x', y='y')
# 세부 서식·차트 종류는 matplotlib·seaborn 글 참고
```
