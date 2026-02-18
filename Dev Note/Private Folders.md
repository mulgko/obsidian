---
title: Private Folders
subject: "[[Dev Note/Dev Note|Dev Note]]"
reference: ""
date: 2026-02-18 16:22
description: ""
tags:
  - folder
  - private
  - next
  - co-location
  - routing
  - 개념
series: ""
seriesOrder:
published: false
---

# Private Folders

Next.js의 App Router에서 폴더 이름 앞에 언더스코어 (' _ ')를 붙이는 것은 Private Folder라고 불리는 기능입니다. 이 패턴을 사용하는 이유는 크게 다음과 같다.

1. 라우팅 제외 (Excluding from Routing): Next.js의 app 리엑토리 내에 있는 모든 폴더는 기본적으로 URL 경로(Route)가 됩니다.  하지만 폴더 이름 앞에 _ 를 붙이면, Next.js는 해당 폴더와 그 안의 모든 내용을 라우팅 시스템에서 제외합니다. ex) admin/posts/_ components 폴더는 /admin/posts/_ components 라는 URL로 접속할 수 없습니다. 
2. 컴포넌트 공동 배치 (Co-location): 라우트와 관련된 파일들 (ex: page.tsx, layout.tsx)과 해당 페이지에서만 사용하는 전용 컴포넌트들을 같은 폴더 내에 두고 싶을 때 사용합니다. 이렇게 하면 src/components 와 같은 공통 폴더에 모든 컴포넌트를 몰아넣지 않고, 해당 기능을 사용하는 위치 바로 옆에 내부 컴포넌트를 둘 수 있어 프로젝트 구조가 더 깔끔해집니다. 
3. 내부 파일 보호: 브라우저에서 직접 접근하면 안 되는 스타일 시트, 테스트 파일, 유틸리티 함수 등을 특정 경로 아래에 안전하게 보관하는 용도로 쓰입니다. 