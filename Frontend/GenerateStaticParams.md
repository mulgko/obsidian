---
title: GenerateStaticParams
subject: ""
reference: ""
date: 2026-03-17 09:22
description: ""
tags:
  - frontend
  - generate-static-params
  - ssg
series: ""
seriesOrder:
published: false
---

# GenerateStaticParams


지금 /posts/[slug] 페이지는 동적 라우팅이에요. [slug] 자리에 어떤 값이든 올 수 있죠.
빌드 시점에 Next.js 입장에서 생각해봐요:  

"나는 /posts/hello-world, /posts/react-guide 같은 페이지들을 미리 HTML로 만들어놔야 하는데... 어떤 slug들이 존재하는지 어떻게 알지?"

generateStaticParams가 없으면 Next.js는 이걸 모르는 상태예요. 그러면 두 가지 중 하나가 일어나요:
1. 빌드 시 해당 페이지를 아예 안 만든다 (접근 시 서버에서 그때그때 생성)
2. 또는 에러

블로그 포스트는 내용이 자주 안 바뀌니까 빌드 때 미리 HTML을 만들어두는 게(SSG) 성능상 훨씬 유리해요. 요청마다 파일을 읽고 변환할 필요가 없으니까요.

즉, generateStaticParams는 데이터를 가져오는 게 아니라 "이 slug들로 페이지를 미리 구워놔라" 고 Next.js에게 알려주는 역할이에요.

SSG는 "모든 포스트를 한 번에 다운로드"하는 게 아니에요.
빌드 시 서버에서 HTML 파일들을 미리 만들어놓는 거예요:

빌드 결과:
  /posts/hello-world.html   ← 미리 만들어둠
  /posts/react-guide.html   ← 미리 만들어둠
  /posts/next-js-tips.html  ← 미리 만들어둠

사용자가 /posts/hello-world 접속하면? 이미 만들어진 HTML 파일을 그냥 줘버리면 끝이에요.
SSG 없으면 (SSR처럼 동작하면)?
사용자 접속 → 서버에서 파일 읽기 → gray-matter 파싱
→ markdownToHtml 변환 → HTML 생성 → 응답
매 요청마다 이 과정을 반복해요.