---
title: SearchParams
subject: "[[Frontend]]"
reference: ""
date: 2026-02-26 18:19
description: ""
tags:
  - search-params
  - next
  - frontend
series: ""
seriesOrder:
published: false
---

# SearchParams

searchParams는 뭘까? URL에서 뒤에 ?가 오는 값들입니다. /?page=2&tag=AI, = 뒤에 나오는 2&tag, AI 등이 searchParams입니다. 
```text
  /?page=2&tag=AI
       ↑       ↑
    searchParams
```


이것을 서버 컴포넌트에서는 어떻게 읽을까요? nextjs가 자동으로 props로 넘겨줍니다. 
```text
// URL: /?page=2
  searchParams = { page: "2" }
  searchParams.page  // → "2" (string이야, number 아님!)
```

왜 await가 필요할까? nextjs 15부터는 searchParams가 즉시 값을 반환하는 것이 아니라, Promise로 바뀌었습니다. Promise는 나중에 값을 주겠습니다. 라는 약속이구요. 
```text
  // await 없이
  searchParams          // → Promise (아직 값 없음)
  searchParams.page     // → undefined

  // await 하면
  await searchParams    // → { page: "2" } (이제 값 있음)
  (await searchParams).page  // → "2"
```

  정리하면:

  URL: /?page=2
    ↓
  Next.js가 searchParams Promise를 page.tsx에 전달
    ↓
  await searchParams로 실제 값 꺼내기
    ↓
  { page: "2" }
    ↓
  Number("2") → 2


```typescript
import React from "react";

import { Metadata } from "next";

import PostCard from "../components/common/PostCard";

import { getPaginatedPosts } from "../lib/posts";

import Pagination from "../components/common/Pagination";

  

export const metadata: Metadata = {

title: "Home | Obslog",

description: "blog main page",

};

  

const page = async ({

searchParams,

}: {

searchParams: Promise<{ page?: string }>;

}) => {

const resolvedSearchParams = await searchParams;

  

const currentPage = Number(resolvedSearchParams.page) || 1;

console.log("currnetPage:", currentPage);

  

const paginatedPosts = getPaginatedPosts(currentPage, 4);

  

return (

<section className="border border-amber-400">

<ul className="flex flex-col gap-4">

{paginatedPosts.posts.map((post) => (

<li key={post.slug} className="border border-amber-400">

<PostCard post={post} />

</li>

))}

</ul>

<Pagination totalPages={paginatedPosts.totalPages} />

</section>

);

};

  

export default page;
```

