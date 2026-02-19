---
title: "Final Blog Project Structure"
subject: "[[folder]]"
reference: ""
date: "2026-02-13"
description: "ROADMAP과 folder.md를 결합한 최종 추천 폴더 구조"
tags:
  - architecture
  - folder
  - final
  - folderPattern
  - setting
series: ""
seriesOrder:
published: false
---

# Blog Project Folder Structure Recommendation

[[Roadmap]]의 Next.js App Router 기반 블로그 + 어드민 프로젝트에 최적화된 폴더 구조 제안입니다.

**핵심 전략: Hybrid Approach**
1. **페이지별 UI**: 해당 페이지 폴더 안에 `_components`로 **Co-location(위치 통합)**
2. **공통 UI**: `src/components`에 **Component Folder Pattern**으로 모음
3. **비즈니스 로직**: `src/lib`에 모아서 순수 함수로 관리

ROADMAP과 folder.md의 장점을 결합하고, 누락된 부분을 보완한 **최종 추천 구조**입니다.

## 전체 구조

```bash
my-blog/
├── content/                     # 마크다운 데이터 (CMS 역할)
│   └── posts/
│       ├── hello-world.md
│       ├── react-hooks-guide.md
│       └── nextjs-routing.md
│
├── public/                      # 정적 파일
│   └── images/
│
├── src/
│   ├── app/                     # Next.js App Router
│   │   │


│   │   ├── (blog)/              # 🌐 공개 블로그 영역 (Route Group)
│   │   │   ├── layout.tsx       # 블로그 레이아웃 (Header + Footer)
│   │   │   ├── page.tsx         # 메인 페이지 (/)
│   │   │   │
│   │   │   ├── posts/
│   │   │   │   └── [slug]/
│   │   │   │       ├── page.tsx
│   │   │   │       └── _components/     # ✅ Co-location
│   │   │   │           ├── PostHeader.tsx
│   │   │   │           ├── PostContent.tsx
│   │   │   │           ├── TableOfContents.tsx
│   │   │   │           └── Comments.tsx
│   │   │   │
│   │   │   ├── tags/
│   │   │   │   ├── page.tsx     # 태그 목록 (/tags)
│   │   │   │   └── [tag]/
│   │   │   │       └── page.tsx # 태그별 포스트 (/tags/react)
│   │   │   │
│   │   │   ├── series/
│   │   │   │   ├── page.tsx     # 시리즈 목록 (/series)
│   │   │   │   └── [slug]/
│   │   │   │       ├── page.tsx
│   │   │   │       └── _components/
│   │   │   │           └── SeriesNav.tsx
│   │   │   │
│   │   │   └── about/
│   │   │       └── page.tsx     # 소개 페이지 (/about)
│   │   │
│   │   ├── (admin)/             # 🔐 어드민 영역 (Route Group)
│   │   │   ├── layout.tsx       # 어드민 레이아웃 (Sidebar + Auth Guard)
│   │   │   └── admin/
│   │   │       ├── page.tsx     # 대시보드 (/admin)
│   │   │       │
│   │   │       ├── posts/
│   │   │       │   ├── page.tsx # 포스트 목록 (/admin/posts)
│   │   │       │   │
│   │   │       │   ├── new/
│   │   │       │   │   ├── page.tsx
│   │   │       │   │   └── _components/     # ✅ Co-location
│   │   │       │   │       ├── MarkdownEditor.tsx
│   │   │       │   │       ├── FrontmatterForm.tsx
│   │   │       │   │       ├── PublishButton.tsx
│   │   │       │   │       └── TagSelector.tsx
│   │   │       │   │
│   │   │       │   └── [slug]/
│   │   │       │       └── edit/
│   │   │       │           └── page.tsx     # 포스트 수정
│   │   │       │
│   │   │       └── login/
│   │   │           └── page.tsx # 로그인 (/admin/login)
│   │   │
│   │   ├── api/                 # API Routes
│   │   │   ├── posts/
│   │   │   │   ├── route.ts     # GET (목록), POST (생성)
│   │   │   │   └── [slug]/
│   │   │   │       └── route.ts # GET, PUT, DELETE
│   │   │   └── auth/
│   │   │       └── [...nextauth]/
│   │   │           └── route.ts # NextAuth API
│   │   │
│   │   ├── layout.tsx           # 루트 레이아웃 (html, body, 폰트)
│   │   └── globals.css          # 전역 스타일
│   │
│   ├── components/              # 전역 재사용 컴포넌트
│   │   │
│   │   ├── ui/                  # ✅ UI 라이브러리 (Barrel Pattern 사용)
│   │   │   │                    # shadcn/ui 스타일 참고
│   │   │   ├── Button/
│   │   │   │   ├── index.ts     # export { Button } from './Button'
│   │   │   │   ├── Button.tsx
│   │   │   │   └── Button.test.tsx
│   │   │   │
│   │   │   ├── Input/
│   │   │   │   ├── index.ts
│   │   │   │   └── Input.tsx
│   │   │   │
│   │   │   ├── Card/
│   │   │   │   ├── index.ts
│   │   │   │   └── Card.tsx
│   │   │   │
│   │   │   └── Badge/
│   │   │       ├── index.ts
│   │   │       └── Badge.tsx
│   │   │
│   │   └── common/              # ❌ 공통 컴포넌트 (Barrel 미사용)
│   │       ├── Header.tsx       # 블로그 헤더
│   │       ├── Footer.tsx       # 블로그 푸터
│   │       ├── ThemeToggle.tsx  # 다크 모드 토글
│   │       └── Sidebar.tsx      # 어드민 사이드바
│   │
│   ├── lib/                     # 비즈니스 로직 (순수 함수)
│   │   ├── posts.ts             # 마크다운 CRUD
│   │   │                        # - getAllPosts()
│   │   │                        # - getPostBySlug(slug)
│   │   │                        # - getAllTags()
│   │   │                        # - getPostsByTag(tag)
│   │   │                        # - getAllSeries()
│   │   │                        # - savePost()
│   │   │                        # - deletePost()
│   │   │
│   │   ├── markdown.ts          # 마크다운 변환
│   │   │                        # - markdownToHtml(md)
│   │   │                        # - extractHeadings(md) → TOC
│   │   │
│   │   ├── auth.ts              # 인증 로직
│   │   ├── github.ts            # GitHub API (Vercel 배포용)
│   │   └── utils.ts             # 유틸리티
│   │                            # - formatDate()
│   │                            # - slugify()
│   │                            # - cn() (clsx)
│   │
│   ├── types/                   # TypeScript 타입 정의
│   │   └── index.ts
│   │       # export interface PostFrontmatter { ... }
│   │       # export interface PostMeta { ... }
│   │       # export interface Post extends PostMeta { ... }
│   │
│   └── styles/
│       └── globals.css          # Tailwind 설정
│
├── .env.local                   # 환경변수 (Git 제외)
├── .gitignore
├── package.json
├── tsconfig.json
├── tailwind.config.ts
├── next.config.ts
└── README.md
```

---

## 핵심 설계 원칙

### 1. Co-location (위치 통합)

**변하는 것들은 함께 둔다** - [[Co-location]] 참고

```bash
# ✅ Good: 포스트 상세 페이지 수정 시 이 폴더만 보면 됨
app/(blog)/posts/[slug]/
├── page.tsx
└── _components/
    ├── PostHeader.tsx      # 이 페이지 전용
    ├── TableOfContents.tsx # 이 페이지 전용
    └── Comments.tsx        # 이 페이지 전용

# ❌ Bad: 여기저기 흩어져 있으면 찾기 어려움
app/(blog)/posts/[slug]/page.tsx
components/blog/PostHeader.tsx
components/blog/TableOfContents.tsx
components/blog/Comments.tsx
```

**장점:**
- 포스트 페이지 수정 시 한 폴더만 확인
- 삭제 용이 (폴더째 제거)
- 의도 명확 ("이 컴포넌트는 여기서만 씀")

---

### 2. Barrel Pattern - 선택적 사용

**UI 라이브러리만** Barrel Pattern 적용 - [[Barrel Pattern]] 참고

```typescript
// ✅ UI 컴포넌트: Barrel 사용 (편의성)
import { Button, Input, Card } from '@/components/ui/button';

// ❌ Common 컴포넌트: 명시적 경로 (순환 참조 방지)
import { Header } from '@/components/common/Header';
import { Footer } from '@/components/common/Footer';
```

**왜?**
- UI 라이브러리는 재사용 빈도 높음 → Barrel로 편의성
- Common은 비즈니스 로직 포함 가능 → 명시적 경로로 안전성

---

### 3. Route Group 활용

`(blog)`와 `(admin)` 완전 분리

```bash
(blog)/              # Header + Footer 레이아웃
├── layout.tsx       # 블로그 전용 레이아웃
└── page.tsx

(admin)/             # Sidebar + Auth Guard 레이아웃
├── layout.tsx       # 어드민 전용 레이아웃
└── admin/
```

**장점:**
- 레이아웃 완전 분리
- URL에 `(blog)`, `(admin)` 포함 안 됨
- 각 영역에 다른 스타일/인증 적용 가능

---

### 4. Server vs Client Component 분리

```typescript
// ✅ Server Component (기본)
// app/(blog)/posts/[slug]/page.tsx
export default async function PostPage({ params }) {
  const post = await getPostBySlug(params.slug);  // fs 사용 가능
  return <PostContent post={post} />;
}

// ✅ Client Component (필요시만)
// app/(blog)/posts/[slug]/_components/Comments.tsx
"use client";
export default function Comments() {
  const [comments, setComments] = useState([]);
  // 상태 관리, 이벤트 핸들러
}
```

**규칙:**
- 기본은 서버 컴포넌트 (파일 읽기, DB 접근 가능)
- `useState`, `useEffect`, `onClick` 필요할 때만 `"use client"`
- `"use client"` 경계를 최대한 아래로 (작은 범위만)

---

## Components 분류 가이드

**어디에 둘지 헷갈릴 때 체크리스트:**

```
Q1. 이 컴포넌트를 한 페이지에서만 쓰나요?
└─ Yes → app/*/page/_components/ (Co-location)
└─ No  → Q2로

Q2. UI 라이브러리 성격인가요? (Button, Input, Card 등)
└─ Yes → components/ui/ (Barrel Pattern)
└─ No  → Q3로

Q3. 여러 페이지에서 공통으로 쓰나요? (Header, Footer 등)
└─ Yes → components/common/
└─ No  → 다시 Q1부터
```

**예시:**

| 컴포넌트 | 위치 | 이유 |
|---------|------|------|
| `PostHeader` | `app/(blog)/posts/[slug]/_components/` | 포스트 상세 페이지 전용 |
| `MarkdownEditor` | `app/(admin)/admin/posts/new/_components/` | 어드민 작성 페이지 전용 |
| `Button` | `components/ui/Button/` | UI 라이브러리, 어디서든 재사용 |
| `Header` | `components/common/Header.tsx` | 모든 블로그 페이지에서 공통 |
| `ThemeToggle` | `components/common/ThemeToggle.tsx` | Header에 포함, 여러 곳에서 사용 |

---

## Import 예시

```typescript
// ✅ 추천 방식

// 1. UI 라이브러리: Barrel 사용
import { Button, Input, Card } from '@/components/ui/button';

// 2. Common: 명시적 경로
import { Header } from '@/components/common/Header';
import { Footer } from '@/components/common/Footer';

// 3. 같은 페이지 내 _components: 상대 경로
import { PostHeader } from './_components/PostHeader';
import { TableOfContents } from './_components/TableOfContents';

// 4. lib: 명시적 경로
import { getAllPosts, getPostBySlug } from '@/lib/posts';
import { formatDate, cn } from '@/lib/utils';

// 5. types
import type { Post, PostMeta } from '@/types';
```

---

## 주의사항

### 1. `_components` vs `components`

```bash
# ✅ Good: underscore prefix (_)
app/posts/[slug]/_components/

# ❌ Bad: underscore 없으면 Next.js가 라우트로 인식할 수 있음
app/posts/[slug]/components/
```

### 2. Barrel Pattern 과용 금지

```typescript
// ❌ Bad: 모든 곳에 index.ts
components/common/
├── Header/
│   ├── index.ts      # 불필요! (한 파일뿐인데 Barrel?)
│   └── Header.tsx
└── Footer/
    ├── index.ts      # 불필요!
    └── Footer.tsx

// ✅ Good: 단순한 컴포넌트는 Barrel 없이
components/common/
├── Header.tsx
└── Footer.tsx
```

**규칙:** 폴더 안에 여러 파일(컴포넌트, 훅, 유틸)이 있을 때만 Barrel 사용

### 3. 순환 참조 방지

```typescript
// ❌ Bad: 순환 참조 위험
// components/common/index.ts
export * from './Header';  // Header가 Footer import
export * from './Footer';  // Footer가 Header import

// ✅ Good: 명시적 경로로 방지
import { Header } from '@/components/common/Header';
import { Footer } from '@/components/common/Footer';
```

---

## 파일 네이밍 규칙

```
컴포넌트:     PascalCase.tsx       (PostCard.tsx, Header.tsx)
유틸리티:     camelCase.ts         (posts.ts, utils.ts)
페이지:       page.tsx             (Next.js 규칙)
레이아웃:     layout.tsx           (Next.js 규칙)
API:          route.ts             (Next.js 규칙)
타입:         index.ts or types.ts
테스트:       *.test.tsx           (Button.test.tsx)
```

---

## 추가 학습 자료

- [[Component Folder Pattern]] - 폴더 구조 패턴 심화
- [[Co-location]] - 위치 통합 개념
- [[Barrel Pattern]] - 배럴 패턴 장단점
- [[UI Library]] - UI 라이브러리 성격 및 분류 기준
- [[Frontend Architecture]] - 전체 아키텍처
- [[Roadmap]] - 프로젝트 단계별 가이드

---

## 결론

이 구조는 다음을 달성합니다:

1. ✅ **Co-location**: 관련 파일이 가까이 있음
2. ✅ **Barrel Pattern 선택적 사용**: UI만 사용, 순환 참조 방지
3. ✅ **명확한 분류**: `ui/` vs `common/` vs `_components/`
4. ✅ **Next.js App Router 최적화**: Route Group, Server Component 활용
5. ✅ **2026년 트렌드 반영**: Hybrid Approach

**프로젝트 시작 시 이 구조를 기본으로, 필요에 따라 조정하세요!** 🚀
