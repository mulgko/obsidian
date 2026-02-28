---
title: NextJS Async
subject: "[[Javascript Async]]"
reference: ""
date: 2026-02-26 10:07
description: ""
tags:
  - next
  - async
series: ""
seriesOrder:
published: false
---

# NextJS Async


왜 서버 컴포넌트에 **async**를 붙일까?
1. JS의 비동기 실행 모델: JS는 싱글 스레드입니다. 한 번에 하나의 작업만 처리할 수 있게 설계되어 있습니다. DB 조회, API 호출 같은 I/O 작업은 필연적으로 오래 걸리게 되는데, 그동안 JS가 멈춰서 기다리면 다른 요청을 처리하지 못하게 되죠. 그래서 비동기 패턴이 나오게 된 것입니다. 
```typescript
// 동기 (blocking) — 파일 다 읽을 때까지 프로그램 멈춤
const data = fs.readFileSync("file.txt");

// 비동기 (non-blocking) — 기다리는 동안 다른 작업 가능
const data = await fs.promises.readFile("file.txt");
```

2. NextJS의 서버 컴포넌트와 async: AppRouter의 서버 컴포넌트는 React가 서버에서 렌더링합니다. 이때 React는 컴포넌트 함수를 호출합니다.  async 함수는 항상 Promise를 반환합니다. Next는 서버 컴포넌트가 Promise를 반환하면 자동으로 await해줍니다. 이게 AppRouter의 핵심 기능 중 하나입니다.
```typescript
// async 없는 버전
const page = () => {
const posts = getAllPosts(); // 동기라 OK
return <div>{posts.map(...)}</div>;
// React가 리턴값을 즉시 받음 ✅
};

// async 있는 버전
const page = async () => {
const posts = await fetch("https://api.example.com/posts"); // 비동기
return <div>{posts.map(...)}</div>;
// React가 Promise를 받아서 resolve될 때까지 기다림 ✅
};
```

3. async 없는 함수는 왜 당장 작동하는가?: 
```typescript
const page = () => {               // async 없음
  const posts = getAllPosts();      // fs.readFileSync — 동기
  return <div>...</div>;           // 즉시 JSX 반환
};
```
getAllPosts가 동기 함수라 이미 실행이 끝난 값을 바로 반환합니다. 그러니까 asnyc, await이 없어도 문제가 없죠. 하지만 이렇게 바꾸면요?

```typescript
const page = () => {                    // async 없음 ❌
  const posts = fetch("/api/posts");    // Promise 반환
  return <div>{posts.map(...)}</div>;   // posts는 Promise 객체, map 없음 💥
};
```
fetch()는 Promise를 반환하는데, await 없이 쓰면, posts가 실제 데이터가 아니라 Promise 객체 그 자체가 됩니다. .map()이 없으니 에러가 발생 되겠죠. 

정리하자면, 결국 나중에는 비동기로 처리해야할 일이 생기게 됩니다. 이를 대비하기 위해서 좋은 습관 + NextJS의 표준 패턴이라고 볼 수 있겠습니다.