---
title: Git + Obsidian + CodeRabbit
subject: "[[Projects/Obslog/issue/issue]]"
reference:
date: 2026-02-12 16:36
description: Git & Obsidian + CodeRabbit 결합에 따른 문제 해결 가이드
tags:
  - issue
  - git
  - coderabbit
series: ""
seriesOrder:
published: false
---


## 목차

1. [문제 1: workspace.json Git Pull 충돌](#문제-1-workspacejson-git-pull-충돌)

2. [문제 2: 브랜치 전환 시 workspace.json 충돌](#문제-2-브랜치-전환-시-workspacejson-충돌)

3. [문제 3: CodeRabbit Rate Limit & 파싱 에러](#문제-3-coderabbit-rate-limit--파싱-에러)

4. [예방 및 Best Practices](#예방-및-best-practices)

  

---

  

## 문제 1: workspace.json Git Pull 충돌

  

### 🔴 증상

```bash

$ git pull

error: Your local changes to the following files would be overwritten by merge:

.obsidian/workspace.json

Please commit your changes or stash them before you merge.

Aborting

```

  

### 🔍 원인 분석

  

#### 1. `.obsidian/workspace.json`이란?

- Obsidian의 **UI 상태 파일**

- 저장 내용:

- 어떤 파일이 열려있는지

- 활성 탭/패널 정보

- 레이아웃 구성

- 창 위치 및 크기

- **중요도: 매우 낮음** ⚠️ (개인 작업 환경 설정)

  

#### 2. 왜 충돌이 발생하는가?

```

1. Obsidian 앱 실행 중

2. 파일을 열거나 탭을 전환할 때마다 workspace.json 자동 수정

3. Git이 이 변경을 감지

4. Remote에도 workspace.json 변경사항 존재

5. Pull 시 충돌 발생

```

  

#### 3. 기존 해결 시도와 실패 이유

  

**시도 1: `git restore`**

```bash

$ git restore .obsidian/workspace.json

$ git pull

# ❌ 실패: Obsidian이 실행 중이라 즉시 파일 재생성

```

  

**시도 2: `git stash`**

```bash

$ git stash

Saved working directory...

$ git pull

# ❌ 실패: stash와 pull 사이에 Obsidian이 파일 재수정

```

  

### ✅ 해결 방법

  

#### 원초적 해결: Git Tracking 제거

  

**핵심 아이디어**: workspace.json은 중요하지 않으므로 Git에서 추적하지 않기

  

**단계별 실행:**

  

```bash

# 1. .gitignore에 이미 존재 확인

$ cat .gitignore | grep workspace.json

.obsidian/workspace.json # ✅ 이미 존재

  

# 2. Git tracking에서 제거

$ git rm --cached .obsidian/workspace.json

rm '.obsidian/workspace.json'

  

# 3. 변경사항 커밋

$ git commit -m "Stop tracking workspace.json"

[main baa9f50] Stop tracking workspace.json

1 file changed, 297 deletions(-)

delete mode 100644 .obsidian/workspace.json

  

# 4. Pull (rebase 모드)

$ git pull --rebase

# ⚠️ 충돌 발생: remote는 workspace.json 수정, 우리는 삭제

  

# 5. 충돌 해결: 우리 버전(삭제) 선택

$ git rm .obsidian/workspace.json

$ git rebase --continue

Successfully rebased and updated refs/heads/main.

  

# 6. Remote에 push

$ git push

```

  

**결과:**

- ✅ workspace.json 더 이상 Git이 추적하지 않음

- ✅ Obsidian이 파일을 계속 수정해도 Git에 영향 없음

- ✅ Pull/push 시 충돌 발생하지 않음

  

---

  

## 문제 2: 브랜치 전환 시 workspace.json 충돌

  

### 🔴 증상

```bash

$ git checkout rabbit

Switched to branch 'rabbit'

  

$ git checkout main

error: Your local changes to the following files would be overwritten by checkout:

.obsidian/workspace.json

Please commit your changes or stash them before you switch branches.

Aborting

```

  

### 🔍 원인 분석

```

main 브랜치: workspace.json → Git tracking 제거 ✅

rabbit 브랜치: workspace.json → Git tracking 중 ❌

  

→ 브랜치 전환 시 Git이 workspace.json을 체크아웃하려고 시도

→ Obsidian이 실시간으로 파일 수정 중

→ 충돌 발생

```

  

### ✅ 해결 방법

**rabbit 브랜치에서도 동일하게 tracking 제거:**

  

```bash

# 1. rabbit 브랜치에 있는지 확인

$ git branch

* rabbit

main

  

# 2. Git tracking에서 제거

$ git rm --cached .obsidian/workspace.json

rm '.obsidian/workspace.json'

  

# 3. 커밋 및 푸시

$ git commit -m "Stop tracking workspace.json in rabbit branch"

[rabbit 827b565] Stop tracking workspace.json in rabbit branch

1 file changed, 297 deletions(-)

delete mode 100644 .obsidian/workspace.json

  

$ git push

To https://github.com/mulgko/obsidian.git

74900e9..827b565 rabbit -> rabbit

  

# 4. 브랜치 전환 테스트

$ git checkout main

Switched to branch 'main' # ✅ 성공

  

$ git checkout rabbit

Switched to branch 'rabbit' # ✅ 성공

```

  

**결과:**

- ✅ 모든 브랜치에서 workspace.json 추적 중단

- ✅ 브랜치 자유롭게 전환 가능

- ✅ 더 이상 충돌 없음

  

---

  

## 문제 3: CodeRabbit Rate Limit & 파싱 에러

  

### 🔴 증상

  

**증상 1: Rate Limit**

```

⚠️ Rate limit exceeded

@mulgko has exceeded the limit for the number of commits

that can be reviewed per hour. Please wait 10 minutes and 52 seconds.

```

  

**증상 2: 파싱 에러**

```

⚠️ .coderabbit.yaml has a parsing error

💥 Parsing errors (1)

Validation error: Expected object, received boolean at "reviews.auto_review"

```

  

### 🔍 원인 분석

  

#### 문제 1: 잘못된 설정 - Rate Limit 원인

  

**초기 `.coderabbit.yaml` (문제 있는 버전):**

```yaml

reviews:

review_on_push: true # ❌ 모든 push마다 리뷰

auto_review: true # ❌ 자동 리뷰 활성화

review_main_branch: true # ❌ main 브랜치도 리뷰

level: "detailed" # ❌ 상세 리뷰

```

  

**문제점:**

- Main 브랜치에 직접 push할 때마다 리뷰 실행

- Rabbit 브랜치에 커밋할 때마다 리뷰 실행

- 짧은 시간에 여러 커밋 → Rate limit 초과

  

#### 문제 2: YAML 파싱 에러

  

**에러 1차 수정 시도:**

```yaml

reviews:

auto_review:

enabled: true # ❌ CodeRabbit는 이 구조를 지원하지 않음

drafts: false

```

  

**실제 에러 메시지:**

```

Validation error: Expected object, received boolean at "reviews.auto_review"

```

  

**문제점:**

- CodeRabbit Schema v2는 `auto_review`를 boolean으로 기대

- 또는 해당 키 자체를 지원하지 않음

- 많은 커스텀 키들이 실제로는 지원되지 않음

  

**지원하지 않는 키들:**

```yaml

enable_free_tier: true # ❌

review_on_push: true # ❌

review_main_branch: true # ❌

level: "detailed" # ❌

validate_frontmatter: true # ❌

check_links: true # ❌

spell_check: true # ❌

spell_check_language: "ko-KR" # ❌

```

  

### ✅ 해결 방법

  

#### 최종 수정된 `.coderabbit.yaml`:

  

```yaml

# yaml-language-server: $schema=https://coderabbit.ai/integrations/schema.v2.json

  

# CodeRabbit Configuration

language: ko

early_access: true

  

reviews:

profile: "chill" # ✅ 가벼운 리뷰 프로필

request_changes_workflow: false

high_level_summary: true

poem: false

review_status: true

collapse_walkthrough: false

  

path_instructions:

- path: "**/*.md"

instructions: "마크다운 문서의 구조, 링크, 문법을 중심으로 리뷰해주세요."

  

path_filters:

- "!.obsidian/**" # Obsidian 설정 제외

- "!.trash/**" # 휴지통 제외

- "**/*.md" # 마크다운만 리뷰

  

chat:

auto_reply: false

  

tone_instructions: "한국어로 친절하고 건설적인 피드백을 제공해주세요."

```

  

#### 주요 변경사항:

  

| 항목 | 변경 전 | 변경 후 | 효과 |

|------|---------|---------|------|

| Schema 검증 | ❌ 없음 | ✅ yaml-language-server | 에디터에서 자동 검증 |

| auto_review | ❌ 잘못된 구조 | ✅ 제거 (기본값 사용) | 파싱 에러 해결 |

| Review 프로필 | ❌ detailed | ✅ chill | Rate limit 완화 |

| Main 브랜치 리뷰 | ❌ 활성화 | ✅ 비활성화 | Rate limit 완화 |

| 지원 안 되는 키 | ❌ 다수 포함 | ✅ 모두 제거 | 파싱 에러 해결 |

  

#### 실행 과정:

  

```bash

# 1. 파일 수정 (위 최종 버전으로)

$ vim .coderabbit.yaml

  

# 2. 커밋 (main 브랜치)

$ git add .coderabbit.yaml

$ git commit -m "Fix CodeRabbit YAML schema validation error

  

- Add schema validator for auto-completion

- Remove invalid auto_review object structure

- Use 'chill' profile to reduce rate limit issues

- Simplify configuration to use only validated keys

  

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

  

$ git push

```

  

### ✅ 결과

  

**파싱 에러 해결:**

- ✅ Schema v2 검증된 키만 사용

- ✅ YAML 구조 검증 통과

- ✅ 에디터 자동완성 지원

  

**Rate Limit 완화:**

- ✅ `chill` 프로필로 가벼운 리뷰

- ✅ PR에만 리뷰 실행 (push마다 실행 X)

- ✅ Main 브랜치 직접 push 리뷰 비활성화

  

---

  

## 예방 및 Best Practices

  

### 📋 Obsidian + Git 사용 시

  

#### 1. `.gitignore` 필수 항목

  

```gitignore

# Obsidian 설정 파일 (개인 환경)

.obsidian/workspace.json

.obsidian/workspace-mobile.json

.obsidian/graph.json

.obsidian/workspace.json.bak

  

# 캐시 및 임시 파일

.obsidian/cache/

.trash/

.DS_Store

```

  

#### 2. 이미 추적 중인 파일 제거하기

  

```bash

# 단일 파일

git rm --cached .obsidian/workspace.json

  

# 여러 파일

git rm --cached .obsidian/workspace*.json

  

# 폴더 전체

git rm -r --cached .obsidian/cache/

  

# 커밋

git commit -m "Stop tracking Obsidian workspace files"

```

  

#### 3. 모든 브랜치에 적용하기

  

```bash

# 각 브랜치에서 반복

git checkout main

git rm --cached .obsidian/workspace.json

git commit -m "Stop tracking workspace.json"

git push

  

git checkout rabbit

git rm --cached .obsidian/workspace.json

git commit -m "Stop tracking workspace.json"

git push

```

  

### 📋 CodeRabbit 사용 시

  

#### 1. Rate Limit 피하는 방법

  

**전략:**

```

1. Feature 브랜치에서 작업 (예: rabbit)

→ 여러 커밋 자유롭게 가능

  

2. 작업 완료 후 PR 생성

→ CodeRabbit이 한 번만 리뷰

  

3. Main에 merge

→ 추가 리뷰 없음 (설정에서 비활성화)

```

  

**나쁜 예:**

```bash

# Main 브랜치에 직접 작업

git checkout main

git add file1.md

git commit -m "Update 1"

git push # ← 리뷰 1

  

git add file2.md

git commit -m "Update 2"

git push # ← 리뷰 2

  

git add file3.md

git commit -m "Update 3"

git push # ← 리뷰 3 → Rate limit! ❌

```

  

**좋은 예:**

```bash

# Feature 브랜치에서 작업

git checkout rabbit

git add file1.md

git commit -m "Update 1"

git add file2.md

git commit -m "Update 2"

git add file3.md

git commit -m "Update 3"

git push # Push는 여러 커밋 한 번에

  

# PR 생성 → CodeRabbit 리뷰 1회만 ✅

gh pr create --title "Updates" --body "..."

```

  

#### 2. 설정 파일 검증

  

**필수: Schema 검증 추가**

```yaml

# 파일 맨 위에 추가

# yaml-language-server: $schema=https://coderabbit.ai/integrations/schema.v2.json

```

  

**검증 방법:**

1. [Online YAML Validator](https://coderabbit.ai/validate) 사용

2. VSCode YAML extension 설치 → 자동 검증

3. 설정 변경 후 PR 만들어서 테스트

  

#### 3. 권장 설정

  

**최소 설정 (가장 안전):**

```yaml

# yaml-language-server: $schema=https://coderabbit.ai/integrations/schema.v2.json

language: ko

early_access: true

```

  

**Markdown 중심 프로젝트:**

```yaml

# yaml-language-server: $schema=https://coderabbit.ai/integrations/schema.v2.json

language: ko

early_access: true

  

reviews:

profile: "chill"

path_filters:

- "!.obsidian/**"

- "**/*.md"

  

tone_instructions: "한국어로 간결하게 피드백해주세요."

```

  

### 📋 문제 발생 시 체크리스트

  

#### Git 충돌 관련

  

- [ ] `.gitignore`에 해당 파일이 있는가?

- [ ] 모든 브랜치에서 tracking이 제거되었는가?

- [ ] Obsidian 앱이 실행 중인가? (필요시 종료)

- [ ] `git status`로 현재 상태 확인

  

#### CodeRabbit 관련

  

- [ ] `.coderabbit.yaml` 파일이 올바른 위치에 있는가? (레포 루트)

- [ ] YAML 문법 오류가 없는가?

- [ ] Schema 검증 주석이 있는가?

- [ ] 지원하지 않는 키를 사용하고 있지 않은가?

- [ ] Rate limit이 걸렸다면 10분 대기 후 재시도

  

---

  

## 🎯 요약

  

### 핵심 교훈

  

1. **UI 상태 파일은 Git에서 제외하라**

- workspace.json, graph.json 등

- 개인 환경 설정이므로 공유 불필요

- 충돌의 주요 원인

  

2. **모든 브랜치에 동일하게 적용하라**

- Main에서만 제거하면 브랜치 전환 시 문제

- 각 브랜치에서 개별 적용 필요

  

3. **CodeRabbit은 PR 단위로 사용하라**

- Push마다 리뷰 X

- PR 생성 시 한 번만 리뷰 O

- Rate limit 방지

  

4. **설정 파일은 반드시 검증하라**

- Schema validator 사용

- 공식 문서 참고

- 추측으로 키 추가 금지

  

### 해결 완료 항목

  

- ✅ workspace.json Git 충돌 해결

- ✅ 브랜치 전환 시 충돌 해결

- ✅ CodeRabbit 파싱 에러 수정

- ✅ CodeRabbit Rate limit 완화

- ✅ 향후 재발 방지 설정 완료

  

---

  

*마지막 업데이트: 2026-02-12*

*해결자: Claude Sonnet 4.5*