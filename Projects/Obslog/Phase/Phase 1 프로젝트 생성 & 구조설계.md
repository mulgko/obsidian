---
title: "Phase 1"
subject: "[[phase]]"
reference: ""
date: "2026-02-12 09:17"
description: "프로젝트 폴더 구조 및 필요한 요소 설계"
tags:
  - folder
  - structure
  - phase
  - folderPattern
series: ""
seriesOrder:
published: false
---

## 필요한 것들: pages / components / utility / contents

- [x] `create-next-app`으로 프로젝트 생성
- [x] `npm run dev`로 기본 페이지 확인
- [x] 생성된 파일 하나씩 열어서 역할 파악
- [x] 폴더 구조를 **직접 설계**해봄 (종이에 그려봐도 좋음)
- [x] Route Group `(blog)`, `(admin)` 개념 이해
- [x] **Co-location**, **Barrel Pattern** 개념 이해
- [x] [[Folder pattern]] 문서 읽고 최종 구조 파악 ✨
- [x] 필요한 패키지 설치
- [x] Git 초기 커밋

```text

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
