---
title: CSR
subject: "[[Rendering]]"
reference:
date: 2026-02-12 15:57
description: 재료만 보내고 요리는 사용자가 하기
tags:
  - rendering
  - csr
  - next
  - 개념
  - hydration
---

- 동작원리: 서버는 텅 빈 HTML 뼈대(

```
<body><div id="root"></div></body>
```

)

와 거대한 JS 파일만 브라우저로 보냅니다. 

브라우저가 이 JS 실행해서 직접 DOM을 그리고 화면을 채웁니다. 

- Hydration: Next.js같은 프레임워크는 순수 CSR보다는 서버에서 보낸 정적 HTML에 JS를 입혀서  '생명력'을 불어넣는 하이드레이션 과정을 거칩니다. 
- 장점: 한 번 로딩되면, 페이지 이동이 매우 부드럽습니다. 서버 부담이 없습니다. (계산은 사용자 PC가)
- 단점: JS파일이 너무 크면, 첫 화면이 뜰 때까지 시간이 오래 걸립니다. 검색 엔진(SEO) 로봇이 빈 HTML만 읽고 가버릴 수도 있습니다.





