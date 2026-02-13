---
title: Gihub Actions로 Submodule 자동 업데이트 가이드
subject: "[[Projects/Obslog/issue/issue]]"
reference: "[[Github Actions]]"
date: 2026-02-12 17:43
description: guide
tags:
  - issue
  - githubActions
  - git
  - submodule
---

생각을 해보니 옵시디언에서 git push를 했을 경우 Obslog에 최신 상황이 업데이트 되어야 했다. 또 Obslog가 옵시디언 콘텐츠를 가져올 수 있어야 했다. 그러므로 Obslog 프로젝트 안에 옵시디언 콘텐츠를 넣고, 사용하는 방법을 생각했는데 이렇게 되면 재사용성이 많이 떨어지는 문제가 생긴다. 다른 프로젝트에서도 옵시디언을 사용하고 싶을 수도 있기 때문에 이렇게 사용하면 안 되겠다는 생각이 들었다. 두번째 방법은 옵시디언 레파지토리를 서브모듈로 분리하는 것. 이렇게 되면, 서로 각각 다른 레포로 별개의 운영을 할 수 있다. 하지만 단점으로는 배포가 어렵다는 것인데 이를 githubActions를 사용해서 해결할 수 있다. submodule을 업데이트해도 Obslog가 업데이트 되는 방식이다. 



> Obsidian에서 push하면 자동으로 Next.js 블로그 배포까지!

  

## 📋 목차

  

- [개요](#개요)

- [전체 구조](#전체-구조)

- [Step 1: 레포지토리 분리](#step-1-레포지토리-분리)

- [Step 2: GitHub Token 발급](#step-2-github-token-발급)

- [Step 3: obslog-content Actions 설정](#step-3-obslog-content-actions-설정)

- [Step 4: obslog Actions 설정](#step-4-obslog-actions-설정)

- [Step 5: 테스트](#step-5-테스트)

- [트러블슈팅](#트러블슈팅)

  

---

  

## 개요

  

### 문제점

```text

Obsidian → obslog-content 레포 push

obslog 메인 레포는 변경 감지 못함

Vercel 배포 안 됨 ❌

```

  

### 해결책

```text

Obsidian → obslog-content push

↓

🤖 GitHub Actions 자동 실행

↓

obslog 레포 submodule 업데이트

↓

Vercel 자동 배포 ✅

```

  

---

  

## 전체 구조



```text

github.com/username/obslog-content (마크다운만)

└── posts/

└── *.md



github.com/username/obslog (Next.js 프로젝트)

├── content/ (submodule → obslog-content)

├── src/

└── .github/workflows/

└── update-submodule.yml 🤖 자동화

```

  

---

  

## Step 1: 레포지토리 분리

  

### 1.1 obslog-content 레포 생성

  

```bash

# 현재 content를 새 레포로

cd /Users/gimdogyeong/projects/obslog/content

  

# Git 초기화

git init

git add .

git commit -m "Initial content"

  

# GitHub에서 obslog-content 레포 생성 후

git remote add origin https://github.com/username/obslog-content.git

git branch -M main

git push -u origin main

```

  

### 1.2 obslog에서 submodule로 연결

  

```bash

cd /Users/gimdogyeong/projects/obslog

  

# 기존 content 제거

git rm -rf content

git commit -m "Remove content folder"

  

# Submodule로 추가

git submodule add https://github.com/username/obslog-content.git content

git commit -m "Add content as submodule"

git push

```

  

---

  

## Step 2: GitHub Token 발급

  

### 2.1 Personal Access Token 생성

  

1. GitHub → **Settings** (프로필 설정)

2. 왼쪽 하단 → **Developer settings**

3. **Personal access tokens** → **Tokens (classic)**

4. **Generate new token (classic)** 클릭

  

**Token 설정:**

```text

Note: obslog-content-to-parent

Expiration: 90 days (90일마다 갱신 권장)



⚠️ 권장: Fine-grained Personal Access Token 사용
- Repository access: obslog 레포만 선택
- Repository permissions:
  ✓ Contents: Read and write
  ✓ Metadata: Read-only (자동)
  ✓ Workflows: Read and write (Actions 트리거용)

Classic Token을 사용하는 경우 최소 권한:
✓ repo (또는 public_repo - public 레포만 해당)
✓ workflow

```



> 📚 **참고:** [GitHub Fine-grained PAT 공식 문서](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token)

> ⏰ **토큰 만료 시:** 90일마다 토큰 재발급 후 obslog-content 레포의 PARENT_REPO_TOKEN Secret 업데이트 필요



5. **Generate token** 클릭

6. **토큰 복사** (한 번만 보임! 꼭 저장하세요)

  

### 2.2 obslog-content 레포에 Secret 추가

  

1. **obslog-content 레포** → Settings → Secrets and variables → Actions

2. **New repository secret** 클릭

3. 입력:

```

Name: PARENT_REPO_TOKEN

Secret: (위에서 복사한 토큰)

```

4. **Add secret** 클릭

  

---

  

## Step 3: obslog-content Actions 설정

  

### 3.1 Workflow 파일 생성

  

```bash

cd /path/to/obslog-content

  

# 폴더 생성

mkdir -p .github/workflows

  

# 파일 생성

touch .github/workflows/notify-parent.yml

```

  

### 3.2 notify-parent.yml 작성

  

```yaml

name: Notify Parent Repo

  

on:

push:

branches: [main]

  

jobs:

notify:

runs-on: ubuntu-latest

steps:

- name: Trigger parent repo update

run: |

curl -X POST \

-H "Accept: application/vnd.github+json" \

-H "Authorization: token ${{ secrets.PARENT_REPO_TOKEN }}" \

https://api.github.com/repos/YOUR_USERNAME/obslog/dispatches \

-d '{"event_type":"update-submodule"}'

  

- name: Log success

run: echo "✅ Successfully notified obslog repo"

```

  

**⚠️ 중요: `YOUR_USERNAME` 부분을 본인 GitHub 사용자명으로 변경!**

  

### 3.3 Commit & Push

  

```bash

git add .github/workflows/notify-parent.yml

git commit -m "Add GitHub Actions workflow"

git push

```

  

---

  

## Step 4: obslog Actions 설정

  

### 4.1 Workflow 파일 생성

  

```bash

cd /Users/gimdogyeong/projects/obslog

  

# 폴더 생성 (이미 있을 수 있음)

mkdir -p .github/workflows

  

# 파일 생성

touch .github/workflows/update-submodule.yml

```

  

### 4.2 update-submodule.yml 작성

  

```yaml

name: Update Content Submodule

  

on:

repository_dispatch:

types: [update-submodule]

  

jobs:

update:

runs-on: ubuntu-latest

steps:

- name: Checkout repository

uses: actions/checkout@v4

with:

submodules: true

token: ${{ secrets.GITHUB_TOKEN }}

  

- name: Update submodule

run: |

git config user.name "GitHub Actions Bot"

git config user.email "github-actions[bot]@users.noreply.github.com"

  

# Submodule 최신 버전으로 업데이트

git submodule update --remote content

  

# 변경사항이 있는지 확인

if git diff --quiet content; then

echo "No changes in submodule"

exit 0

fi

  

# Commit & Push

git add content

git commit -m "chore: Update content submodule [automated]"

git push

  

- name: Log success

run: echo "✅ Submodule updated successfully"

```

  

### 4.3 Commit & Push

  

```bash

git add .github/workflows/update-submodule.yml

git commit -m "Add submodule auto-update workflow"

git push

```

  

---

  

## Step 5: 테스트

  

### 5.1 Obsidian에서 테스트

  

1. **Obsidian에서 obslog-content 레포 열기**

```bash

git clone https://github.com/username/obslog-content.git ~/Documents/Obsidian/MyBlog

```

  

2. **Obsidian에서 폴더 열기**

```text

Obsidian → Open folder as vault → ~/Documents/Obsidian/MyBlog

```

  

3. **Obsidian Git Plugin 설치**

```text

Settings → Community Plugins → Browse → "Obsidian Git" 검색 & 설치

```

  

4. **새 글 작성**

```markdown

---

title: "GitHub Actions 테스트"

published: true

createdAt: "2026-02-11"

---

  

# 자동 배포 테스트

  

이 글이 자동으로 배포되는지 확인!

```

  

5. **Obsidian Git으로 push**

```text

Ctrl/Cmd + P → "Obsidian Git: Commit and push"

```

  

### 5.2 GitHub Actions 확인

  

**obslog-content 레포:**

```text

GitHub → obslog-content 레포 → Actions 탭

└─> "Notify Parent Repo" 워크플로우 실행 확인

└─> 녹색 체크 = 성공!

```

  

**obslog 레포:**

```text

GitHub → obslog 레포 → Actions 탭

└─> "Update Content Submodule" 워크플로우 실행 확인

└─> 녹색 체크 = 성공!

└─> 최신 커밋: "chore: Update content submodule [automated]"

```

  

### 5.3 Vercel 배포 확인



```text

Vercel 대시보드 → obslog 프로젝트

└─> Deployments 탭

└─> 최신 배포 진행 중

└─> 완료 후 블로그 확인!

```

  

---

  

## 트러블슈팅

  

### ❌ "Workflow not found"

  

**원인:** obslog 레포에 `update-submodule.yml`이 없음

  

**해결:**

```bash

cd /Users/gimdogyeong/projects/obslog

ls -la .github/workflows/update-submodule.yml

  

# 없으면 Step 4 다시 진행

```

  

---

  

### ❌ "Bad credentials"

  

**원인:** PARENT_REPO_TOKEN이 잘못됨

  

**해결:**

1. Token 재발급 (Step 2)

2. obslog-content Secret 재설정

3. Token 권한 확인 (repo, workflow 필요)

  

---

  

### ❌ "Resource not accessible by integration"

  

**원인:** Token 권한 부족

  

**해결:**

```text

GitHub → Settings → Developer settings → Tokens

→ 기존 토큰 Edit → repo, workflow 체크 확인

```

  

---

  

### ❌ Submodule 업데이트는 되는데 Vercel 배포 안 됨

  

**원인:** Vercel이 `[automated]` 커밋을 무시함

  

**해결:**

```yaml

# update-submodule.yml에서 커밋 메시지 변경

git commit -m "Update content submodule"

# [automated] 제거

```

  

---

  

### ❌ "detached HEAD" 경고

  

**원인:** Submodule이 detached HEAD 상태

  

**해결:**

```bash

cd /Users/gimdogyeong/projects/obslog/content

git checkout main

git pull

cd ..

git add content

git commit -m "Fix submodule HEAD"

git push

```

  

---

  

## 추가 최적화

  

### 1. 커밋 메시지에 변경 내용 포함

  

```yaml

# update-submodule.yml

- name: Update submodule

run: |

git submodule update --remote content

  

# 변경된 파일 목록 가져오기

CHANGES=$(git -C content log --oneline -1)

  

git add content

git commit -m "chore: Update content - $CHANGES"

git push

```

  

### 2. Slack 알림 추가

  

```yaml

# notify-parent.yml에 추가

- name: Notify Slack

uses: slackapi/slack-github-action@v1

with:

payload: |

{

"text": "✅ Blog content updated!"

}

env:

SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

```

  

### 3. 실패 시 이메일 알림

  

```yaml

# update-submodule.yml에 추가

on:

repository_dispatch:

workflow_dispatch: # 수동 실행도 가능

  

jobs:

update:

# ... 기존 내용

  

- name: Notify on failure

if: failure()

run: |

echo "::error::Submodule update failed!"

```

  

---

  

## 전체 워크플로우 요약



```text

┌─────────────────────────────────────────────────────────────┐

│ 1. Obsidian에서 글 작성 │

│ └─> posts/new-post.md │

└─────────────────────────────────────────────────────────────┘

↓

┌─────────────────────────────────────────────────────────────┐

│ 2. Obsidian Git Plugin으로 push │

│ └─> obslog-content 레포 │

└─────────────────────────────────────────────────────────────┘

↓

┌─────────────────────────────────────────────────────────────┐

│ 3. 🤖 GitHub Actions: notify-parent.yml │

│ └─> obslog 레포에 신호 전송 │

└─────────────────────────────────────────────────────────────┘

↓

┌─────────────────────────────────────────────────────────────┐

│ 4. 🤖 GitHub Actions: update-submodule.yml │

│ └─> git submodule update --remote │

│ └─> git commit & push │

└─────────────────────────────────────────────────────────────┘

↓

┌─────────────────────────────────────────────────────────────┐

│ 5. Vercel 자동 배포 │

│ └─> obslog 변경 감지 → 빌드 → 배포 │

└─────────────────────────────────────────────────────────────┘

↓

┌─────────────────────────────────────────────────────────────┐

│ 6. 블로그에 새 글 표시! 🎉 │

└─────────────────────────────────────────────────────────────┘

```

  

---

  

## 참고 자료

  

- [GitHub Actions 공식 문서](https://docs.github.com/en/actions)

- [Repository Dispatch 이벤트](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event)

- [Git Submodules 가이드](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

  

---

  

**작성일:** 2026-02-11

**최종 수정:** 2026-02-11