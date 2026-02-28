---
title: Git Issue / Stash
subject: "[[Git]]"
reference: ""
date: 2026-02-26 14:20
description: ""
tags:
  - git
  - issue
  - stash
series: ""
seriesOrder:
published: false
---

# Git Issue / Stash

## 상황

- 작업 브랜치(`dev`)에서 커밋/푸시 안된 변경사항이 있음
- PR은 열려있고 이전 커밋들은 푸시된 상태
- 작업 중 버그 발견 → Issue로 관리하고 싶음
- 버그는 현재 작업 브랜치(`dev`)에서 발생

---

## 워크플로우

### 1. 현재 작업 임시 저장 (stash)

```bash
git stash push -m "WIP: 작업 중이던 내용"
```

### 2. Issue 열기

```bash
gh issue create --title "버그 제목" --body "버그 설명"
# 예: issue #42 생성됨
```

### 3. 현재 브랜치(`dev`)에서 fix 브랜치 분기

```bash
git checkout -b fix/42-버그이름
```

> `main`이 아닌 `dev`에서 분기하는 게 핵심

### 3-1. stash 내용이 버그를 유발하는 경우

stash에 보관한 코드가 버그의 원인이라면, fix 브랜치에서 바로 `stash pop`해서 에러를 재현해야 함

```bash
git stash pop
```

이 경우 stash는 fix 브랜치에서 소진되므로, 나중에 `dev`로 돌아올 때 `stash pop`은 필요 없음 → [6단계](#6-머지-후-원래-작업-복귀) 참고

### 4. 버그 수정 후 커밋/푸시

```bash
git add .
git commit -m "fix: 버그 설명 (closes #42)"
git push -u origin fix/42-버그이름
```

### 5. PR 열기 (base를 `dev`로 설정)

```bash
gh pr create --title "fix: 버그 설명" --base dev --body "Closes #42"
```

> `--base dev` 를 명시해야 `main`이 아닌 `dev`로 머지됨

#### PR 본문 작성 요령

- **Issue**: 문제 정의 (What / Why)
- **PR**: 해결 방법 (How) — 중복 설명 불필요

```markdown
## 변경 사항

`next.config.js`에 substackcdn.com 호스트 허용 추가

## 관련 이슈

Closes #42
```

`Closes #42` 키워드가 핵심 — 머지 시 Issue 자동 종료 + Issue ↔ PR 연결

같은 효과를 내는 키워드: `Fixes #42`, `Resolves #42`

#### `Closes`가 동작하지 않는 경우

GitHub는 **default 브랜치(main)** 로 머지될 때만 Issue를 자동으로 닫음
`dev`로 머지하면 `Closes #42`가 있어도 자동으로 닫히지 않음 → **수동으로 Close 필요**

### 6. fix 브랜치 삭제

GitHub에서 머지 후 "Delete branch" 버튼으로 원격 브랜치 삭제
로컬 브랜치는 수동 삭제:

```bash
git checkout dev
git branch -d fix/42-버그이름
```

> 브랜치를 삭제해도 추적은 가능 — Issue 번호와 `Closes` 키워드가 추적 수단이며,
> GitHub Issue 페이지에 연결된 PR 기록이 영구 보존됨

### 7. 머지 후 원래 작업 복귀

```bash
git checkout dev
git pull  # fix 내용 반영
git stash pop
```

> 3-1을 거쳐 stash를 fix 브랜치에서 이미 소진했다면, `stash pop`은 생략

---

## stash pop 후 conflict 발생 시

### 상황

```bash
git stash pop
# CONFLICT (content): Merge conflict in src/foo.ts
```

### 해결 순서

**1. conflict 파일 확인**

```bash
git status
# both modified: src/foo.ts
```

**2. 파일 열어서 직접 수정**

```
<<<<<<< Updated upstream    ← dev에 머지된 fix 내용
fix 브랜치에서 수정한 코드
=======
내가 stash에 보관했던 WIP 코드
>>>>>>> Stashed changes
```

원하는 내용으로 정리하고 마커(`<<<<`, `====`, `>>>>`) 제거

**3. 해결 후**

```bash
git add src/foo.ts
git stash drop  # stash 항목 수동으로 제거
```

> `stash pop` = `stash apply` + `stash drop` 인데,
> conflict 나면 `drop`이 자동으로 안 되므로 수동으로 해줘야 함

---

## conflict 예방

stash pop 전에 fix 내용과 stash 내용이 겹치는지 미리 확인:

```bash
git diff dev stash@{0}
```
 