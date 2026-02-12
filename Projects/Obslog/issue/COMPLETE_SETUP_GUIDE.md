# 완벽 개발 환경 구축 가이드 (2026)

> 재사용 가능한 컴포넌트 시스템 + Figma MCP + Obsidian + CodeRabbit 연동

**작성일:** 2026-02-11
**프로젝트:** display-flex, obslog, Obsidian vault.

---

## Obsidian + Git + CodeRabbit

### 초기 목표

Obsidian에서 rabbit branch push → CodeRabbit 자동 리뷰

### 문제 발견

**CodeRabbit은 PR 전용!**
- Main branch push는 리뷰 안 함
- `.coderabbit.yaml`의 `review_on_push: true` 작동 안 함

### 해결: 브랜치 + PR 방식

#### 1. .coderabbit.yaml 설정

```yaml
# /Users/gimdogyeong/Documents/Obsidian/.coderabbit.yaml

language: ko  # 한국어 리뷰
early_access: true
enable_free_tier: true

reviews:
  review_on_push: true
  auto_review: true
  level: "detailed"

  path_filters:
    - "**/*.md"
    - "Projects/**"

  path_ignores:
    - ".obsidian/**"
    - ".trash/**"
    - ".DS_Store"

markdown:
  validate_frontmatter: true
  check_links: true
  spell_check: true
  spell_check_language: "ko-KR"

notifications:
  post_review_comment: true
  summary: true
```

#### 2. .gitignore 업데이트

```gitignore
# .obsidian 설정 파일 무시
.obsidian/cache
.obsidian/workspace
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.DS_Store
Thumbs.db
```

**이유:**
- `workspace.json`은 Obsidian 개인 설정
- Git에 commit할 필요 없음
- Obsidian이 실시간 수정하여 conflict 발생

#### 3. 워크플로우

```bash
# 1. 브랜치 생성
cd ~/Documents/Obsidian
git checkout -b post/new-article

# 2. Obsidian에서 글 작성

# 3. Commit & Push
git add .
git commit -m "feat: Add new article"
git push origin post/new-article

# 4. PR 생성
gh pr create --title "New article" --body "CodeRabbit please review" --fill

# 5. CodeRabbit 리뷰 확인 (1-2분)

# 6. PR 코멘트로 한국어 요청
@coderabbitai 한국어로 리뷰해주세요

# 7. 리뷰 확인 후 Merge
gh pr merge --auto --squash

# 8. main으로 돌아가기
git checkout main
git pull
```

### 한국어 리뷰 설정

**문제:** `.coderabbit.yaml`에 `language: ko` 설정했는데 영어로 리뷰

**해결:** coderabbit 사이트에 가서 직접 언어설정

---

## 트러블슈팅

### 1. Git Conflict: workspace.json

**문제:**
```
error: Your local changes to the following files would be overwritten by checkout:
        .obsidian/workspace.json
```

**원인:**
- Obsidian이 실시간으로 `workspace.json` 수정
- Git이 변경사항 감지

**해결:**
```bash
# 방법 1: 변경사항 버리기
git restore .obsidian/workspace.json

# 방법 2: Stash
git stash

# 방법 3: Obsidian 종료 후 진행
# 1. Obsidian 종료 (Cmd + Q)
# 2. git stash && git pull
# 3. Obsidian 다시 열기
```

**근본 해결:**
```bash
# .gitignore에 추가
echo ".obsidian/workspace.json" >> .gitignore
echo ".obsidian/workspace-mobile.json" >> .gitignore

git add .gitignore
git commit -m "chore: Ignore workspace files"
git push
```

---

### 2. CodeRabbit Main Push 안 됨

**문제:** Obsidian에서 main push했는데 CodeRabbit 리뷰 없음

**원인:** CodeRabbit은 기본적으로 PR 전용

**해결:** 브랜치 + PR 방식 사용 (위 워크플로우 참고)

---

