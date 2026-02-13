---
title: "Obsidian + Blog 연동 구조 가이드"
subject: "[[Obslog]]"
reference: "[[ROADMAP]]"
date: "2026-02-13"
description: "Obsidian vault를 블로그와 연동할 때 시리즈, 태그 구조화 가이드"
tags:
  - obsidian
  - blog
  - structure
series: ""
seriesOrder:
published: false
---

# Obsidian + Blog 연동 구조 가이드

Obsidian vault를 블로그와 연동할 때 **가독성**과 **관리 편의성**을 동시에 달성하는 구조입니다.

---

## 1. 폴더 구조 (3가지 접근법)

### 방법 A: Published/Draft 분리 (추천) ⭐

```bash
Obsidian/
├── 📝 Blog/                    # 블로그 발행용
│   ├── Published/              # 이미 발행된 글
│   │   ├── 2024-01-15-nextjs-guide.md
│   │   ├── 2024-02-01-react-hooks.md
│   │   └── 2024-02-10-typescript-tips.md
│   │
│   └── Drafts/                 # 작성 중인 글
│       ├── wip-frontend-architecture.md
│       └── idea-rust-basics.md
│
├── 📚 Dev Note/                # 개인 학습 노트 (비공개)
│   ├── Component Folder Pattern.md
│   ├── Frontend Architecture.md
│   └── Modern Frontend Terms.md
│
├── 🗂️ Templates/               # 템플릿
│   └── blog-post-template.md
│
└── 💼 Projects/
    └── Obslog/
```

**장점:**
- 발행 상태를 폴더로 명확히 구분
- Obsidian에서 보기 편함
- Draft → Published로 이동만 하면 됨

**frontmatter 예시:**
```markdown
---
title: "Next.js 시작하기"
description: "Next.js 입문 가이드"
tags: [nextjs, react, tutorial]
series: "Next.js 완벽 가이드"
seriesOrder: 1
published: true                    # 폴더와 일치
createdAt: "2024-01-15"
updatedAt: "2024-01-15"
---
```

---

### 방법 B: 시리즈 중심 구조

```bash
Obsidian/
├── 📝 Blog/
│   ├── Series/
│   │   ├── Next.js 완벽 가이드/
│   │   │   ├── 01-nextjs-intro.md
│   │   │   ├── 02-nextjs-routing.md
│   │   │   └── 03-nextjs-data-fetching.md
│   │   │
│   │   ├── React 심화/
│   │   │   ├── 01-react-hooks.md
│   │   │   └── 02-react-context.md
│   │   │
│   │   └── TypeScript 마스터/
│   │       └── 01-typescript-basics.md
│   │
│   └── Standalone/              # 시리즈 없는 단독 글
│       ├── vscode-tips.md
│       └── git-workflow.md
│
└── 📚 Dev Note/
```

**장점:**
- 시리즈별로 관리하기 쉬움
- 연관된 글 찾기 편함
- 파일명에 순서 명시

**frontmatter 예시:**
```markdown
---
title: "Next.js 라우팅 완벽 가이드"
series: "Next.js 완벽 가이드"
seriesOrder: 2                     # 폴더 내 파일명과 일치
tags: [nextjs, routing, app-router]
published: true
---
```

---

### 방법 C: 태그 기반 (Flat 구조)

```bash
Obsidian/
├── 📝 Blog/
│   ├── 2024-01-15-nextjs-guide.md
│   ├── 2024-01-20-react-hooks.md
│   ├── 2024-02-01-typescript-tips.md
│   ├── 2024-02-10-vscode-setup.md
│   └── 2024-02-15-git-workflow.md
│
└── 📚 Dev Note/
```

**장점:**
- 가장 단순함
- 날짜로 정렬 쉬움
- 태그/시리즈는 frontmatter에서만 관리

**단점:**
- 많아지면 찾기 어려움
- 시리즈 파악 어려움

---

## 2. 태그 체계 설계

### 계층적 태그 구조

```yaml
# 1단계: 대분류 (카테고리)
- frontend
- backend
- devops
- career

# 2단계: 기술 스택
- nextjs
- react
- typescript
- nodejs
- docker

# 3단계: 주제/목적
- architecture
- pattern
- tutorial
- troubleshooting
- optimization
```

### 실전 태그 예시

```markdown
---
title: "Next.js App Router 마이그레이션"
tags:
  - frontend              # 대분류
  - nextjs                # 기술
  - react                 # 관련 기술
  - architecture          # 주제
  - tutorial              # 목적
---
```

### 태그 네이밍 규칙

```yaml
✅ Good:
  - nextjs (소문자, 하이픈 없음)
  - app-router (하이픈 사용)
  - typescript

❌ Bad:
  - Next.js (대문자, 특수문자)
  - Next_js (언더스코어)
  - "Next JS" (공백)
```

---

## 3. 시리즈 관리

### 시리즈 네이밍

```markdown
# ✅ 명확한 시리즈명
series: "Next.js 완벽 가이드"
series: "React 심화 시리즈"
series: "TypeScript 마스터 클래스"

# ❌ 애매한 시리즈명
series: "Next.js"          # 너무 광범위
series: "가이드"           # 무엇의 가이드?
series: "Part 1"           # 시리즈명이 아님
```

### seriesOrder 규칙

```markdown
---
series: "Next.js 완벽 가이드"
seriesOrder: 1              # 1부터 시작
---

---
series: "Next.js 완벽 가이드"
seriesOrder: 2
---

---
series: "Next.js 완벽 가이드"
seriesOrder: 3
---
```

### 시리즈 메타데이터 (선택)

별도 파일로 시리즈 정보 관리:

```markdown
# Blog/Series/_metadata/nextjs-guide.md
---
seriesName: "Next.js 완벽 가이드"
description: "Next.js를 처음 시작하는 분들을 위한 완벽 가이드"
totalPosts: 10
status: "진행중"  # 완료, 진행중, 계획중
---

## 시리즈 구성

1. ✅ Next.js 소개
2. ✅ App Router 기본
3. 🚧 데이터 페칭
4. 📝 서버 컴포넌트
5. 📝 인증 구현
...
```

---

## 4. Frontmatter 표준 템플릿

### 최소 필수 필드

```markdown
---
title: "포스트 제목"
description: "포스트 요약 (150자 이내)"
tags: [tag1, tag2, tag3]
published: true
createdAt: "2024-01-15"
---
```

### 완전한 템플릿

```markdown
---
# 필수
title: "Next.js 15 App Router 완벽 가이드"
description: "Next.js 15의 App Router를 실전 예제와 함께 완벽하게 이해하기"
tags: [nextjs, react, app-router, tutorial]
published: true
createdAt: "2024-01-15"

# 시리즈 (선택)
series: "Next.js 완벽 가이드"
seriesOrder: 1

# 추가 정보 (선택)
updatedAt: "2024-01-20"
thumbnail: "/images/nextjs-guide-cover.jpg"
author: "Your Name"
canonical: "https://original-url.com"  # 재발행인 경우

# 커스텀 (선택)
difficulty: "beginner"     # beginner, intermediate, advanced
readingTime: 15            # 분
featured: true             # 메인에 노출
---
```

---

## 5. Obsidian 플러그인 활용

### 추천 플러그인

1. **Templater**
   - 템플릿 자동화
   - frontmatter 자동 생성
   - 날짜, 시간 자동 삽입

2. **Dataview**
   - 태그별 포스트 목록
   - 시리즈별 모아보기
   - Published/Draft 구분

3. **Tag Wrangler**
   - 태그 일괄 변경
   - 태그 자동완성

### Dataview 쿼리 예시

```dataviewjs
// Published 포스트만 보기
TABLE title, tags, createdAt
FROM "Blog/Published"
WHERE published = true
SORT createdAt DESC

// 특정 시리즈 모아보기
TABLE title, seriesOrder
FROM "Blog"
WHERE series = "Next.js 완벽 가이드"
SORT seriesOrder ASC

// 특정 태그 포스트
LIST
FROM #nextjs AND #tutorial
WHERE published = true
```

---

## 6. 블로그와 연동 시 주의사항

### .gitignore 설정

```gitignore
# Obsidian 설정 제외 (개인 설정)
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/cache/

# 개인 노트 제외 (비공개)
Dev Note/
Templates/
_private/

# 허용 (블로그 발행용)
!Blog/Published/
!Blog/Series/
```

### 파일명 규칙

```bash
# ✅ Good: URL 친화적
2024-01-15-nextjs-app-router.md
react-hooks-guide.md
typescript-best-practices.md

# ❌ Bad: URL에 문제 발생
2024/01/15 Next.js 가이드.md     # 공백, 슬래시, 한글
Next.js (App Router).md         # 괄호, 특수문자
```

### Published 필드 활용

```markdown
# Dev Note (개인 노트)
---
published: false    # 블로그에 노출 안 됨
---

# Blog (발행)
---
published: true     # 블로그에 노출
---

# Draft (작성 중)
---
published: false    # 완성되면 true로
---
```

---

## 7. 최종 추천 구조 (실전)

```bash
Obsidian/
├── 📝 Blog/
│   ├── Published/                      # 발행된 글
│   │   ├── Frontend/
│   │   │   ├── 2024-01-15-nextjs-guide.md
│   │   │   └── 2024-02-01-react-hooks.md
│   │   ├── Backend/
│   │   └── DevOps/
│   │
│   ├── Drafts/                         # 작성 중
│   │   └── wip-typescript-advanced.md
│   │
│   └── Series/                         # 시리즈별
│       ├── _metadata/                  # 시리즈 정보
│       ├── Next.js 완벽 가이드/
│       └── React 심화/
│
├── 📚 Dev Note/                        # 개인 노트 (published: false)
│   ├── Component Folder Pattern.md
│   └── Frontend Architecture.md
│
├── 🗂️ Templates/
│   ├── blog-post.md
│   └── series-post.md
│
└── 💼 Projects/
    └── Obslog/
```

**이 구조의 장점:**
1. ✅ Published/Drafts로 상태 명확
2. ✅ Frontend/Backend로 큰 카테고리 분류
3. ✅ Series 폴더로 시리즈 관리
4. ✅ Dev Note는 개인 노트로 분리

---

## 8. Templater 템플릿 예시

### `Templates/blog-post.md`

```markdown
---
title: "<% tp.file.cursor(1) %>"
description: "<% tp.file.cursor(2) %>"
tags: []
series: ""
seriesOrder:
published: false
createdAt: "2026-02-13"
updatedAt: "2026-02-13"
---

# <% tp.file.cursor(1) %>

## 들어가며

<% tp.file.cursor(3) %>

## 본문

## 마치며

---

**관련 글:**
- [[]]

**참고 자료:**
-
```

---

## 9. 워크플로우

```
1. 새 글 작성
   ├── Drafts/ 폴더에서 시작
   └── Templater로 frontmatter 자동 생성

2. 작성 중
   ├── published: false 유지
   └── Obsidian에서 편집

3. 완성
   ├── published: true로 변경
   ├── Drafts/ → Published/ 이동
   └── Git commit & push

4. 블로그 자동 반영
   └── Vercel/Netlify 자동 빌드
```

---

## 10. 체크리스트

블로그 연동 전 확인:

- [ ] 폴더 구조 결정 (A, B, C 중 선택)
- [ ] 태그 체계 정의 (대분류, 기술, 주제)
- [ ] 시리즈 네이밍 규칙 정의
- [ ] Frontmatter 템플릿 작성
- [ ] Templater 설치 & 설정
- [ ] Dataview 설치 (선택)
- [ ] .gitignore 설정
- [ ] Published/Draft 구분 전략
- [ ] 샘플 포스트 3개 작성
- [ ] 블로그 연동 테스트

---

## 결론

**가장 추천하는 구조:**

```
방법 A (Published/Draft 분리) + 태그 계층 + 시리즈 폴더
```

**이유:**
1. ✅ 상태 관리 명확 (발행/작성중 구분)
2. ✅ Obsidian에서 보기 편함
3. ✅ 확장성 좋음 (시리즈 추가 쉬움)
4. ✅ 블로그와 동기화 간단

**시작 팁:**
- 처음엔 단순하게 시작 (Published/Drafts만)
- 글이 많아지면 Frontend/Backend 같은 하위 분류 추가
- 시리즈는 필요할 때 Series 폴더로 분리

이 구조로 시작하면 **가독성**, **관리 편의성**, **확장성**을 모두 달성할 수 있습니다! 🚀
