---
title: SEO
subject: "[[Dev Note/Dev Note|Dev Note]]"
reference: ""
date: 2026-02-23 16:21
description: ""
tags:
  - seo
  - 개념
series: ""
seriesOrder:
published: false
---

# obslog SEO 전략 가이드

## 현재 상태 진단

### ❌ 없는 것들

- `sitemap.xml` 없음
- `robots.txt` 없음
- 루트 `layout.tsx`에 글로벌 `metadata` 없음
- 각 포스트 페이지의 동적 `metadata` 미적용

### ⚠️ 미흡한 것들

```ts
// 현재 (blog)/page.tsx
description: "blog main page"; // 너무 성의없음
```

---

## 전략 로드맵

```
지금 바로     → 메타데이터 정비 (title, description, OG)
오늘 중       → sitemap.ts + robots.ts 추가
배포 후       → Google Search Console에 사이트맵 제출
포스트 쌓이면 → JSON-LD 구조화 데이터
```

---

## 1순위 — 기초 메타데이터

### 루트 `layout.tsx` 글로벌 metadata 추가

```ts
export const metadata: Metadata = {
  title: { default: "Obslog", template: "%s | Obslog" },
  description: "개인 블로그 한 줄 설명",
  metadataBase: new URL("https://your-domain.com"),
  openGraph: {
    type: "website",
    locale: "ko_KR",
    siteName: "Obslog",
  },
  twitter: { card: "summary_large_image" },
};
```

### 각 포스트 페이지에 `generateMetadata` 적용

```ts
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: { images: [post.thumbnail] },
  };
}
```

---

## 2순위 — sitemap + robots

Next.js App Router에서 파일 2개로 해결됩니다.

### `src/app/sitemap.ts`

```ts
import { getAllPosts } from "./lib/posts";

export default async function sitemap() {
  const posts = await getAllPosts();
  return [
    { url: "https://your-domain.com", lastModified: new Date() },
    ...posts.map((post) => ({
      url: `https://your-domain.com/posts/${post.slug}`,
      lastModified: new Date(post.date),
    })),
  ];
}
```

### `src/app/robots.ts`

```ts
export default function robots() {
  return {
    rules: { userAgent: "*", allow: "/" },
    sitemap: "https://your-domain.com/sitemap.xml",
  };
}
```

---

## 3순위 — JSON-LD 구조화 데이터

포스트 페이지에 추가하면 구글 검색 결과에서 작성자·날짜가 표시됩니다.

```tsx
<script
  type="application/ld+json"
  dangerouslySetInnerHTML={{
    __html: JSON.stringify({
      "@context": "https://schema.org",
      "@type": "BlogPosting",
      headline: post.title,
      datePublished: post.date,
      author: { "@type": "Person", name: "작성자명" },
    }),
  }}
/>
```

---

## 4순위 — Google Search Console 등록

코드 작업 없이 즉시 가능:

1. [search.google.com/search-console](https://search.google.com/search-console) 접속
2. 도메인 등록 → sitemap URL 제출 (`/sitemap.xml`)
3. 크롤링 요청 → 보통 1~2주 내 인덱싱
