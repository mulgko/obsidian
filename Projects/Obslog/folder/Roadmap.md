---
tags: [setting]
---

# Obsidian + Git 블로그 — 처음부터 만들기 로드맵

> 이 문서는 **초보 개발자**가 `npm init`부터 시작하여
> 블로그를 **직접 설계하고 구축**할 수 있도록 작성되었습니다.
> 답을 주지 않습니다. **왜 그런지**, **어떤 선택지가 있는지** 알려주고, 당신이 결정합니다.
> 막히면 Claude에게 "Phase X-Y에서 막혔어"라고 물어보세요.

> 💡 **최종 폴더 구조가 궁금하다면?**
> [[Folder pattern]] 문서에서 **ROADMAP + 2026년 최신 패턴**을 결합한 완성형 구조와
> **Co-location**, **Barrel Pattern** 등 핵심 설계 원칙을 확인할 수 있습니다!

---

## 목차

0. **[[Folder pattern]]** — 최종 폴더 구조 & 설계 원칙 (먼저 읽기 추천!) ✨
1. [프로젝트 전체 그림](#1-프로젝트-전체-그림)
2. [Phase 0 - 개발 환경 & 핵심 개념](#phase-0---개발-환경--핵심-개념)
3. [Phase 1 - 프로젝트 생성 & 구조 설계](#phase-1---프로젝트-생성--구조-설계)
4. [Phase 2 - 레이아웃 & 공통 컴포넌트](#phase-2---레이아웃--공통-컴포넌트)
5. [Phase 3 - 마크다운 포스트 시스템](#phase-3---마크다운-포스트-시스템)
6. [Phase 4 - 메인 페이지](#phase-4---메인-페이지) ← 현재 진행 중
7. [Phase 5 - 포스트 상세 페이지](#phase-5---포스트-상세-페이지)
8. [Phase 6 - 시리즈 & 검색 & 소개](#phase-6---시리즈--검색--소개)
9. [Phase 7 - 웹 어드민 (인증 + CRUD)](#phase-7---웹-어드민-인증--crud)
10. [Phase 8 - Obsidian + Git 동기화](#phase-8---obsidian--git-동기화)
11. [Phase 9 - 댓글 기능](#phase-9---댓글-기능)
12. [Phase 10 - 배포 & 실전 운영](#phase-10---배포--실전-운영)
13. [Phase 11 - 고도화 (선택)](#phase-11---고도화-선택)
12. [부록 A - 디자인 패턴 & 설계 원칙](#부록-a---디자인-패턴--설계-원칙)
13. [부록 B - 유용한 레퍼런스](#부록-b---유용한-레퍼런스)

---

## 1. 프로젝트 전체 그림

### 최종 완성 모습

```
┌─────────────────┐         ┌──────────────┐         ┌─────────────────┐
│                 │         │              │         │                 │
│    Obsidian     │──push──▶│    GitHub     │◀──push──│   웹 어드민      │
│  (마크다운 편집)  │         │  (중앙 저장소)  │         │  (브라우저 편집)   │
│                 │         │              │         │                 │
└─────────────────┘         └──────┬───────┘         └─────────────────┘
                                   │
                              자동 빌드
                                   │
                                   ▼
                            ┌──────────────┐
                            │   블로그 웹    │
                            │  (Next.js)   │ ← 댓글 기능 포함
                            └──────────────┘
```

### 핵심 설계 철학

**"마크다운 파일이 진실의 원천(Single Source of Truth)"**

- DB 없음. 마크다운 파일 = 데이터
- 어디서 수정하든 결국 `.md` 파일이 변경됨
- Git이 버전 관리 & 동기화를 담당

### 사용할 기술 스택

| 역할 | 기술 | 왜? |
|------|------|-----|
| 프레임워크 | **Next.js (App Router)** | React 기반, SSG 지원, 풀스택 가능 |
| 언어 | **TypeScript** | 타입 안전성, 실수 방지 |
| 스타일 | **Tailwind CSS** | 빠른 스타일링, 익숙한 도구 |
| 마크다운 | **gray-matter + remark** | frontmatter 파싱 + HTML 변환 |
| 코드 하이라이팅 | **Shiki** | VS Code와 동일한 하이라이팅 |
| 인증 | **NextAuth.js** | Next.js에 최적화된 인증 |
| 댓글 | **Giscus** | GitHub Discussions 기반, 무료 |
| 배포 | **Vercel** | Next.js에 최적화, 자동 빌드 |

---

## Phase 0 - 개발 환경 & 핵심 개념

> **목표:** 코딩 전에 알아야 할 것들을 정리합니다
> **소요:** 1~2일
> **코딩 없음** - 순수 학습

### 0-1. 개발 환경 확인

시작 전에 이것들이 설치되어 있어야 합니다:

```bash
# 버전 확인 명령어
node -v    # Node.js 18 이상 권장
npm -v     # npm (Node.js 설치 시 같이 설치됨)
git -v     # Git
```

설치 안 되어 있다면:
- **Node.js:** [nodejs.org](https://nodejs.org)에서 LTS 버전 설치
- **Git:** [git-scm.com](https://git-scm.com)에서 설치

**에디터:** VS Code 권장 (확장 프로그램: ESLint, Tailwind CSS IntelliSense, Prettier)

### 0-2. Next.js App Router 이해하기

Next.js에는 두 가지 라우팅 방식이 있습니다:

| Pages Router (옛날 방식) | App Router (현재 방식) |
|-------------------------|----------------------|
| `pages/` 폴더 사용 | `app/` 폴더 사용 |
| 모든 컴포넌트가 클라이언트 | 기본이 서버 컴포넌트 |
| `getStaticProps` 사용 | 컴포넌트에서 직접 fetch |

**우리는 App Router를 사용합니다.** 최신 방식이고, 서버 컴포넌트가 기본이라 성능이 좋습니다.

**핵심 개념: 서버 컴포넌트 vs 클라이언트 컴포넌트**

```
서버 컴포넌트 (기본값):
- 서버에서 실행됨
- 파일 읽기(fs), DB 접근 가능
- useState, useEffect 사용 불가
- 번들 크기에 포함 안 됨 (가벼움)

클라이언트 컴포넌트 ("use client" 선언):
- 브라우저에서 실행됨
- 파일 읽기 불가
- useState, useEffect, onClick 등 사용 가능
- 번들에 포함됨
```

**규칙:** 가능하면 서버 컴포넌트로, 상호작용이 필요할 때만 클라이언트 컴포넌트로.

**학습할 것:**
- 검색: "Next.js App Router tutorial"
- 검색: "React Server Components 이해하기"

### 0-3. 파일 기반 라우팅

Next.js App Router에서는 **폴더 구조 = URL 구조**입니다:

```
app/
├── page.tsx              → /
├── about/
│   └── page.tsx          → /about
├── posts/
│   ├── page.tsx          → /posts
│   └── [slug]/
│       └── page.tsx      → /posts/hello-world (동적 라우팅)
└── admin/
    └── page.tsx          → /admin
```

**특수 파일들:**
- `page.tsx` → 해당 경로의 페이지 (이것만 URL로 접근 가능)
- `layout.tsx` → 레이아웃 (자식 페이지를 감쌈)
- `loading.tsx` → 로딩 UI
- `error.tsx` → 에러 UI
- `not-found.tsx` → 404 UI

### 0-4. Frontmatter란?

마크다운 파일 맨 위에 `---`로 감싸는 메타 정보입니다:

```markdown
---
title: "Next.js 시작하기"
description: "Next.js를 처음 시작하는 분들을 위한 가이드"
tags: ["Next.js", "React"]
published: true
createdAt: "2024-01-15"
---

# 실제 포스트 내용은 여기부터
```

DB의 컬럼 역할을 파일 안에서 합니다. `gray-matter`라는 패키지가 이걸 파싱해줍니다.

### 0-5. 렌더링 방식 이해하기

Next.js는 여러 렌더링 방식을 지원합니다. 블로그에 중요한 것들:

```
SSG (Static Site Generation) - 정적 생성:
  빌드 시점에 HTML을 미리 만들어놓음
  → 블로그 포스트에 최적! 내용이 자주 안 바뀌니까
  → 로딩 매우 빠름, SEO 완벽

SSR (Server-Side Rendering) - 서버 렌더링:
  요청할 때마다 서버에서 HTML을 생성
  → 실시간 데이터가 필요할 때 사용

CSR (Client-Side Rendering) - 클라이언트 렌더링:
  브라우저에서 JavaScript로 렌더링
  → 인터랙션이 많은 UI (어드민, 에디터 등)
```

**우리 블로그:** 공개 페이지 = SSG, 어드민 페이지 = CSR/SSR

### 0-6. Phase 0 체크리스트

- [x] VS Code + 확장 프로그램 설정
- [x] App Router의 파일 기반 라우팅 이해
- [x] 서버 컴포넌트 vs 클라이언트 컴포넌트 차이 이해
- [x] Frontmatter 개념 이해
- [x] SSG / SSR / CSR 차이 이해
- [x] TypeScript 기본 문법 (interface, type, 제네릭 정도)

---

## Phase 1 - 프로젝트 생성 & 구조 설계

> **목표:** Next.js 프로젝트를 만들고, 폴더 구조를 직접 설계합니다
> **소요:** 1~2일
> **배우는 것:** create-next-app, 프로젝트 구조, 설계 사고방식

### 1-1. 프로젝트 생성

```bash
npx create-next-app@latest my-blog
```

설치 중 물어보는 질문들:

```
✔ Would you like to use TypeScript?            → Yes
✔ Would you like to use ESLint?                → Yes
✔ Would you like to use Tailwind CSS?          → Yes
✔ Would you like your code inside a `src/` directory? → Yes
✔ Would you like to use App Router?            → Yes
✔ Would you like to use Turbopack?             → Yes
✔ Would you like to customize the import alias? → No (기본값 @/* 사용)
```

**각 선택의 의미:**
- `TypeScript`: 타입이 있어서 실수를 줄여줌
- `ESLint`: 코드 품질 검사 도구
- `Tailwind CSS`: 유틸리티 CSS 프레임워크
- `src/`: 소스 코드를 src 폴더에 모아서 깔끔하게
- `App Router`: 최신 라우팅 방식
- `Turbopack`: 빠른 개발 서버 (Webpack 대체)

### 1-2. 생성된 프로젝트 살펴보기

```bash
cd my-blog
npm run dev
```

브라우저에서 `http://localhost:3000` 열어보세요. Next.js 기본 페이지가 보입니다.

**생성된 파일들을 하나씩 열어보세요:**

```
my-blog/
├── src/
│   └── app/
│       ├── layout.tsx      ← 전체 앱을 감싸는 최상위 레이아웃
│       ├── page.tsx        ← 메인 페이지 (/)
│       ├── globals.css     ← 전역 스타일
│       └── favicon.ico
├── public/                 ← 정적 파일 (이미지 등)
├── package.json            ← 프로젝트 설정 & 의존성 목록
├── tsconfig.json           ← TypeScript 설정
├── tailwind.config.ts      ← Tailwind 설정
├── next.config.ts          ← Next.js 설정
└── eslint.config.mjs       ← ESLint 설정
```

**해볼 것:**
1. `src/app/page.tsx`를 열어서 내용을 전부 지우고 `<h1>내 블로그</h1>`만 남겨보세요
2. 브라우저에서 바로 반영되는지 확인 (Hot Reload)
3. `src/app/layout.tsx`를 열어서 구조를 파악하세요

### 1-3. 폴더 구조 직접 설계하기

여기서부터가 진짜입니다. **스스로 구조를 설계해보세요.**

> 💡 **최종 추천 구조가 궁금하다면?**
> [[Folder pattern]] 문서에서 **ROADMAP + 2026년 최신 패턴**을 결합한 완성형 구조를 확인할 수 있습니다.
> 하지만 먼저 스스로 설계해보고, 나중에 비교해보는 것을 추천합니다!

블로그에 필요한 것들을 나열해봅시다:

```
페이지:
  - / (메인 - 포스트 목록)
  - /posts/[slug] (포스트 상세)
  - /tags (태그 목록)
  - /tags/[tag] (태그별 포스트)
  - /series (시리즈 목록)
  - /series/[slug] (시리즈별 포스트)
  - /about (소개)
  - /admin (어드민 대시보드)
  - /admin/posts (포스트 관리)
  - /admin/posts/new (포스트 작성)
  - /admin/posts/[slug]/edit (포스트 수정)
  - /admin/login (로그인)

컴포넌트:
  - Header (네비게이션)
  - Footer
  - PostCard (포스트 미리보기 카드)
  - PostContent (마크다운 렌더러)
  - ThemeToggle (다크 모드)
  - 어드민용 컴포넌트들

유틸리티:
  - 마크다운 파일 읽기/쓰기 함수
  - 인증 관련 함수
  - 유틸리티 함수 (날짜 포맷, slug 변환 등)

콘텐츠:
  - 마크다운 포스트 파일들
```

**이걸 폴더 구조로 어떻게 표현할지 직접 고민해보세요.**

아래는 힌트입니다. 바로 보지 말고 먼저 시도해보세요!

> 📌 **2026년 최신 패턴 적용**
> 아래 힌트는 기본 구조입니다. [[Folder pattern]]에서는 **Co-location**, **Barrel Pattern** 등
> 최신 설계 패턴이 적용된 개선된 구조를 확인할 수 있습니다.

<details>
<summary>힌트: 권장 폴더 구조 (클릭해서 펼치기)</summary>

```
my-blog/
├── content/                      ← 마크다운 콘텐츠 (Git으로 관리)
│   └── posts/
│       ├── hello-world.md
│       └── my-first-post.md
│
├── public/                       ← 정적 파일
│   └── images/
│
├── src/
│   ├── app/                      ← 페이지 (라우팅)
│   │   ├── (blog)/               ← 공개 블로그 (Route Group)
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx          → /
│   │   │   ├── posts/[slug]/
│   │   │   │   ├── page.tsx      → /posts/[slug]
│   │   │   │   └── _components/  ← ✨ Co-location
│   │   │   ├── tags/
│   │   │   │   ├── page.tsx      → /tags
│   │   │   │   └── [tag]/
│   │   │   │       └── page.tsx  → /tags/[tag]
│   │   │   ├── series/
│   │   │   │   ├── page.tsx      → /series
│   │   │   │   └── [slug]/
│   │   │   │       └── page.tsx  → /series/[slug]
│   │   │   └── about/
│   │   │       └── page.tsx      → /about
│   │   │
│   │   ├── (admin)/              ← 어드민 (Route Group)
│   │   │   ├── layout.tsx
│   │   │   └── admin/
│   │   │       ├── page.tsx      → /admin
│   │   │       ├── posts/
│   │   │       │   ├── page.tsx
│   │   │       │   ├── new/
│   │   │       │   │   ├── page.tsx
│   │   │       │   │   └── _components/  ← ✨ Co-location
│   │   │       │   └── [slug]/
│   │   │       │       └── edit/
│   │   │       │           └── page.tsx
│   │   │       └── login/
│   │   │           └── page.tsx
│   │   │
│   │   ├── api/                  ← API 라우트
│   │   │   └── ...
│   │   │
│   │   ├── layout.tsx            ← 루트 레이아웃
│   │   └── globals.css
│   │
│   ├── components/               ← 재사용 가능한 컴포넌트
│   │   ├── ui/                   ← ✨ UI 라이브러리 (Barrel Pattern)
│   │   │   ├── Button/
│   │   │   │   ├── index.ts      ← export { Button }
│   │   │   │   └── Button.tsx
│   │   │   └── Input/
│   │   └── common/               ← 공통 (Header, Footer)
│   │
│   ├── lib/                      ← 유틸리티 & 비즈니스 로직
│   │   ├── posts.ts              ← 마크다운 CRUD
│   │   ├── markdown.ts           ← MD → HTML 변환
│   │   ├── auth.ts               ← 인증
│   │   ├── github.ts             ← GitHub API (Vercel용)
│   │   └── utils.ts              ← 공통 유틸리티
│   │
│   └── types/                    ← TypeScript 타입 정의
│       └── index.ts
│
├── package.json
├── tsconfig.json
└── next.config.ts
```

**💡 최신 개선 버전:**
위 구조는 기본형입니다. [[Folder pattern]]에서는:
- ✨ `_components` 폴더로 **Co-location** 적용
- ✨ `ui/` vs `common/` 명확한 분류
- ✨ **Barrel Pattern** 선택적 사용 (UI만)
- ✨ `lib/` 파일 세분화 (posts.ts, markdown.ts 분리)

자세한 설계 원칙과 가이드는 [[Folder pattern]]를 참고하세요!

</details>

### 1-4. 폴더 구조 핵심 개념

#### Route Group: `(blog)`, `(admin)`

괄호로 감싼 폴더는 URL에 포함되지 않습니다:

```
src/app/(blog)/page.tsx    → URL: /       (blog가 URL에 안 나옴)
src/app/(admin)/admin/     → URL: /admin  (admin만 나옴)
```

**왜 쓰는가?**
- 블로그 페이지와 어드민 페이지에 **다른 레이아웃**을 적용하기 위해
- (blog)에는 Header + Footer가 있는 레이아웃
- (admin)에는 Sidebar가 있는 레이아웃
- URL에는 영향 없음

#### Co-location: `_components` 폴더

**"변하는 것들은 함께 둔다"** - 페이지 전용 컴포넌트를 같은 폴더에!

```
app/(blog)/posts/[slug]/
├── page.tsx
└── _components/           ← 이 페이지에서만 쓰는 컴포넌트
    ├── PostHeader.tsx
    ├── TableOfContents.tsx
    └── Comments.tsx
```

**장점:**
- 포스트 수정 시 관련 파일이 가까이 있음
- 삭제 용이 (폴더째 제거)
- `_` prefix로 Next.js 라우팅에서 제외

**언제 사용?**
- Phase 2-4에서 페이지별 컴포넌트 만들 때 적극 활용
- 자세한 내용: [[Folder pattern]] - "Co-location" 섹션

#### Barrel Pattern: UI 라이브러리에만 선택적 사용

```typescript
// components/ui/Button/
├── index.ts      ← export { Button } from './Button'
└── Button.tsx

// 사용 시
import { Button, Input } from '@/components/ui/button';  // ✅ 깔끔!
```

**주의:** `common/`에는 Barrel 사용 안 함 (순환 참조 방지)

자세한 내용: [[Folder pattern]] - "Barrel Pattern" 섹션

---

### 1-5. 추가 패키지 설치

프로젝트에서 필요한 패키지들을 설치합니다:

```bash
# 마크다운 처리
npm install gray-matter remark remark-html

# 코드 하이라이팅
npm install shiki

# 인증 (Phase 5에서 사용하지만 미리 설치해도 됨)
npm install next-auth@beta

# 유틸리티
npm install clsx

# Tailwind 타이포그래피 (마크다운 렌더링 스타일)
npm install @tailwindcss/typography
```

**각 패키지의 역할:**
| 패키지 | 역할 |
|--------|------|
| `gray-matter` | 마크다운 파일에서 frontmatter를 파싱 |
| `remark` + `remark-html` | 마크다운 문자열을 HTML로 변환 |
| `shiki` | 코드 블록에 문법 하이라이팅 적용 |
| `next-auth` | 로그인/인증 처리 |
| `clsx` | 조건부 className 합치기 (Tailwind와 잘 어울림) |
| `@tailwindcss/typography` | `prose` 클래스로 마크다운 HTML에 예쁜 스타일 적용 |

**설치 후 `package.json`을 열어서 dependencies에 추가된 것을 확인하세요.**

### 1-6. Git 초기화

```bash
git init
git add .
git commit -m "Initial commit: Next.js project setup"
```

**습관:** 각 Phase가 끝날 때마다 커밋하세요. 문제가 생기면 돌아갈 수 있습니다.

### 1-7. Phase 1 체크리스트

- [x] `create-next-app`으로 프로젝트 생성
- [x] `npm run dev`로 기본 페이지 확인
- [x] 생성된 파일 하나씩 열어서 역할 파악
- [x] 폴더 구조를 **직접 설계**해봄 (종이에 그려봐도 좋음)
- [x] Route Group `(blog)`, `(admin)` 개념 이해
- [x] **Co-location**, **Barrel Pattern** 개념 이해
- [x] [[Folder pattern]] 문서 읽고 최종 구조 파악 ✨
- [x] 필요한 패키지 설치
- [x] Git 초기 커밋

---

## Phase 2 - 레이아웃 & 공통 컴포넌트

> **목표:** 페이지의 뼈대(레이아웃)와 재사용 컴포넌트를 만듭니다
> **소요:** 2~3일
> **배우는 것:** 레이아웃 시스템, 컴포넌트 분리, Tailwind 실전 사용

### 2-1. 레이아웃 구조 이해

```
루트 레이아웃 (src/app/layout.tsx)
├── (blog) 레이아웃 (Header + Footer)
│   ├── / (메인 페이지)
│   ├── /posts/[slug]
│   ├── /tags
│   └── /about
│
└── (admin) 레이아웃 (Sidebar + 인증 체크)
    ├── /admin
    ├── /admin/posts
    └── /admin/posts/new
```

**레이아웃은 중첩(nesting)됩니다:**
1. 루트 레이아웃: `<html>`, `<body>`, 폰트, 전역 스타일
2. 블로그 레이아웃: Header + main + Footer
3. 어드민 레이아웃: Sidebar + main + 인증 체크

### 2-2. 루트 레이아웃 만들기

`src/app/layout.tsx` — 모든 페이지의 최상위 레이아웃입니다.

**직접 만들어보세요!** 포함할 것:
- `<html>`, `<body>` 태그
- 한국어 설정 (`lang="ko"`)
- 전역 CSS 임포트
- 폰트 설정 (next/font 사용)
- `{children}` — 자식 페이지가 여기에 렌더링됨

```typescript
// src/app/layout.tsx

// 힌트: next/font에서 한글 폰트를 가져올 수 있습니다
// import { Noto_Sans_KR } from "next/font/google";

// 힌트: metadata를 export하면 SEO 설정 가능
// export const metadata = { title: "...", description: "..." };
```

**검색 키워드:** "next.js app router root layout", "next/font google"

### 2-3. 블로그 레이아웃 만들기

`src/app/(blog)/layout.tsx` — Header와 Footer를 포함하는 레이아웃

```typescript
// 이런 구조를 목표로:
// <Header />
// <main>{children}</main>
// <Footer />
```

### 2-4. Header 컴포넌트

`src/components/common/Header.tsx`

> 💡 **컴포넌트 위치 가이드**
> Header는 **여러 페이지에서 공통으로** 사용하므로 `components/common/`에 둡니다.
> 만약 특정 페이지에서만 쓴다면 `_components/` 폴더에 두면 됩니다. ([[Folder pattern]] 참고)

**포함할 기능:**
- 블로그 이름/로고
- 네비게이션 링크 (홈, 태그, 시리즈, 소개)
- 다크 모드 토글 (나중에 추가해도 됨)

**직접 만들어보세요!** 고민할 것:
- Next.js에서 페이지 이동은 `<a>` 대신 뭘 쓸까? → `Link` 컴포넌트
- 현재 페이지를 어떻게 표시할까? (네비게이션 활성 상태)
- 모바일 반응형은 어떻게?

```typescript
// 힌트:
// import Link from "next/link";
// import { usePathname } from "next/navigation"; // 현재 경로 확인
```

**검색 키워드:** "next.js Link component", "tailwind responsive navbar"

### 2-5. Footer 컴포넌트

`src/components/common/Footer.tsx`

간단하게: 저작권, GitHub 링크 등

### 2-6. 다크 모드 구현 (선택, 나중에 해도 됨)

다크 모드 구현 방법:

```
방법 1: Tailwind의 dark: 클래스 사용 (간단)
방법 2: next-themes 패키지 사용 (체계적)
```

**방법 2 권장:**
```bash
npm install next-themes
```

**동작 원리:**
1. `<html>` 태그에 `class="dark"` 또는 `class="light"`를 추가/제거
2. Tailwind의 `dark:` 접두사가 이에 반응
3. `next-themes`가 토글 로직 + localStorage 저장을 처리

**검색 키워드:** "next-themes next.js dark mode tailwind"

### 2-7. Phase 2 체크리스트

- [x] 루트 레이아웃 (`src/app/layout.tsx`) 작성
- [x] 한글 폰트 설정
- [x] metadata (title, description) 설정
- [x] 블로그 레이아웃 (`src/app/(blog)/layout.tsx`) 작성
- [x] Header 컴포넌트 작성 (`components/common/` 위치)
- [x] Footer 컴포넌트 작성 (`components/common/` 위치)
- [x] **컴포넌트 위치 선택 기준** 이해 ([[Folder pattern]] 참고) ✨
- [x] (선택) 다크 모드 토글
- [x] 반응형 확인 (모바일/데스크톱)
- [x] `npm run dev`로 레이아웃 확인
- [x] Git 커밋

---

## Phase 3 - 마크다운 포스트 시스템

> **목표:** `.md` 파일을 읽어서 데이터로 변환하는 핵심 시스템을 만듭니다
> **소요:** 3~5일
> **배우는 것:** 파일 시스템(fs), frontmatter 파싱, 데이터 모델링
>
> **이 Phase가 프로젝트의 핵심입니다. 천천히, 꼼꼼하게 하세요.**

### 3-1. 콘텐츠 폴더 & 샘플 파일

프로젝트 루트에 `content/posts/` 폴더를 만들고 샘플 파일을 작성하세요.

```
content/
└── posts/
    ├── hello-world.md
    ├── react-hooks-guide.md
    └── nextjs-routing.md
```

**각 파일에 frontmatter를 포함하세요:**

```markdown
---
title: "제목"
description: "설명"
tags: ["태그1", "태그2"]
series: "시리즈 이름"       # 선택
seriesOrder: 1              # 선택
published: true
createdAt: "2024-01-15"
updatedAt: "2024-01-15"
thumbnail: ""               # 선택
---

# 포스트 내용
```

**직접 해보세요:** 최소 3개의 샘플 포스트를 만들어보세요.
- 태그가 겹치는 것도 있고
- 시리즈에 속한 것도 있게
- published: false인 것도 하나 만들어보세요

### 3-2. 타입 정의부터 하기

코딩 전에 **데이터의 형태를 먼저 정의**합니다. 이게 좋은 습관입니다.

`src/types/index.ts`:

```typescript
/**
 * 마크다운 파일의 frontmatter 타입
 * 이 타입이 content/posts/*.md 파일의 --- 안 내용과 일치해야 합니다
 */
export interface PostFrontmatter {
  // 직접 채워보세요!
  // 위의 frontmatter 예시를 보고 타입을 정의하세요
  // 선택적 필드는 ? 를 붙입니다 (예: thumbnail?: string)
}

/**
 * 포스트 목록에서 사용 (본문 없이 메타데이터만)
 */
export interface PostMeta {
  slug: string;
  frontmatter: PostFrontmatter;
}

/**
 * 포스트 상세에서 사용 (본문 포함)
 */
export interface Post extends PostMeta {
  content: string;
}
```

### 3-3. 핵심 유틸리티: `src/lib/posts.ts`

이 파일이 **블로그의 엔진**입니다. 직접 하나씩 구현해보세요.

```typescript
import fs from "fs";
import path from "path";
import matter from "gray-matter";

const postsDirectory = path.join(process.cwd(), "content", "posts");
```

**구현할 함수 목록:**

#### 함수 1: `getAllPosts()`

```
입력: 없음
출력: PostMeta[] (모든 포스트의 메타데이터, 날짜 내림차순)

동작:
1. content/posts/ 폴더의 모든 .md 파일 읽기
2. 각 파일의 frontmatter 파싱
3. slug = 파일명에서 .md 제거
4. 날짜 내림차순 정렬
5. 반환
```

**힌트:**
- `fs.readdirSync(dir)` → 파일명 배열
- `fs.readFileSync(path, 'utf-8')` → 파일 내용
- `matter(content)` → `{ data, content }`
- `Array.sort()` → 정렬
- `fileName.replace(/\.md$/, '')` → 확장자 제거

#### 함수 2: `getPostBySlug(slug)`

```
입력: slug (string)
출력: Post (메타데이터 + 본문 HTML)

동작:
1. content/posts/{slug}.md 파일 읽기
2. frontmatter + 본문 분리
3. 본문 마크다운 → HTML 변환
4. 반환
```

**힌트:**
- `path.join(postsDirectory, `${slug}.md`)`
- 마크다운 → HTML: `remark().use(remarkHtml).process(markdown)`
- 파일이 없으면? → 에러 처리 or null 반환

#### 함수 3: `getAllTags()`

```
입력: 없음
출력: { name: string, count: number }[]

동작:
1. 모든 포스트의 tags 배열을 모음
2. 중복 제거 + 개수 계산
```

#### 함수 4: `getPostsByTag(tag)`

```
입력: tag (string)
출력: PostMeta[]

동작:
1. 모든 포스트 중 해당 tag가 있는 것만 필터링
```

#### 함수 5: `getAllSeries()`와 `getPostsBySeries(series)`

시리즈 관련 함수도 비슷한 패턴으로 만들어보세요.

### 3-4. 마크다운 → HTML 변환

remark를 사용해서 마크다운을 HTML로 변환합니다.

```typescript
import { remark } from "remark";
import remarkHtml from "remark-html";

async function markdownToHtml(markdown: string): Promise<string> {
  // remark()
  //   .use(remarkHtml)
  //   .process(markdown)
  //   → 결과에서 .toString()하면 HTML 문자열
}
```

**이건 async 함수입니다** (.process()가 Promise를 반환).

**나중에 추가할 것:**
- 코드 하이라이팅 (Shiki) — Phase 4에서
- 목차(TOC) 추출 — Phase 4에서

### 3-5. 테스트해보기

함수를 다 만들었으면 간단하게 테스트하세요.

**방법: 임시 페이지에서 확인**

```typescript
// src/app/(blog)/page.tsx

import { getAllPosts } from "@/lib/posts";

export default function HomePage() {
  const posts = getAllPosts();

  return (
    <div>
      <h1>포스트 {posts.length}개</h1>
      {posts.map((post) => (
        <div key={post.slug}>
          <h2>{post.frontmatter.title}</h2>
          <p>{post.frontmatter.description}</p>
          <p>태그: {post.frontmatter.tags.join(", ")}</p>
        </div>
      ))}
    </div>
  );
}
```

**확인할 것:**
- 포스트가 잘 나오는가?
- 날짜순으로 정렬되었는가?
- frontmatter가 잘 파싱되었는가?
- published: false인 포스트는 어떻게 처리했는가?

### 3-6. 자주 하는 실수

| 문제 | 원인 | 해결 |
|------|------|------|
| `fs is not defined` | 클라이언트 컴포넌트에서 fs 사용 | fs는 서버 컴포넌트에서만 사용 가능 |
| frontmatter 파싱 안됨 | `---` 구분자 오타, 들여쓰기 문제 | YAML 문법 확인 |
| 파일을 못 찾음 | 경로 문제 | `console.log(postsDirectory)`로 경로 확인 |
| 한글 파일명 | URL 인코딩 문제 | 파일명은 영문 slug, 제목은 frontmatter에 한글 |

### 3-7. Phase 3 체크리스트

- [x] `content/posts/` 폴더에 샘플 .md 파일 3개 이상 작성
- [x] `src/types/index.ts`에 타입 정의
- [x] `getAllPosts()` 구현 & 테스트
- [x] `getPostBySlug()` 구현 & 테스트
- [x] `getAllTags()` 구현 & 테스트
- [x] `getPostsByTag()` 구현 & 테스트
- [x] 시리즈 관련 함수 구현 & 테스트
- [ ] `markdownToHtml()` 구현 & 테스트 <!-- WIP: Intentional intermediate commit — implementation pending -->
- [x] 임시 페이지에서 데이터가 잘 나오는지 확인
- [x] Git 커밋

---

## Phase 4 - 블로그 페이지 구현

> **목표:** 실제 블로그 페이지들을 만듭니다 (메인, 포스트 상세, 시리즈, 소개)
> **소요:** 4~6일
> **배우는 것:** 동적 라우팅, 정적 생성, 컴포넌트 설계, SEO, URL 쿼리스트링

### 4-1. 메인 페이지 (`/`)

`src/app/(blog)/page.tsx`

**레이아웃 구조 (2컬럼):**

```
┌────────────────────────────────┬─────────────────┐
│  모든 글 모아보기                │  Tag            │
│                                │  ☐ Frontend     │
│  ┌──────────────┬──────────┐   │  ☑ AI           │
│  │ 태그 제목    │ [thumb]  │   │  ☐ Server       │
│  │ 설명...      │          │   │  ☐ Next         │
│  └──────────────┴──────────┘   │                 │
│  ┌──────────────┬──────────┐   │  Comments       │
│  │ ...          │ [thumb]  │   │  유저명          │
│  └──────────────┴──────────┘   │  댓글 내용...    │
│                                │  → 포스트 링크   │
│  ← 1 2 3 4 5 →                 │                 │
└────────────────────────────────┴─────────────────┘
```

**표시할 것:**
- 포스트 목록 (PostCard 컴포넌트, published: true인 것만)
- 페이지네이션
- 사이드바: 태그 필터 체크박스 + 최근 댓글 (댓글은 Phase 7 전까지 숨김 처리)

**만들 컴포넌트:** `PostCard` — 포스트 미리보기 카드

> 💡 **컴포넌트 위치**
> PostCard는 메인 페이지뿐 아니라 시리즈 페이지에서도 재사용하므로
> `components/common/PostCard.tsx`에 두는 것을 추천합니다.

```
PostCard 구조 (썸네일 항상 포함):
┌────────────────────────────────────────────────────┐
│  Frontend  Next                   ┌──────────────┐ │
│  제목                              │  썸네일 이미지 │ │
│  설명 텍스트 (2줄까지 말줄임처리)    │  (없으면      │ │
│                                   │  placeholder) │ │
└────────────────────────────────────────────────────┘
```

**썸네일 처리 전략:**
- frontmatter에 `thumbnail` 필드 **사용하지 않음**
- 대신 본문에서 첫 번째 이미지를 자동 추출 → 없으면 회색 placeholder
- `lib/posts.ts`에서 파싱 시점에 추출 → `PostMeta`에 포함시켜 전달

```typescript
// lib/posts.ts — 본문 첫 이미지 추출
function extractFirstImage(content: string): string | null {
  // 마크다운 이미지 문법: ![alt](url)
  const match = content.match(/!\[.*?\]\((.*?)\)/);
  return match ? match[1] : null;
}

// getAllPosts()에서 호출
return {
  slug,
  frontmatter: data as PostFrontmatter,
  thumbnail: extractFirstImage(content), // ← frontmatter 밖에서 별도 관리
};
```

```typescript
// types/index.ts — PostMeta 타입 업데이트
export interface PostMeta {
  slug: string;
  frontmatter: PostFrontmatter;
  thumbnail: string | null; // 본문 첫 이미지 (없으면 null)
}
```

```typescript
// PostCard — 썸네일 렌더링
{post.thumbnail ? (
  <Image src={post.thumbnail} alt={post.frontmatter.title} fill className="object-cover" />
) : (
  <div className="w-full h-full bg-gray-200 dark:bg-gray-700" />
)}
```

**주의사항:**
- 이미지가 외부 URL(`https://...`)이면 `next.config.ts`에 `images.remotePatterns` 등록 필요
- Obsidian 내부 링크 `![[image.png]]` 문법은 위 정규식으로 안 잡힘 → `![](./path)` 형식 사용 권장
- Templater 템플릿에서 `thumbnail` frontmatter 필드 제거해도 됨

**태그 필터 (사이드바):**
- URL 쿼리스트링 방식: `/` → `/?tag=AI` → `/?tag=AI,Next` (다중 선택, 쉼표 구분)
- 체크박스 선택 시 URL 업데이트 → 서버에서 필터링
- `useSearchParams()` + `useRouter()` 사용 (클라이언트 컴포넌트)

```typescript
// 사이드바 TagFilter 힌트:
"use client";
import { useRouter, useSearchParams } from "next/navigation";

// 체크박스 onChange → router.push(`/?tag=${tags.join(",")}`)
// 현재 선택된 태그는 searchParams.get("tag")?.split(",").filter(Boolean) || [] 로 읽기
```

**페이지네이션:**
- URL 방식: `/?page=2`, `/?tag=AI&page=3`
- 한 페이지에 보여줄 포스트 수 결정 (예: 10개)
- `getAllPosts()`에서 slice로 페이지별 데이터 분리

```typescript
// lib/posts.ts에 추가할 함수:
// getPaginatedPosts(page: number, tag?: string[], pageSize = 10)
// → { posts: PostMeta[], totalPages: number, currentPage: number }
```

### 4-2. Phase 4 체크리스트

- [x] `extractFirstImage()` 함수 구현 (`lib/posts.ts`) — 본문 첫 이미지 자동 추출
- [x] `PostMeta` 타입에 `thumbnail: string | null` 추가 (`types/index.ts`)
- [x] `getPaginatedPosts()` 함수 구현 (`lib/posts.ts`)
- [x] PostCard 컴포넌트 구현 (`components/common/PostCard.tsx`)
  - [x] 썸네일 있을 때 — `<Image>` 컴포넌트 (외부 URL이면 `next.config.ts` 설정)
  - [x] 썸네일 없을 때 — placeholder div (`bg-gray-200`)
- [x] 메인 페이지 2컬럼 레이아웃 구현 (포스트 목록 + 사이드바)
- [x] 태그 필터 사이드바 (`TagFilter` 컴포넌트, 체크박스 + URL 쿼리스트링)
- [x] 페이지네이션 컴포넌트 구현
- [x] 최근 댓글 사이드바 UI (Phase 9 전까지 숨김 처리)
- [ ] 반응형 확인 (모바일/데스크톱)
- [ ] **컴포넌트 위치가 적절한지 확인** ([[Folder pattern]] 가이드) ✨
- [ ] Git 커밋

---

## Phase 5 - 포스트 상세 페이지

> **목표:** 개별 포스트를 렌더링하는 상세 페이지를 만듭니다
> **소요:** 2~3일
> **배우는 것:** 동적 라우팅, 마크다운 렌더링, 코드 하이라이팅, SEO

### 5-1. 포스트 상세 페이지 (`/posts/[slug]`)

`src/app/(blog)/posts/[slug]/page.tsx`

**동적 라우팅:** `[slug]`는 URL의 변하는 부분입니다.
- `/posts/hello-world` → slug = "hello-world"
- `/posts/react-guide` → slug = "react-guide"

```typescript
// 페이지 컴포넌트는 params를 받습니다
export default async function PostPage({
  params,
}: {
  params: { slug: string };
}) {
  const post = await getPostBySlug(params.slug);

  // post가 없으면? → 404 페이지
  // 힌트: import { notFound } from "next/navigation";
}
```

**표시할 것:**
- 제목, 작성일, 태그
- 본문 (HTML로 변환된 마크다운)
- 목차 (Table of Contents) — 선택
- 시리즈 네비게이션 — 선택
- 댓글 — Phase 9에서

**만들 컴포넌트:**

> 💡 **Co-location 활용하기!**
> 포스트 상세 페이지 전용 컴포넌트들은 `app/(blog)/posts/[slug]/_components/`에 두세요.
>
> ```
> app/(blog)/posts/[slug]/
> ├── page.tsx
> └── _components/          ← ✨ 이 페이지 전용!
>     ├── PostHeader.tsx    (제목, 날짜, 태그)
>     ├── PostContent.tsx   (마크다운 렌더링)
>     ├── TableOfContents.tsx
>     └── Comments.tsx
> ```
>
> **왜?** 관련 파일이 가까이 있어 수정하기 편하고, 삭제 시 폴더째 제거 가능!
>
> 자세한 내용: [[Folder pattern]] - "Co-location" 섹션

`PostContent` — 마크다운 HTML을 예쁘게 렌더링

```typescript
// PostContent 힌트:
// Tailwind의 @tailwindcss/typography를 사용하면
// prose 클래스로 마크다운 HTML에 예쁜 스타일이 적용됩니다

// <article className="prose dark:prose-invert">
//   <div dangerouslySetInnerHTML={{ __html: htmlContent }} />
// </article>
```

**검색 키워드:** "tailwind typography prose", "next.js dynamic routes app router"

#### 코드 하이라이팅 (Shiki)

```typescript
// 방법: remark로 HTML 변환 후, Shiki로 코드 블록만 변환
// 또는: rehype-shiki 플러그인 사용
// 검색: "shiki next.js markdown", "rehype-shiki"
```

#### 정적 생성 (generateStaticParams)

빌드 시점에 모든 포스트 페이지를 미리 생성합니다:

```typescript
export function generateStaticParams() {
  const posts = getAllPosts();
  return posts.map((post) => ({ slug: post.slug }));
}
```

#### SEO: generateMetadata

```typescript
export async function generateMetadata({ params }: { params: { slug: string } }) {
  const post = await getPostBySlug(params.slug);
  return {
    title: post.frontmatter.title,
    description: post.frontmatter.description,
  };
}
```

### 5-2. SeriesNav 컴포넌트

포스트가 시리즈에 속하면 본문 하단에 시리즈 네비게이션을 표시합니다.

**동작 흐름:**

```
/posts/[slug] (시리즈 첫 번째 글)
  └─ 본문 내용
  └─ [SeriesNav]
       시리즈: AppRouter 완전정복 (1/6)
       ──────────────────────────────
       다음 글 →  AppRouter 심화편
```

```typescript
// SeriesNav 힌트:
// frontmatter의 series, seriesOrder를 활용
// 같은 series인 포스트를 getAllPosts()로 가져와 seriesOrder 정렬
// 현재 포스트의 이전/다음 포스트 링크 표시

interface SeriesNavProps {
  currentSlug: string;
  series: string;
  seriesOrder: number;
}

// 이전 글 (seriesOrder - 1), 다음 글 (seriesOrder + 1)
// 시리즈 첫 글이면 이전 버튼 숨김, 마지막 글이면 다음 버튼 숨김
```

> 💡 **컴포넌트 위치:** `posts/[slug]/_components/SeriesNav.tsx`

### 5-3. 목차(TOC) 컴포넌트 (선택)

```
동작 원리:
1. 마크다운에서 # ## ### 헤딩을 추출
2. 목록으로 표시
3. 스크롤에 따라 현재 위치 하이라이팅 (Intersection Observer)
```

**검색 키워드:** "intersection observer scroll spy", "table of contents react"

### 5-4. Phase 5 체크리스트

- [ ] 포스트 상세 페이지 구현 (`/posts/[slug]`, 동적 라우팅)
- [ ] **`_components/` 폴더 생성** (Co-location 활용) ✨
- [ ] PostHeader 컴포넌트 — 제목, 날짜, 태그 (`posts/[slug]/_components/`)
- [ ] PostContent 컴포넌트 — 마크다운 렌더링 (`posts/[slug]/_components/`)
- [ ] SeriesNav 컴포넌트 — 시리즈 이전/다음 포스트 네비게이션 (`posts/[slug]/_components/`)
- [ ] 코드 하이라이팅 (Shiki)
- [ ] generateStaticParams로 정적 생성
- [ ] generateMetadata로 SEO 설정
- [ ] (선택) TableOfContents (`_components/`)
- [ ] 반응형 확인 (모바일/데스크톱)
- [ ] Git 커밋

---

## Phase 6 - 시리즈 & 검색 & 소개

> **목표:** 시리즈 페이지, 검색 기능, 소개 페이지를 만듭니다
> **소요:** 3~4일
> **배우는 것:** 클라이언트 사이드 검색, 동적 라우팅 심화

### 6-1. 시리즈 목록 페이지 (`/series`)

**`/series/[slug]` 별도 페이지 없음.** 시리즈 카드 클릭 → 시리즈의 첫 번째 포스트로 바로 이동. 이후 읽기 흐름은 각 포스트 하단의 `SeriesNav`(Phase 5에서 구현)가 담당.

**페이지 구조:**

```
/series
  ├─ 시리즈 카드 A  (첫 번째 포스트 썸네일, 시리즈 제목, "6 posts" 뱃지)
  │    └─ 클릭 → /posts/[첫번째-slug]
  ├─ 시리즈 카드 B
  └─ ...
```

**`lib/posts.ts`에 추가할 함수:**

```typescript
// getAllSeries(): 시리즈별로 그룹핑
// 반환: { name: string, postCount: number, firstPost: PostMeta }[]

// 동작:
// 1. 모든 포스트에서 series 필드가 있는 것만 필터
// 2. series 이름으로 그룹핑
// 3. 각 그룹을 seriesOrder 오름차순 정렬 → 첫 번째 포스트 추출
// 4. 반환
```

**SeriesCard 컴포넌트:**
- 첫 번째 포스트의 썸네일 (`extractFirstImage` 재활용)
- 시리즈 이름
- `N posts` 뱃지
- 클릭 시 → 첫 번째 포스트(`/posts/[slug]`)로 이동

### 6-2. 검색 페이지 (`/search`)

포스트 제목 / 본문 키워드 검색.

**구현 방식 비교:**

| 방식 | 설명 | 난이도 |
|------|------|--------|
| **클라이언트 사이드** | 빌드 시 전체 포스트 인덱스를 JSON으로 만들어 브라우저에서 검색 | 쉬움 |
| **API 검색** | 검색어를 서버에 보내고 서버에서 필터링 | 보통 |

**클라이언트 사이드 검색 권장 (포스트 수가 적은 초기에):**

```typescript
// 빌드 시 생성: public/search-index.json
// { slug, title, description, tags, content(일부) }[]

// 검색 페이지에서:
"use client";
// 1. fetch("/search-index.json")으로 인덱스 로드
// 2. 입력한 키워드로 title, description, content 필터링
// 3. 결과를 PostCard 형태로 표시
```

**검색 키워드:** "next.js static search fuse.js", "lunr.js next.js blog search"

### 6-3. 소개 페이지 (`/about`)

자유롭게 만드세요. 이것도 마크다운 파일로 관리하면 깔끔합니다 (`content/about.md`).

### 6-4. Phase 6 체크리스트

- [ ] `getAllSeries()` 함수 구현 (`lib/posts.ts`) — 시리즈 그룹핑 + 첫 번째 포스트 추출
- [ ] 시리즈 목록 페이지 구현 (`/series`) — `/series/[slug]` 없음
- [ ] SeriesCard 컴포넌트 — 썸네일, 시리즈 이름, "N posts" 뱃지, 클릭 시 첫 번째 포스트로 이동
- [ ] 검색 인덱스 생성 (`public/search-index.json`)
- [ ] 검색 페이지 구현 (`/search`, 클라이언트 사이드)
- [ ] 소개 페이지 구현 (`/about`)
- [ ] 반응형 확인
- [ ] Git 커밋

---

## Phase 7 - 웹 어드민 (인증 + CRUD)

> **목표:** 관리자만 접근 가능한 어드민 페이지에서 포스트를 생성/수정/삭제합니다
> **소요:** 4~6일
> **배우는 것:** 인증(Auth), API Routes, 파일 쓰기, 폼 처리

### 7-1. 인증 방식 선택

| 방식 | 설명 | 난이도 |
|------|------|--------|
| **환경변수 비밀번호** | .env에 비밀번호 저장, 단순 비교 | 쉬움 |
| **NextAuth Credentials** | 이메일/비밀번호 로그인 | 보통 |
| **NextAuth GitHub** | GitHub OAuth 로그인 | 보통 |

**NextAuth 방식 (학습용으로 권장):**
- 세션 관리, JWT 토큰 등을 NextAuth가 처리
- 나중에 OAuth도 쉽게 추가 가능

**검색 키워드:** "next-auth credentials provider app router"

### 7-2. 로그인 페이지

`src/app/(admin)/admin/login/page.tsx`

**만들 것:** 이메일/비밀번호 폼, 로그인 버튼, 에러 메시지

**이건 클라이언트 컴포넌트!** (`"use client"`) — 폼 입력 = useState 필요

### 7-3. 어드민 레이아웃 (인증 가드)

`src/app/(admin)/layout.tsx`

**동작:**
1. 로그인 확인 → 안 했으면 로그인 페이지로 리다이렉트
2. 로그인 했으면 → Sidebar 포함 레이아웃 표시

### 7-4. 파일 쓰기 함수

`src/lib/posts.ts`에 추가:

#### savePost

```
입력: slug, frontmatter, content
동작: content/posts/{slug}.md 파일을 생성/덮어쓰기
힌트: matter.stringify(content, frontmatter) + fs.writeFileSync()
```

#### deletePost

```
입력: slug
동작: content/posts/{slug}.md 삭제
힌트: fs.existsSync() 확인 후 fs.unlinkSync()
```

### 7-5. API 라우트

```
클라이언트 (브라우저)              서버 (Next.js)
    │                               │
    │  POST /api/posts           ──▶│  savePost() → 파일 생성
    │  PUT /api/posts/[slug]     ──▶│  savePost() → 파일 수정
    │  DELETE /api/posts/[slug]  ──▶│  deletePost() → 파일 삭제
```

**만들 API 라우트:**

```
src/app/api/posts/route.ts           → GET (목록), POST (생성)
src/app/api/posts/[slug]/route.ts    → GET (상세), PUT (수정), DELETE (삭제)
```

**각 API에서 해야 할 것:**
1. 인증 확인
2. 입력 검증
3. 파일 읽기/쓰기/삭제
4. 응답 반환

**검색 키워드:** "next.js route handler app router"

### 7-6. 어드민 페이지들

#### 대시보드 (`/admin`)
전체 포스트 수, 공개/초안 수, 최근 포스트

#### 포스트 목록 (`/admin/posts`)
`PostTable` — 제목, 상태, 날짜, 수정/삭제 버튼

#### 포스트 작성/수정

> 💡 **Co-location 활용**
> 어드민의 복잡한 폼 컴포넌트들은 `app/(admin)/admin/posts/new/_components/`에 두세요.
>
> ```
> app/(admin)/admin/posts/new/
> ├── page.tsx
> └── _components/
>     ├── MarkdownEditor.tsx
>     ├── FrontmatterForm.tsx
>     ├── PublishButton.tsx
>     └── TagSelector.tsx
> ```

`PostEditor` — 제목, slug, 마크다운 textarea, 태그, 시리즈, published 토글, 저장 버튼

```typescript
"use client";
import { useState } from "react";

export default function PostEditor() {
  const [title, setTitle] = useState("");
  const [content, setContent] = useState("");

  const handleSubmit = async () => {
    // fetch("/api/posts", { method: "POST", body: ... })
  };
}
```

수정 페이지는 PostEditor를 재사용하되 기존 데이터를 불러와서 채움.

### 7-7. Phase 7 체크리스트

- [ ] 인증 방식 결정 & 구현
- [ ] 로그인 페이지 구현
- [ ] 어드민 레이아웃 (인증 가드 + Sidebar)
- [ ] `savePost()`, `deletePost()` 구현 (`lib/posts.ts`)
- [ ] API Routes: POST, PUT, DELETE, GET (`app/api/`)
- [ ] 대시보드, 포스트 목록, 작성, 수정 페이지
- [ ] **어드민 전용 컴포넌트** `_components/`에 배치 ✨
- [ ] 작성/수정/삭제 → .md 파일 변경 확인
- [ ] Git 커밋

---

## Phase 8 - Obsidian + Git 동기화

> **목표:** Obsidian에서 push하면 블로그에 반영되는 양방향 시스템 완성
> **소요:** 2~3일
> **배우는 것:** Git 워크플로우, Obsidian 설정, webhook 개념

### 8-1. Obsidian Vault 연결

**가장 간단한 방법: 블로그 저장소 = Obsidian Vault**

1. Obsidian에서 "Open folder as vault" → 블로그 프로젝트 폴더 선택
2. `content/posts/` 안의 파일만 편집
3. `.gitignore`에 `.obsidian/` 추가

### 8-2. Obsidian Git Plugin 설정

1. Obsidian → Settings → Community Plugins → "Obsidian Git" 검색 & 설치
2. 설정:
   - Commit message: `blog: {{date}}`
   - Auto backup: 끄기 (수동 push 권장)
   - Push on backup: 켜기

### 8-3. 워크플로우 테스트

**Obsidian → 블로그:**
1. Obsidian에서 `content/posts/new-post.md` 작성 (frontmatter 포함)
2. Git Plugin으로 commit & push
3. `npm run dev`에서 새 포스트 확인

**웹 어드민 → Obsidian:**
1. 어드민에서 새 글 작성
2. 터미널에서 `git add . && git commit && git push`
3. Obsidian에서 pull → 새 파일 확인

### 8-4. 충돌 방지

1인 블로그에서는 **규칙으로 해결**하면 충분합니다:
- 한 번에 한 곳에서만 수정
- 수정 전에 항상 pull 먼저

### 8-5. 에러 처리 강화

```typescript
// getAllPosts()에서 잘못된 파일 건너뛰기

// validateFrontmatter(data) 함수를 만들어서
// 필수 필드(title, createdAt, tags, published)가 있는지 확인
// 검증 실패 → console.warn 후 건너뛰기
// 파일 읽기 실패 → try/catch로 잡아서 건너뛰기
```

### 8-6. Phase 8 체크리스트

- [ ] .gitignore에 `.obsidian/` 추가
- [ ] Obsidian Vault 연결 & Git Plugin 설정
- [ ] Obsidian → push → 블로그 반영 확인
- [ ] 어드민 → push → Obsidian pull 확인
- [ ] frontmatter 검증 & 에러 처리
- [ ] Git 커밋

---

## Phase 9 - 댓글 기능

> **목표:** 독자들이 포스트에 댓글을 남길 수 있도록 합니다
> **소요:** 1~3일
> **배우는 것:** 외부 라이브러리 통합, OAuth 개념, React 컴포넌트 통합

### 9-1. 댓글 시스템 선택지

| 방식 | 대표 서비스 | 난이도 | 비용 | 특징 |
|------|------------|--------|------|------|
| **GitHub 기반** | Giscus, Utterances | 쉬움 | 무료 | GitHub 계정으로 댓글, 개발자 블로그에 최적 |
| **외부 서비스** | Disqus | 쉬움 | 무료/유료 | 범용적, 광고 있음, 무거움 |
| **직접 구현** | 없음 (직접 만듦) | 어려움 | DB 비용 | 완전한 커스터마이징, 공부 많이 됨 |

### 9-2. 각 방식 상세 비교

#### 방식 A: Giscus (GitHub Discussions 기반) — 권장

```
독자가 댓글 작성
     │
     ▼
GitHub 계정으로 로그인 (OAuth)
     │
     ▼
GitHub Discussions에 댓글이 저장됨
     │
     ▼
블로그에서 실시간으로 표시
```

**장점:**
- 무료, 광고 없음
- GitHub Discussions에 저장 → 데이터가 내 저장소에 있음
- 마크다운 문법으로 댓글 작성 가능
- 대댓글(nested replies) 지원
- 다크 모드 지원
- 우리 프로젝트가 이미 GitHub 기반이라 자연스러운 선택

**단점:**
- 댓글 쓰려면 GitHub 계정이 필요 (비개발자에겐 진입 장벽)

#### 방식 B: Utterances (GitHub Issues 기반)

Giscus와 비슷하지만 GitHub **Issues**에 댓글을 저장합니다.

- Utterances: Issues 사용, 대댓글 불가
- Giscus: Discussions 사용, 대댓글 가능, 더 다양한 기능

**결론: Giscus가 상위호환이므로 Giscus를 추천합니다.**

#### 방식 C: 직접 구현 (도전 과제)

DB에 댓글을 저장하는 시스템을 직접 만듭니다. Giscus를 먼저 연동하고, 이후에 도전하세요.

### 9-3. Giscus 연동하기 (권장 방식)

#### Step 1: GitHub Discussions 활성화

1. GitHub 저장소 → **Settings** 탭
2. 아래로 스크롤 → **Features** 섹션
3. **Discussions** 체크박스 활성화

```
저장소 Settings → Features → ✅ Discussions
```

#### Step 2: Giscus 앱 설치

1. [giscus.app](https://giscus.app) 방문
2. **GitHub에 Giscus 앱 설치** 링크 클릭
3. 본인 저장소에 Giscus 앱 권한 부여

#### Step 3: Giscus 설정 생성

[giscus.app](https://giscus.app)에서 설정을 만듭니다:

1. **Repository:** `your-username/my-blog` 입력
2. **Page ↔ Discussions Mapping:** `pathname` 선택 (권장)
   - 각 포스트의 URL 경로가 Discussion과 매핑됨
   - `/posts/hello-world` → 별도의 Discussion 스레드 생성
3. **Discussion Category:** `Announcements` 선택 (권장)
   - 일반 유저가 새 Discussion을 만들 수 없는 카테고리
4. **Theme:** `preferred_color_scheme` (시스템 다크모드 따라감)
5. **설정 완료 시 script 태그가 생성됨** → 이 값들을 기록해두세요

생성되는 코드 예시:
```html
<script src="https://giscus.app/client.js"
  data-repo="your-username/my-blog"
  data-repo-id="R_xxxxxxxxx"
  data-category="Announcements"
  data-category-id="DIC_xxxxxxxxx"
  data-mapping="pathname"
  data-strict="0"
  data-reactions-enabled="1"
  data-emit-metadata="0"
  data-input-position="bottom"
  data-theme="preferred_color_scheme"
  data-lang="ko"
  crossorigin="anonymous"
  async>
</script>
```

**여기서 필요한 값들:**
- `data-repo`: 저장소 이름
- `data-repo-id`: 저장소 ID
- `data-category`: 카테고리 이름
- `data-category-id`: 카테고리 ID

#### Step 4: Giscus React 패키지 설치

```bash
npm install @giscus/react
```

**왜 React 패키지를 쓰나?**
- `<script>` 태그 직접 넣어도 되지만
- React 컴포넌트로 쓰면 Next.js와 더 잘 통합됨
- 테마 변경, 언어 변경 등을 props로 쉽게 제어 가능

#### Step 5: 댓글 컴포넌트 만들기

`src/components/blog/Comments.tsx`:

```typescript
"use client";

import Giscus from "@giscus/react";

/**
 * 블로그 포스트 하단에 표시되는 댓글 컴포넌트
 *
 * 동작:
 * 1. Giscus가 현재 페이지의 pathname을 기반으로
 *    GitHub Discussions에서 해당 스레드를 찾음
 * 2. 스레드가 없으면 첫 댓글 작성 시 자동 생성
 * 3. GitHub OAuth로 로그인 후 댓글 작성 가능
 */
export default function Comments() {
  return (
    <Giscus
      id="comments"
      repo="your-username/my-blog"          // ← 본인 저장소로 변경!
      repoId="R_xxxxxxxxx"                   // ← giscus.app에서 확인한 값
      category="Announcements"
      categoryId="DIC_xxxxxxxxx"              // ← giscus.app에서 확인한 값
      mapping="pathname"
      strict="0"
      reactionsEnabled="1"
      emitMetadata="0"
      inputPosition="bottom"
      theme="preferred_color_scheme"
      lang="ko"
      loading="lazy"                          // 스크롤해서 보일 때 로드
    />
  );
}
```

**주의: `"use client"` 필수!** Giscus는 브라우저에서 동작하는 컴포넌트입니다.

**직접 해보세요!** 4가지 값을 본인 것으로 교체:
- `repo`, `repoId`, `category`, `categoryId`

#### Step 6: 포스트 상세 페이지에 추가

```typescript
import Comments from "@/components/blog/Comments";

export default async function PostPage({ params }) {
  const post = await getPostBySlug(params.slug);

  return (
    <article>
      {/* 포스트 내용 */}
      <h1>{post.frontmatter.title}</h1>
      <div>{/* 본문 */}</div>

      {/* 댓글 */}
      <hr className="my-8" />
      <section className="mt-8">
        <h2 className="text-2xl font-bold mb-4">댓글</h2>
        <Comments />
      </section>
    </article>
  );
}
```

### 9-4. 다크 모드 연동

블로그에 다크 모드가 있다면 댓글도 테마에 맞춰야 합니다:

```typescript
"use client";

import Giscus from "@giscus/react";
import { useTheme } from "next-themes";

export default function Comments() {
  const { theme } = useTheme();
  const giscusTheme = theme === "dark" ? "dark" : "light";

  return (
    <Giscus
      // ... 기존 props
      theme={giscusTheme}  // 동적으로 변경!
    />
  );
}
```

**동작:** 다크 모드 토글 → `theme` 변경 → Giscus도 자동 테마 변경

### 9-5. 댓글 관리

Giscus의 댓글은 GitHub Discussions에 저장됩니다.

**관리 방법:**
1. GitHub 저장소 → **Discussions** 탭에서 모든 댓글 확인/관리
2. 스팸 댓글은 Discussion에서 직접 삭제
3. 특정 Discussion을 잠글 수도 있음

```
GitHub 저장소 → Discussions 탭
  ├── /posts/hello-world  (자동 생성된 Discussion)
  │   ├── 댓글 1
  │   ├── 댓글 2
  │   └── 대댓글
  └── /posts/react-guide  (자동 생성된 Discussion)
      └── 댓글 1
```

### 9-6. (도전 과제) 직접 댓글 시스템 만들기

> **선택사항**입니다. Giscus로 충분하다면 건너뛰세요.

직접 만들면 GitHub 계정 없는 사람도 댓글을 남길 수 있습니다.

#### 필요한 것들

```
1. 댓글 저장소 (DB)
   - 옵션 A: Supabase (무료, 클라우드 DB, 권장)
   - 옵션 B: Prisma + SQLite
   - 옵션 C: PlanetScale, Neon 등

2. API 라우트
   - POST /api/comments     (댓글 작성)
   - GET  /api/comments      (댓글 목록)
   - DELETE /api/comments/[id] (댓글 삭제 - 어드민만)

3. 댓글 UI 컴포넌트
   - 댓글 입력 폼
   - 댓글 목록
   - (선택) 대댓글
```

#### DB 스키마 예시 (Prisma)

```prisma
model Comment {
  id        String   @id @default(cuid())
  postSlug  String                          // 어떤 포스트의 댓글인지
  author    String                          // 작성자 이름
  email     String?                         // 이메일 (선택, 비공개)
  content   String                          // 댓글 내용
  parentId  String?                         // 대댓글인 경우 부모 댓글 ID
  parent    Comment? @relation("Replies", fields: [parentId], references: [id])
  replies   Comment[] @relation("Replies")
  createdAt DateTime @default(now())
}
```

#### API 구현 힌트

```typescript
// POST /api/comments
export async function POST(request: Request) {
  // 1. body에서 { postSlug, author, content, parentId? } 추출
  // 2. 입력 검증 (빈 문자열, 최대 길이)
  // 3. (선택) 스팸 방지 (rate limiting, 금지 단어)
  // 4. DB에 저장
  // 5. 성공 응답
}

// GET /api/comments?postSlug=hello-world
export async function GET(request: Request) {
  // 1. 쿼리에서 postSlug 추출
  // 2. 해당 포스트 댓글 조회 (replies 포함)
  // 3. 날짜순 정렬
  // 4. 반환
}
```

#### 댓글 UI 힌트

```typescript
"use client";
import { useState } from "react";

export default function CommentForm({ postSlug }: { postSlug: string }) {
  const [author, setAuthor] = useState("");
  const [content, setContent] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    // fetch("/api/comments", { method: "POST", body: ... })
    // 성공 시 폼 초기화 & 댓글 목록 새로고침
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* 이름 입력 */}
      {/* 댓글 내용 textarea */}
      {/* 제출 버튼 */}
    </form>
  );
}
```

#### 보안 주의사항

| 위험 | 설명 | 방지 방법 |
|------|------|-----------|
| **XSS** | 댓글에 `<script>` 삽입 | HTML 이스케이프, 마크다운만 허용 |
| **스팸** | 봇이 대량 댓글 | Rate limiting, CAPTCHA, 허니팟 |
| **SQL Injection** | 악의적 쿼리 | Prisma가 자동 방지 (ORM 장점) |
| **CSRF** | 외부에서 댓글 요청 | CSRF 토큰, Origin 헤더 확인 |

**초보자라면 Giscus를 먼저 쓰고, 보안 공부 후 직접 구현하세요.**

### 9-7. Phase 9 체크리스트

**Giscus 방식 (권장):**
- [ ] GitHub Discussions 활성화
- [ ] giscus.app에서 Giscus 앱 설치 & 설정
- [ ] `@giscus/react` 패키지 설치
- [ ] `Comments.tsx` 컴포넌트 생성 (본인 repo 정보로)
- [ ] 포스트 상세 페이지에 `<Comments />` 추가
- [ ] 로컬에서 댓글 작성 테스트
- [ ] 다크 모드 연동 확인
- [ ] 배포 후 정상 동작 확인
- [ ] Git 커밋

**직접 구현 (도전 과제):**
- [ ] Comment DB 스키마 작성
- [ ] POST/GET/DELETE /api/comments 구현
- [ ] CommentForm, CommentList 컴포넌트
- [ ] XSS, 스팸 방지
- [ ] (선택) 대댓글, 어드민 댓글 관리
- [ ] Git 커밋

---

## Phase 10 - 배포 & 실전 운영

> **목표:** 인터넷에 블로그를 공개합니다
> **소요:** 1~2일
> **배우는 것:** Vercel 배포, 환경변수, GitHub API

### 10-1. GitHub에 코드 올리기

```bash
git remote add origin https://github.com/your-username/my-blog.git
git push -u origin main
```

### 10-2. Vercel 배포

1. [vercel.com](https://vercel.com) 가입
2. "Import Project" → GitHub 저장소 선택
3. 환경변수 설정 (ADMIN_PASSWORD, AUTH_SECRET 등)
4. Deploy!

**Vercel의 마법:** push할 때마다 자동 빌드 & 배포
→ Obsidian에서 push = 블로그 자동 업데이트!

### 10-3. Vercel에서 어드민 파일 쓰기 문제

Vercel은 서버리스 환경이라 **런타임에 파일 쓰기가 불가**합니다.

**해결: GitHub API로 파일 커밋**

```
웹 어드민에서 글 작성
     │
     ▼
GitHub API로 .md 파일 커밋
     │
     ▼
Vercel이 자동 감지 → 재빌드
     │
     ▼
블로그에 반영!
```

**구현할 것:** `src/lib/github.ts`

```typescript
// GitHub REST API를 사용해서 파일을 생성/수정/삭제
// PUT /repos/{owner}/{repo}/contents/{path}

// 필요한 환경변수:
// GITHUB_TOKEN — GitHub Personal Access Token
// GITHUB_OWNER — GitHub 사용자명
// GITHUB_REPO  — 저장소 이름
```

**GitHub Token 발급:**
1. GitHub → Settings → Developer Settings → Personal Access Tokens
2. "Generate new token (classic)"
3. repo 권한 체크
4. 토큰 복사 → .env에 저장

**검색 키워드:** "github api create file contents", "github personal access token"

### 10-4. Phase 10 체크리스트

- [ ] GitHub 저장소 생성 & push
- [ ] Vercel 연결 & 배포
- [ ] 환경변수 설정
- [ ] 배포 후 블로그 정상 동작 확인
- [ ] Obsidian push → 자동 빌드 확인
- [ ] GitHub API 파일 커밋 기능 구현
- [ ] 어드민 → GitHub API → 자동 반영 확인
- [ ] (선택) 커스텀 도메인 연결
- [ ] Git 커밋

---

## Phase 11 - 고도화 (선택)

> 블로그 완성 후 하나씩 도전하세요. 전부 할 필요 없습니다.

### 11-1. RSS 피드
`/feed.xml` 경로에 RSS 피드 생성. 검색: "next.js rss feed"

### 11-2. 사이트맵
`/sitemap.xml` 자동 생성. 검색 엔진 최적화. 검색: "next.js sitemap"

### 11-3. 이미지 최적화
Next.js `<Image>` 컴포넌트 활용. 검색: "next.js Image component"

### 11-4. 조회수 카운터
- DB 없이: Vercel Analytics, GoatCounter
- DB로: Upstash Redis (무료 티어)

### 11-5. 마크다운 에디터 업그레이드
textarea → Monaco Editor (VS Code 에디터) 또는 MDX 지원 + 실시간 미리보기

### 11-6. 페이지네이션 / 무한 스크롤
포스트 많아지면 필요. 페이지네이션 or Intersection Observer로 무한 스크롤.

### 11-7. 이메일 구독 (뉴스레터)
Resend, Mailchimp 연동. 새 글 올라오면 구독자에게 이메일.

---

## 부록 A - 디자인 패턴 & 설계 원칙

### A-1. 컴포넌트 설계 원칙

#### 단일 책임 원칙 (Single Responsibility)

하나의 컴포넌트는 하나의 역할만:

```
나쁜 예: PostPage에서 데이터 fetch + 마크다운 변환 + 렌더링 + 댓글 + TOC 전부 처리

좋은 예:
  PostPage        → 데이터를 가져와서 하위 컴포넌트에 전달
  PostContent     → 마크다운 HTML 렌더링
  PostMeta        → 제목, 날짜, 태그 표시
  TableOfContents → 목차
  Comments        → 댓글
```

#### 서버 vs 클라이언트 컴포넌트 분리

```
규칙:
- 기본은 서버 컴포넌트 (아무것도 안 쓰면 서버)
- useState, useEffect, onClick 등이 필요할 때만 "use client"
- 서버 안에 클라이언트를 넣을 수 있음 (반대는 안됨)
- "use client" 경계를 최대한 아래로 (작은 범위에만 사용)

예시:
  PostPage (서버) — 데이터 fetch
    ├── PostContent (서버) — HTML 렌더링
    ├── TableOfContents (클라이언트) — 스크롤 감지
    └── Comments (클라이언트) — 사용자 인터랙션
```

#### Props로 데이터 흘려보내기

```typescript
// 서버 컴포넌트에서 데이터를 가져와서 클라이언트에 props로 전달
export default async function PostPage({ params }) {
  const post = await getPostBySlug(params.slug);

  return (
    <>
      <PostContent html={post.content} />
      <TableOfContents headings={post.headings} />
    </>
  );
}
```

### A-2. 폴더 구조 패턴

#### Feature-based (기능별 분류) — 우리가 사용하는 방식

```
components/
├── blog/       ← 블로그 기능 관련
├── admin/      ← 어드민 기능 관련
└── common/     ← 공통
```

**장점:** 관련된 파일이 한 곳에 모여있어서 찾기 쉬움

#### Type-based (유형별 분류) — 비교용

```
components/     ← 모든 컴포넌트
hooks/          ← 모든 커스텀 훅
utils/          ← 모든 유틸리티
```

### A-3. 데이터 흐름

```
content/posts/*.md (마크다운 파일)
        │
        ▼
src/lib/posts.ts (파일 읽기 & 파싱)
        │
        ▼
src/app/**/page.tsx (서버 컴포넌트에서 함수 호출)
        │
        ▼
src/components/** (컴포넌트에서 렌더링)
```

**규칙:**
- 데이터는 **위에서 아래로** (서버 → 클라이언트, 부모 → 자식)
- 파일 읽기(fs)는 **서버에서만**
- API 호출은 **클라이언트에서만** (어드민 CRUD)

### A-4. 네이밍 컨벤션

```
파일명:
  컴포넌트  → PascalCase    (PostCard.tsx, Header.tsx)
  유틸리티  → camelCase     (posts.ts, utils.ts)
  페이지    → page.tsx      (Next.js 규칙)
  레이아웃  → layout.tsx    (Next.js 규칙)

변수/함수:
  함수     → camelCase     (getAllPosts, formatDate)
  상수     → UPPER_SNAKE   (POSTS_DIRECTORY)
  타입     → PascalCase    (PostFrontmatter, Post)
  컴포넌트  → PascalCase    (PostCard, Header)

CSS 클래스:
  Tailwind  → 유틸리티 그대로 (text-lg, bg-gray-100)
```

---

## 부록 B - 유용한 레퍼런스

### B-1. 핵심 패키지 사용법

#### gray-matter

```typescript
import matter from "gray-matter";

// 파싱: 마크다운 → { data(frontmatter), content(본문) }
const { data, content } = matter(fileString);

// 생성: frontmatter + 본문 → 마크다운 문자열
const md = matter.stringify("본문", { title: "제목", tags: ["태그"] });
```

#### Node.js fs (파일 시스템)

```typescript
import fs from "fs";
import path from "path";

fs.readdirSync(dir)                    // 폴더 내 파일 목록
fs.readFileSync(path, 'utf-8')         // 파일 읽기
fs.writeFileSync(path, data, 'utf-8')  // 파일 쓰기
fs.unlinkSync(path)                    // 파일 삭제
fs.existsSync(path)                    // 파일 존재 확인
fs.mkdirSync(dir, { recursive: true }) // 폴더 생성
path.join(a, b, c)                     // 경로 합치기
```

#### remark

```typescript
import { remark } from "remark";
import remarkHtml from "remark-html";

const result = await remark().use(remarkHtml).process(markdownString);
const html = result.toString();
```

### B-2. 검색 키워드 모음

| 주제 | 검색 키워드 |
|------|-------------|
| 프로젝트 설정 | "create-next-app typescript tailwind" |
| App Router | "next.js app router tutorial 2024" |
| 레이아웃 | "next.js nested layouts app router" |
| 동적 라우팅 | "next.js dynamic routes [slug]" |
| 서버 컴포넌트 | "react server components explained" |
| frontmatter | "markdown frontmatter gray-matter" |
| 마크다운 렌더링 | "next.js markdown blog remark" |
| 코드 하이라이팅 | "shiki next.js markdown" |
| 정적 생성 | "next.js generateStaticParams" |
| SEO | "next.js generateMetadata" |
| 인증 | "next-auth credentials app router" |
| API Routes | "next.js route handler app router" |
| 다크 모드 | "next-themes tailwind dark mode" |
| 타이포그래피 | "tailwind typography prose plugin" |
| 댓글 (Giscus) | "giscus next.js react" |
| 댓글 (직접) | "next.js comment system tutorial" |
| 배포 | "deploy next.js vercel" |
| GitHub API | "github api create file contents" |
| Obsidian Git | "obsidian git plugin setup" |

### B-3. 참고할 오픈소스

- **contentlayer** — 마크다운 파일을 타입 안전하게 관리
- **next-mdx-remote** — MDX 렌더링
- **Nextra** — Next.js 기반 문서/블로그 프레임워크
- **Astro** — 정적 사이트 생성기 (비교용)
- **tailwind-nextjs-starter-blog** — Tailwind + Next.js 블로그 템플릿

소스 코드를 GitHub에서 구경하면 구조를 참고할 수 있습니다.
하지만 그대로 복사하지 말고, 원리를 이해하고 본인 방식으로 만들어보세요.

---

## 전체 진행 요약

```
Phase 0  [개발 환경 & 개념]         ░░░░░░░░░░  1~2일
Phase 1  [프로젝트 생성 & 설계]      ░░░░░░░░░░  1~2일
Phase 2  [레이아웃 & 공통]          ░░░░░░░░░░  2~3일
Phase 3  [마크다운 시스템]          ░░░░░░░░░░  3~5일  ← 핵심!
Phase 4  [메인 페이지]             ░░░░░░░░░░  2~3일  ← 현재!
Phase 5  [포스트 상세 페이지]        ░░░░░░░░░░  2~3일  ← 핵심!
Phase 6  [시리즈 & 검색 & 소개]     ░░░░░░░░░░  3~4일
Phase 7  [웹 어드민]               ░░░░░░░░░░  4~6일
Phase 8  [Obsidian + Git]         ░░░░░░░░░░  2~3일
Phase 9  [댓글]                   ░░░░░░░░░░  1~3일
Phase 10 [배포]                   ░░░░░░░░░░  1~2일
Phase 11 [고도화]                 ░░░░░░░░░░  원하는 만큼
                                           ────────
                                      총 약 4~6주
```

**원칙:**
1. 한 Phase를 끝내고 다음으로
2. 각 Phase 끝에 Git 커밋
3. 완벽하지 않아도 됨 — 일단 돌아가게 만들고, 나중에 개선
4. 막히면 Claude에게 "Phase X-Y에서 막혔어" 라고 말하기
5. 검색을 두려워하지 않기

---

> 이 로드맵은 살아있는 문서입니다.
> 진행하면서 배운 것, 바꾼 것을 여기에 업데이트하세요.
> 화이팅!
