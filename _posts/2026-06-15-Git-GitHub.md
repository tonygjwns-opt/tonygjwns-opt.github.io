---
title: Git / GitHub
date: 2026-06-15 10:00:00 +0900
categories: [컴공, 기타]
tags: [git, github, 버전관리]
---

## 본질

- **Git**: 버전 관리 시스템(VCS). 파일을 개별 관리하는 대신 파일을 추적하는 repository로 관리한다. 이력을 전부 기록하므로 → **과거 버전으로 되돌리거나(rollback), 잘못된 작업을 수정·취소**할 수 있고, 한 파일을 여러 갈래로 수정하거나 여러 명이 동시에 작업할 수 있다.
- **GitHub**: Git 저장소를 올려두는 클라우드(코드 저장·공유·협업), repository hosting 서비스. git은 '개발 도구', GitHub는 그 도구로 일하는 '서비스' — GitHub 없이도 git은 된다.

## 핵심 구조

commit들은 시간순 + 각자 SHA 해시(앞 7자)로 이어진 사슬이다(대충 linked list 느낌). **HEAD** = 지금 있는 commit(보통 최신), 브랜치는 이 사슬에서 갈라지는 다른 가지.

**세 영역 + 원격**

- **Working Directory**: 실제 작업(생성/수정/삭제)하는 파일들.
- **Staging Area(index)**: 다음 commit에 담을 변화만 골라 담아두는 '선별·대기' 공간. (작업은 Working에서 하고, `git add`로 그중 commit할 것만 올림.)
- **Repository(.git)**: 프로젝트의 여러 버전(commit)을 저장하는 확정된 이력.
- **Remote**: 원격 저장소(GitHub 등). clone해온 원본은 `origin`이라 부른다. — `origin`=저장소 별명(어디), `main`=브랜치(무엇). 그 정체(URL/로컬경로)는 `git remote -v`로 확인.

**흐름 (두 축)**

1. **저장 축**: 수정(working) → `git add`(staging) → `git commit`(local repo) → `git push`(remote)
2. **분기 축**: `git branch`/`checkout`로 새 갈래(브랜치)를 떠서 작업·commit → 합칠 때:
   - 혼자: main으로 돌아와(`git checkout main`) `git merge 브랜치`
   - 협업: 브랜치를 push하고 GitHub에서 Pull Request → 리뷰 후 merge

평소엔 ①로 저장하고, 독립적인 작업·실험·협업은 ②로 브랜치를 떠서 한 뒤 main에 합친다.

## A. 기본 워크플로 — 로컬 폴더를 깃허브에 올리기

GitHub에서 빈 repo 생성 → 그 폴더에서 터미널:

```bash
git init                 # 이 폴더를 git 저장소로(아직 추적 시작은 아님)
git add .                # 전부 staging (특정 파일만은: git add 파일1 파일2)
git commit -m "메시지"    # -m이면 50자 내, 현재 시제 권장
git remote add origin <url>
git push -u origin main  # 옛 저장소는 master
```

상태·이력 확인:

```bash
git status   # 현재 상태 (untracked = 감지는 하지만 아직 추적 안 함)
git diff     # 아직 staging 안 된 변경 내용
git log      # commit 이력(시간순)
```

## B. 기존/남의 repo 가져오기

```bash
git clone <url> [이름]   # init + remote + pull을 한 번에. 원본은 origin
```

## C. 되돌리기 (backtrack)

- `git show HEAD` : HEAD commit 내용 확인.
- `git checkout HEAD 파일` (= `git checkout -- 파일`) : working dir 파일을 마지막 commit 상태로 되돌림.
- `git reset HEAD 파일` : staging에서 빼되(unstage) working 변화는 유지(2단계만 되돌림).
- `git reset <SHA 7자>` : HEAD를 그 commit으로 이동(이후 commit은 떨어져 나감). 기본은 working 변화 유지, `--hard`면 working까지 되돌림.
- `git stash` / `git stash pop` : 현재 변화를 잠시 치워두고 다른 작업·commit 후 pop으로 복원.
- `git commit --amend` : 방금 add한 걸 직전 commit에 덮어씀(메시지도 수정). `--no-edit`이면 메시지 유지.

## D. 브랜치 (갈래 작업)

- `git branch` : 브랜치 목록(현재 것 위에 `*`).
- `git branch 이름` : 새 브랜치 생성(공백 X). 기존 commit 그대로 이어받음. 자동 이동은 안 함.
- `git checkout 이름` : 그 브랜치로 이동. (`git checkout -b 이름` = 생성+이동)
- `git merge 이름` : 그 브랜치를 현재(보통 main)에 합침 — 먼저 main으로 돌아가서 실행.
  - **merge 방식**:
    - *fast-forward*: 갈라진 뒤 main에 새 commit이 없으면 main 포인터를 브랜치 끝으로 앞당김(merge commit 없음, 선형).
    - *merge commit(3-way)*: 양쪽 다 새 commit이 있으면 둘을 합치는 부모 2개짜리 merge commit을 새로 만듦.
  - **자동 병합 vs conflict**: 서로 다른 파일/다른 줄을 고쳤으면 git이 둘 다 자동으로 합친다. 같은 파일의 **같은 부분**을 양쪽이 다르게 고쳤을 때만 **conflict** → 파일에 `<<<<` `====` `>>>>` 표시가 생기고, 남길 내용을 직접 정리한 뒤 `add`/`commit`으로 마무리.
- `git branch -d 이름` : 삭제(merge 안 한 건 `-D` 대문자).

## E. 협업 (remote)

- 공유 저장소를 `git clone <remote> <이름>`으로 복제(remote는 GitHub 주소나 내 컴퓨터 폴더). 원본은 `origin`.
- `git remote -v` : 등록된 remote 확인(fetch용·push용이 보임).
- `git fetch` : remote 변화를 내 local로 가져옴(바로 merge 안 하고 remote-tracking branch만 갱신) → 그 뒤 `git merge`로 합침.

팀 흐름:

```text
fetch&merge로 최신화 → 내 브랜치 만들어 작업 → commit → 다시 fetch&merge → git push origin 내_브랜치
```

## F. GitHub 협업 흐름 (Pull Request)

**Pull Request(PR)** = "내 브랜치를 main에 합쳐달라"는 요청. git 기본 기능이 아니라 **GitHub의 기능**이다.
핵심 목적: **main을 직접 건드리지 않고**, 합치기 전에 리뷰·자동검사(CI) 게이트를 거쳐 품질을 지키고 변경 이력·논의를 남긴다.

한 사이클:

1. main을 최신화하고 기능별 브랜치를 만든다.

   ```bash
   git checkout main && git pull
   git checkout -b feature/login   # 작업용 브랜치
   ```

2. 작업하며 작은 단위로 여러 번 commit.
3. 브랜치를 GitHub에 올린다.

   ```bash
   git push -u origin feature/login
   ```

4. GitHub에서 **Pull Request** 생성 → 제목·설명에 *무엇을 왜* 바꿨는지 적는다. (충분히 작은 단위로 요청.)
5. 동료가 리뷰: 코드 줄에 코멘트, **승인(approve)** 또는 **변경요청(request changes)**. 설정돼 있으면 CI(자동 테스트)도 함께 돌아간다.
6. 지적 사항은 **같은 브랜치에 commit을 더 쌓아 push** → PR이 자동으로 갱신된다(새 PR을 만들지 않음).
7. 승인되면 **merge** → main에 반영. 그 뒤 안 쓰는 브랜치 삭제, 로컬 main을 `git pull`로 최신화.

PR을 merge할 때 GitHub 버튼은 세 가지 — 이력을 어떻게 남길지가 다르다:

- **Merge commit**: 브랜치 commit 다 보존 + merge commit 1개(3-way).
- **Squash and merge**: 브랜치의 여러 commit을 하나로 뭉쳐 main에 1 commit(이력 깔끔).
- **Rebase and merge**: 브랜치 commit을 main 끝에 재배치 → merge commit 없이 선형.

> 혼자 작업하면 PR 없이 main에 바로 push해도 된다(이 블로그가 그 예). PR은 협업/오픈소스 기여처럼 **리뷰가 필요할 때** 쓴다.
{: .prompt-tip }

## G. 자주 쓰는 명령 요약

```bash
git init / git add 파일(.) / git status / git commit -m "..." / git log
git clone url / git diff / git reset 파일 / git rm 파일 / git mv 기존 신규 / git log --stat -M
```

## H. 자주 겪는 문제

- **SSH key 미설정**: key를 GitHub에 안 올렸거나 agent에 안 더하면 인증 에러.
- **이미 파일 있는 repo에 push 거절**: 내 첫 commit과 GitHub의 initial commit이 별개라(공통 조상 없음) 거부됨 → 빈 repo로 시작하거나 먼저 `pull`로 맞추기. (사슬의 시작점이 같아야 함)
- **인증**: 요즘 GitHub는 비밀번호 대신 토큰(PAT)/SSH 키를 씀. 외부에서 push 시 로그인 요구.

## I. 단축어 (alias)

```bash
git config --global alias.co "checkout"   # 이후 git co 브랜치명 처럼 사용
```
