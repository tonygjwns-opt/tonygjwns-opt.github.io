---
title: matplotlib · seaborn 시각화
date: 2026-06-21 18:00:00 +0900
categories: [컴공, 파이썬]
tags: [python, matplotlib, seaborn, 시각화]
toc: true
---

## 전체 흐름 (작동 모델)

matplotlib은 **Figure(도화지)** 위에 **Axes(축이 있는 플롯 영역)** 를 얹고 거기에 그린다. 두 스타일이 있다.

- pyplot: `plt.plot()·plt.title()` — "현재 그림"에 누적. 빠른 스크립트용.
- 객체지향: `fig, ax = plt.subplots()` 후 `ax.plot()` — 여러 subplot·세밀 제어에 좋고 권장. (아래는 이 스타일로 통일)

흐름: figure 만들기 → 그리기 → 서식(축·범례·제목) → `show()`/`savefig()`. seaborn은 그 위의 고수준 wrapper로, DataFrame을 바로 받아 한 줄에 통계 그래프를 그린다.

## 기본 그리기·서식

```python
import matplotlib.pyplot as plt
fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(x, y, color='g', marker='o', linestyle='-', label='price')   # 축약 'go-'
ax.set_title('제목'); ax.set_xlabel('x'); ax.set_ylabel('y')
ax.legend(loc='best')
plt.savefig('out.png'); plt.show()
```

한글 폰트(안 하면 깨짐):

```python
from matplotlib import rc, font_manager
f = font_manager.FontProperties(fname='C:/Windows/Fonts/malgun.ttf').get_name()
rc('font', family=f)
plt.rcParams['axes.unicode_minus'] = False   # 음수 부호(-) 깨짐 방지
```

축 눈금·영역 채우기:

```python
ax.set_xticks([0, 250, 500]); ax.set_xticklabels(['0%', '25%', '50%'], rotation=30)
import matplotlib.ticker as mtick
ax.yaxis.set_major_formatter(mtick.FormatStrFormatter('%.1f%%'))   # % 서식
ax.fill_between(x, lo, hi, alpha=0.2)        # 두 선 사이 채움(신뢰구간 밴드)
```

여러 subplot·이중 y축:

```python
fig, axes = plt.subplots(1, 2, figsize=(10, 4))   # axes[0], axes[1] 로 각각 제어
axes[0].plot(...); axes[1].bar(...)
plt.tight_layout()                                 # 라벨 겹침 방지

ax2 = ax.twinx()     # 오른쪽 y축 — 범위 다른 두 데이터를 한 그림에 겹쳐 그릴 때
```

## 차트 종류

```python
ax.scatter(x, y)                            # 산점도
ax.hist(data, bins=20, edgecolor='black')   # 히스토그램(분포)
ax.boxplot([d1, d2], labels=['A', 'B'])     # 박스플롯(사분위·이상치)
ax.bar(x, h); ax.barh(x, h)                 # 막대 / 가로막대
ax.bar(x, y2, bottom=y1)                    # 누적 막대
ax.pie(data, autopct='%d%%'); ax.axis('equal')   # 파이(equal로 찌그러짐 방지)
ax.violinplot([d1, d2])                     # 분포(밀도)까지 보는 박스플롯
ax.imshow(mat2d, cmap='coolwarm')           # 2D 행렬을 색으로(히트맵·이미지)
```

## seaborn (고수준 통계 그래프)

DataFrame을 바로 받아 한 줄로, 통계 요약(평균·신뢰구간)을 자동으로 그린다.

```python
import seaborn as sns
sns.barplot(data=df, x='gender', y='score')                   # 기본: 평균 + 95% 신뢰구간
sns.barplot(data=df, x='gender', y='score', hue='age_group')  # hue 로 추가 분류
sns.boxplot(data=df, x='cat', y='val')
sns.violinplot(data=df, x='cat', y='val')   # 흰점=중앙값, 굵은 box=Q1~Q3, 양옆=KDE 곡선
sns.kdeplot(df['x'], fill=True)             # 커널 밀도(연속 분포 추정)
sns.heatmap(df.corr(), annot=True)          # 상관행렬 히트맵(퀀트 단골)
```

스타일:

```python
sns.set_style('whitegrid')    # darkgrid/whitegrid/dark/white/ticks
sns.set_context('talk')       # paper/notebook/talk/poster (전체 크기 스케일)
sns.set_palette('colorblind') # 색맹 안전 팔레트 권장(보색 빨/초 회피)
sns.despine()                 # 위·오른쪽 테두리 제거(plot 호출 후)
# 집계·오차 변경: estimator=np.median, ci='sd' 등
```
