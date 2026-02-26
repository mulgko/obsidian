---
title: Github Workflow
subject: "[[Git]]"
reference: ""
date: 2026-02-26 13:54
description: ""
tags:
  - git
series: ""
seriesOrder:
published: false
---

# Github Workflow

## Conventional Commits (커밋 타입)

| 타입 | 설명 |
|------|------|
| `feat` | 새로운 기능 추가 |
| `fix` | 버그 수정 |
| `docs` | 문서 변경 (README, 주석 등) |
| `style` | 코드 포맷팅, 세미콜론 누락 등 (로직 변경 없음) |
| `refactor` | 기능 변경 없는 코드 리팩토링 |
| `test` | 테스트 코드 추가/수정 |
| `chore` | 빌드 프로세스, 패키지 매니저 설정 등 |
| `perf` | 성능 개선 |
| `ci` | CI/CD 설정 변경 |
| `build` | 빌드 시스템, 의존성 변경 |
| `revert` | 이전 커밋 되돌리기 |

### 스코프 추가 (선택)
```
feat(auth): 소셜 로그인 기능 추가
fix(api): 타임아웃 오류 수정
```

### BREAKING CHANGE
```
feat!: API 응답 형식 변경
```

---

## GitHub Issues

**프로젝트에서 해야 할 일이나 문제를 기록하는 게시판**

### 주로 쓰는 경우
| 용도 | 예시 |
|------|------|
| 버그 신고 | "로그인 버튼 클릭 시 오류 발생" |
| 기능 요청 | "다크모드 추가해줬으면 좋겠어요" |
| 할 일 관리 | "리팩토링 필요한 부분 정리" |
| 질문/토론 | "이 부분 어떻게 구현할까요?" |

### Issue vs PR
```
Issue  → "이런 문제가 있어요 / 이런 기능 필요해요"  (계획/논의)
PR     → "이렇게 코드 고쳤어요"                    (실제 코드 변경)
```

### Issue와 PR 연결 (PR 본문에 작성)
```
Closes #12
Fixes #12
Resolves #12
```
→ PR 머지 시 Issue 자동으로 닫힘

### Issue 만드는 법 (gh CLI)
```bash
gh issue create
gh issue create --title "버그: 로그인 오류" --body "클릭 시 500 에러 발생"
gh issue create --title "기능 요청: 다크모드" --label "enhancement"
```

---

## 전체 워크플로우

```
Issue 생성 (무엇을 할지 기록)
    ↓
브랜치 생성 (실제 작업 공간)
    ↓
코드 작업
    ↓
PR 생성 (작업 결과물 제출) + Closes #이슈번호
    ↓
PR 머지 → main 업데이트 + Issue 자동 닫힘
```

### 브랜치 네이밍 관례
```bash
git checkout -b feat/12-기능명
git checkout -b fix/12-버그명
git checkout -b refactor/12-내용
```

---

## main 최신화 받기

PR 머지 후 다른 개발자가 최신 코드 받는 법:

```bash
git checkout main
git pull origin main
```

### 작업 중 main 변경사항 반영
```bash
# 방법 1: merge
git merge main

# 방법 2: rebase (히스토리 깔끔)
git rebase main
```

---

## Merge vs Rebase

### Merge
```
main:   A → B → C
                 ↘
branch: D → E → Merge commit
```
- 머지 커밋이 새로 생김
- 히스토리 그대로 보존 (언제 어디서 합쳤는지 기록)

### Rebase
```
main:   A → B → C
                 ↓
branch:          D' → E'
```
- 브랜치 시작점을 main 최신으로 옮겨 붙임
- 히스토리가 일직선으로 깔끔해짐

### 언제 뭘 쓸까?
| 상황 | 추천 |
|------|------|
| 팀 협업, 공식 머지 | `merge` |
| PR 올리기 전 main 최신화 | `rebase` |
| 히스토리 깔끔하게 유지 | `rebase` |
| 충돌 추적이 중요할 때 | `merge` |

### 주의
> rebase는 커밋을 **재작성**하기 때문에, 다른 사람과 공유된 브랜치에서 사용하면 충돌이 생길 수 있음.
> 혼자 작업하는 브랜치에서만 사용 권장.
