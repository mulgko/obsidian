
- 참고: [[Next.js Rendering]]
- 태그: #rendering #ssg #next #개념 #isr

빌드할 때 미리 생성하기


- 동작원리: npm run build 할 때 서버가 데이터를 가져와서 미리 HTML을 생성, 서버에 물리적으로 파일 생성
- 장점: - 사용자가 접속하면 이미 만들어진 HTML을 그냥 던져주기만 하면 끝. CDN(Content Delivery Network)**에 파일을 복사해둘 수 있어 전 세계 어디서든 가깝게 접속 가능합니다.