---
title: 퀀트 엔진 짓기 #0 — 프로젝트 뼈대와 Supabase 연결
date: 2026-07-19 12:00:00 +0900
categories: [퀀트, 실무]
tags: [python, sqlalchemy, supabase, postgresql, pytest, setup]
toc: true
---

포트폴리오 최적화·백테스팅을 하는 퀀트 엔진을 **빈 폴더에서부터** 짓는 시리즈다. 최종 목표는 매일 데이터 수집 → 알파 채굴 → 백테스트·검증 → (제일 나중) 자동매매까지, 그리고 그 전 과정을 웹으로 관리하는 것. 이 글은 그 0번째 — 아무것도 없는 폴더에서 **프로젝트 뼈대**와 **DB에 안전하게 연결되는 첫 기능**까지다.

## 전체 그림

엔진의 파이프라인 순서는 이렇다:

> **data → returns → metrics → optimizer → backtest → 검증**

전체를 한번에 다 구현하려고 하는 대신, 먼저 **균등가중(EW, equal-weight)** 으로 이 파이프라인을 끝까지 한 바퀴 관통해 돌아가는 모델을 구현한 뒤, 그 위에 살을 붙이는 방식으로 진행한다. 이때 각 단계는 작게 쪼갠 함수와 그 함수를 검증하는 테스트로 쌓는다.

저장소는 둘로 나눈다.

- **공개 (`quantproject_open`)** — 엔진과 방법론: 데이터 골격, returns, metrics, optimizer, backtest, 검증, 웹 틀, 예제, 테스트, 문서.
- **비공개** — 실제 알파 시그널 로직, 실거래 API, 주문·계좌·운영, 그리고 자격증명(`.env`).

이 글의 코드는 전부 공개 엔진 쪽이다.

## 1. 셋업

셋업을 하면서 내린 결정들을 기록해 둔다.

### src-layout

패키지를 루트가 아니라 `src/` 아래에 둔다.

```
quantproject_open/
├─ pyproject.toml         # 패키지 신분증 · 빌드 · src 위치
├─ requirements.txt       # 런타임 의존성
├─ requirements-dev.txt   # 개발 의존성(pytest)
├─ .gitignore  LICENSE  README.md
├─ src/portoptim/
│  └─ __init__.py
└─ tests/
```

src-layout는 테스트가 **"설치된 패키지"** 를 import하도록 강제한다. 패키지가 루트에 노출돼 있으면 "로컬에선 되는데 배포하면 import 안 되는" 사고가 나기 쉬운데, 이를 구조적으로 막는다. (참고로 GitHub 레포 이름 `quantproject_open`과 파이썬 패키지 이름 `portoptim`은 달라도 된다 — `scikit-learn`→`import sklearn`처럼.)

### pip + venv + requirements

의존성 관리는 poetry·uv 대신 가장 전통적인 `pip + venv + requirements.txt`로 갔다. 학습·기록용 프로젝트에는 **설명할 것이 적고 투명한 쪽**이 낫다. 대신 자동 락이 없으니 재현성은 requirements로 직접 관리한다.

그런데 src-layout를 쓰면 테스트가 패키지를 import하려면 **editable 설치**(`pip install -e .`)가 필요하고, 그러려면 빌드 메타가 담긴 `pyproject.toml`이 있어야 한다. 그래서 pip 방식이어도 `pyproject.toml`은 남는다. 역할을 이렇게 나눴다:

- `pyproject.toml` — 패키지 이름·빌드 백엔드(setuptools)·src 위치
- `requirements.txt` — 런타임 의존성
- `requirements-dev.txt` — 개발 의존성. 첫 줄 `-r requirements.txt`로 런타임까지 함께 설치

설치와 검증은 한 번에:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements-dev.txt -e .
pytest
```

### 빌드 백엔드 (setuptools)

`pip install -e .`을 실행하면 뒤에서 이렇게 동작한다. pip이 `pyproject.toml`을 읽어 빌드 백엔드를 확인하고(여기선 setuptools), 그 백엔드에게 패키지 조립을 맡긴다. setuptools는 `src/portoptim/`을 `portoptim`이라는 설치 가능한 패키지로 만든다. editable(`-e`) 모드라 소스를 고치면 재설치 없이 바로 반영되고, 그 결과 어디서든 `import portoptim`이 된다.

(setuptools: 파이썬에서 가장 표준적인 빌드 백엔드. `pyproject.toml`이 그 설정을 담는다.)

### 철학: 작은 순수함수 + pytest

각 기능을 **단일 책임 · 결정적 입출력** 으로 쪼개 테스트로 바로 검증한다.

## 2. DB 고르기

먼저 이 프로젝트의 실제 요구사항부터 못박았다.

- **주 워크로드**: 가격 패널을 날짜 범위로 **대량 읽어** returns·backtest 계산 → 분석(OLAP) 성격
- **PC를 꺼도 매일 수집** 이 돌아야 함
- **대상은 전종목이 아님**: 각국·섹터 ETF + 대형주 + 섹터 주도주(한·미·기타) → 규모가 작다
- 읽기 위주 대시보드, 무료

후보를 이 기준으로 비교했다.

| | 서버 | 자격증명 | 분석 대량읽기 | 원격 | 웹/API 내장 | 재현성 |
|---|---|---|---|---|---|---|
| **DuckDB + parquet** | 없음(임베디드) | 불필요 | 매우 빠름 | 로컬 | ✗ | 파일 하나 |
| 로컬 Docker Postgres | Docker | 로컬 | 보통 | ✗ | ✗ | 중 |
| **Supabase** (호스팅 PG) | 호스팅 | 필요 | 네트워크로 느림 | ✓ | Auth·API | 중 |
| Neon (서버리스 PG) | 호스팅 | 필요 | 네트워크로 느림 | ✓ | ✗(브랜칭 ✓) | 중 |

판단의 핵심은 **"PC를 꺼도 돌아간다"** 를 두 가지로 쪼갠 데 있다.

1. **항상 켜진 컴퓨트** (매일 데이터를 긁는 작업) — DB 선택과 무관하게 로컬 PC가 아니라 클라우드에서 돌아야 한다. 무료로는 **GitHub Actions cron**.
2. **공유 저장소** (수집기가 쓰고, 로컬 PC·대시보드가 읽는 곳) — 이게 DB 선택이다.

쓰는 주체가 클라우드면 **로컬 DuckDB 파일은 공유 저장소가 될 수 없다.** 그래서 호스팅 DB로 기운다. 그리고 대상 유니버스가 작아 네트워크 대량읽기도 느리지 않고 용량도 무료 티어로 충분하다 — 원래 DuckDB의 강점이던 "로컬·빠른 분석"의 이점이 이 프로젝트에선 결정타가 아니게 된다.

최종적으로 **Supabase**를 골랐다. 로드맵에 웹 대시보드 트랙이 크게 있어서, Auth·자동 REST API가 내장인 편이 나중 작업을 줄여준다. DuckDB는 버리지 않고 **나중에 로컬 테스트·분석용 옵션 백엔드**로 저장 계층 인터페이스 뒤에 남겨둘 여지를 열어뒀다.

## 3. 연결 만들기 — 그리고 접속문자열 3함정

이 단계의 목표는 하나다.

> `.env`의 Supabase 접속정보를 읽어 → 연결 → `SELECT 1`로 살아있음을 확인한다.

역할을 둘로 나눴다(작은 순수함수 원칙).

- `config.py` — **읽고 다듬기**(네트워크 없음, 순수). URL 정규화, 환경변수 가드.
- `db.py` — **붙기**. 엔진 생성, 연결 검증.

정규화는 접속문자열의 스킴을 psycopg3용(`postgresql+psycopg://`)으로 통일하고, `pgbouncer`처럼 psycopg가 이해하지 못하는 파라미터를 걸러낸다. 연결 검증은 그 URL로 만든 엔진에 `SELECT 1`을 던져 확인한다. 구현은 [`config.py`](https://github.com/tonygjwns-opt/quantproject_open/blob/main/src/portoptim/config.py)·[`db.py`](https://github.com/tonygjwns-opt/quantproject_open/blob/main/src/portoptim/db.py)에 있다.

### 실제로 밟은 3함정

Supabase는 접속문자열을 **여러 종류**로 준다. 연결 과정에서 셋을 순서대로 마주쳤다.

1. **Direct connection** (`db.<ref>.supabase.co:5432`) — **IPv6 전용**이다. IPv4만 되는 네트워크나 (나중의) GitHub Actions에서 연결이 실패할 수 있다.
2. **Transaction pooler** (`...pooler.supabase.com:6543`, `?pgbouncer=true`) — psycopg3가 곧바로 `invalid connection option "pgbouncer"` 에러를 뱉는다. 게다가 이 모드는 prepared statement와 충돌 소지가 있다.
3. **Session pooler** (`...pooler.supabase.com:5432`) — IPv4 호환이고 psycopg3와도 무난하다. 최종적으로 이걸 썼다.

정리하면 노려야 할 것은 **pooler 호스트 + 포트 5432 + 사용자 `postgres.<ref>`**. `pgbouncer` 파라미터는 손으로 매번 지우는 대신 `normalize_url`이 자동으로 걸러낸다.

자격증명은 `.env`(gitignore)에만 둔다. 커밋되는 건 값이 빠진 **`.env.example` 견본**뿐이다. 실제로 `git check-ignore`로 `.env`가 추적에서 빠지는지 확인하고서야 커밋했다.

## 코드

공개 엔진 저장소: **[quantproject_open](https://github.com/tonygjwns-opt/quantproject_open)**. 이 글에서 다룬 파일은 `src/portoptim/config.py`, `src/portoptim/db.py`, `tests/test_config.py`.

## 다음

다음 글에서는 **yfinance로 미국 시장부터 데이터를 취득**해 DB에 넣는다. 파이프라인의 맨 앞 `data` 칸이 채워지기 시작한다.
