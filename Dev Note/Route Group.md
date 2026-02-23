---
title: Route Group
subject: "[[Dev Note]]"
reference: "[[Phase 1 [프로젝트 생성 & 구조설계]|Phase 1 프로젝트 생성 & 구조설계]]"
date: 2026-02-12 15:47
description: Route Group에 대해서
tags:
  - 개념
series: ""
seriesOrder:
published: false
---
Route Group? (blog), (admin) 

괄호로 감싼 폴더는 URL에 포함되지 않는 것. 

```

src/app/(blog)/page.tsx → URL: / (blog가 URL에 안 나옴)

src/app/(admin)/admin/ → URL: /admin (admin만 나옴)

```


### 쓰는 이유

블로그 페이지와 어드민 페이지에 다른 레이아웃을 적용하기 위해서 사용합니다.
(blog)는 header가 있고, (admin)은 sidebar가 있는 형태