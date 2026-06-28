---
title: pandas 기초 — 생성·조회·정제
date: 2026-06-21 16:00:00 +0900
categories: [컴공, 파이썬]
tags: [python, pandas, dataframe]
toc: true
---

pandas는 행·열 표(DataFrame)와 1차원 열(Series)을 다룬다. 전형적 작업은 한 방향으로 흐른다: **읽기(read_csv) → 훑기(head·info·describe) → 고르기(loc·iloc·조건) → 정제(결측·타입·변환)**. 이 글은 그 앞부분(생성~정제)이고, 인덱스·집계·병합은 심화 글에서 다룬다.

## 생성

```python
import pandas as pd
import numpy as np

s = pd.Series([1, 2, 3, np.nan])      # 1차원 (np.nan → NaN)
df = pd.DataFrame({                    # dict → 표
    'name': ['Lee', 'Park'],
    'age':  [21, 35],
})
pd.DataFrame(np.random.rand(6, 4), columns=['A', 'B', 'C', 'D'])   # 배열 + 컬럼 지정
# columns 에 없던 이름을 넣으면 그 컬럼은 전부 NaN
```

## 읽기·쓰기

```python
df = pd.read_csv('my.csv')                 # 첫 행=컬럼명, 콤마 구분
df.to_csv('out.csv', index=False)          # index=False: 인덱스 열 제외
df.to_excel('out.xlsx', sheet_name='S', index=False)
df.to_csv('out.csv', index=False, encoding='utf-8-sig')   # 한글 깨질 때

import glob
glob.glob('file*.csv')                     # 패턴에 맞는 파일 목록

import os; os.getcwd()                      # 현재 작업 폴더
path = r'C:\Users\me\data.csv'             # 윈도우 역슬래시는 raw 문자열 r'...'
```

## 훑어보기

```python
df.head(3); df.tail(3)     # 앞/뒤 몇 줄
df.info()                  # 행 수·컬럼·타입·결측
df.describe()              # 수치 컬럼 요약 통계
df.shape                   # (행, 열)
df.index; df.columns; df.dtypes
```

## 고르기 (선택·필터)

컬럼 vs 행, 위치 vs 라벨을 구분하는 게 핵심.

```python
df['age']            # 컬럼 하나 → Series
df[['name', 'age']]  # 여러 컬럼 → DataFrame

df.iloc[0]           # 위치(정수) 기반 — 0번째 행
df.iloc[1:3]         # 1,2 행 (3 미포함)
df.loc[label]        # 라벨(인덱스 값) 기반
df.loc[:, ['A', 'B']]            # 모든 행, 특정 컬럼
df.loc[df.age > 30, 'name']      # 조건 + 컬럼 동시

df[df.age == 39]                 # boolean 인덱싱
df[(df.A > 0) & (df.B < 5)]      # 여러 조건은 () + & | (and/or 아님)
df[df.name.isin(['Lee', 'Kim'])] # 목록에 포함되는 행
```

★ 원본 수정은 한 번에: `df.loc[row, col] = x`. `df.loc[row][col] = x`는 복사본을 건드려 원본이 안 바뀐다(SettingWithCopy 경고).

## 정제: 결측치(NaN)

```python
# None==None 은 True 지만 np.nan==np.nan 은 False → 비교 대신 isna()
df.isna().sum()              # 컬럼별 결측 개수
df.dropna()                  # 결측 있는 행 제거. subset=['b'] 로 특정 컬럼만
df.fillna(0)                 # 결측을 0 으로
df['C'] = np.where(df['C'].notna(), df['C'], df['B'])   # C 결측이면 B 로 대체
df['b'] = df['b'].where(df['b'] > 0, np.nan)            # 조건 거짓인 값을 NaN 으로
```

주의: `isna()`는 NaN/None만 센다. "불가능한 값"(양수여야 하는데 0 등)은 따로 걸러야 한다.

## 정제: 변환 (apply·타입·문자열)

```python
df['flag'] = df.b.apply(lambda x: x == 'a')           # 컬럼 단위 변환(없던 컬럼 생성도)
df.apply(lambda row: row.price * 1.075                 # 행 단위는 axis=1
         if row.taxed == 'Yes' else row.price, axis=1)

df['code'] = df['code'].astype(str)                    # 타입 변환
df['amt']  = df['amt'].astype(str).str.replace(',', '').astype(int)   # 콤마 제거 후 int
df['code'] = df['code'].map('{:06d}'.format)           # 6자리 zero-fill(종목코드 등)
df['b']    = df['b'].replace(r'[\- ]', '', regex=True) # 정규식 치환
```

## 인덱스 리셋

```python
df.reset_index(drop=True)    # 띄엄띄엄해진 인덱스를 0,1,2.. 로. drop=True 면 옛 인덱스 버림
# inplace=True 면 새 df 안 만들고 원본 변경
```
