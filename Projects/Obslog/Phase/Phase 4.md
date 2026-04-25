---
title: Phase 4
subject: ""
reference: ""
date: 2026-03-03 17:11
description: ""
tags:
  - phase
series: ""
seriesOrder:
published: false
---

# Phase 4

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