---
title: App Router vs Page Router
subject: "[[Dev Note]]"
reference:
date: 2026-02-12 15:26
description: 앱 라우터와 페이지 라우터의 차이
tags:
  - "#router"
  - appRouter
  - pageRouter
  - component
  - next
  - 개념
---

|         | AppRouter            | PageRouter          |
| :------ | :------------------- | :------------------ |
| 폴더사용    | app/                | pages/              |
| 기본 컴포넌트 | server               | client              |
| 사용방법    | component fetch      | getStaticProps 사용   |
| 링크 구조   | 폴더 + slug + page.tsx | pages안의 모든 파일이 곧 경로 |

### 왜 PageRouter에서 AppRouter로 점점 바뀌게 되었나

pagesRouter는 직관적이지만, 레이아웃 중첩이 어렵고, 모든 코드가 클라이언트 코드로 섞이기 쉬웠습니다. AppRouter는 좀 복잡해보일 수 있어도, 서버 컴포넌트를 활용해 성능을 극대화하고, 레이아웃을 폴더별로 유연하게 쪼갤 수 있는 엄청난 장점이 있습니다.