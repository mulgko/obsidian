
- 주제: [[개념정리]]

- 태그: #rendering #next #ssg #ssr #csr #개념 

Next.js는 여러가지 렌더링 방식을 제공합니다.  

- SSG: (Static Site Generation) - 정적 생성 / 빌드 시점에 미리 HTML을 만들어 놓습니다.
	- 내용이 별로 바뀌지 않는 블로그 포스트에 최적입니다. 
	- 로딩이 매우 빠릅니다. 
	- SEO가 완벽합니다.
- SSR: (Server Side Rendering) - 서버 렌더링 / 요청할 때마다 서버에서 HTML을 생성합니다. 
	- 실시간으로 데이터 생성이 필요할 때 사용합니다. 
- CSR: (Client Slde Rendering) - 클라이언트 렌더링 / 브라우저에서 JS로 렌더링합니다. 
	- 인터렉션이 많은 UI / 어드민, 댓글 등
