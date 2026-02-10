
- 주제: [[개념정리]]

- 참고 [[Create Blog]]

- 태그: #serverComponent #clientComponent #next #useState #useEffect #onClick #onChange


|            | Server                        | Client                  |
| ---------- | ----------------------------- | ----------------------- |
| 실행위치       | 서버(빌드 시점, 요청 시)               | 브라우저(사용자 PC)            |
| 기본값        | App Router의 모든 컴포넌트           | 파일 최상단에 "use Client" 명시 |
| 장점         | 보안 우수, 번들 크기 감소, (속도 최상)      | 사용자와 상호 작용 가능           |
| 파일읽기       | 가능 (fs, readFilSync 등)        | 불가능                     |
| React Hook | 사용 불가 (useState, useEffect 등) | 사용 가능                   |
| 이벤트 리스너    | 사용 불가 (onClick, onChange 등)   | 사용 가능                   |
