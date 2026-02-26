---
title: Async
subject: "[[Javascript Async]]"
reference: ""
date: 2026-02-26 10:00
description: ""
tags:
  - async
  - promise
  - await
  - frontend
series: ""
seriesOrder:
published: false
---

# Async

**"이 함수는 비동기 작업을 한다"고 선언하는 키워드.**

함수 앞에 `async`를 붙이면 두 가지 일이 생긴다:

1. 함수 안에서 `await`을 쓸 수 있게 된다
2. 함수의 반환값이 자동으로 **Promise로 감싸진다**

```ts
// async 없는 일반 함수
const greet = () => {
  return "안녕"; // 문자열 반환
};

// async 함수
const greet = async () => {
  return "안녕"; // Promise<"안녕"> 반환
};
```

---

## async 함수는 항상 Promise를 반환한다

```ts
const result = greet(); // async 함수 호출
console.log(result); // Promise { '안녕' }
```

값을 꺼내려면 `await`이 필요하다.

```ts
const result = await greet();
console.log(result); // '안녕'
```

---

## Next.js 서버 컴포넌트에서 async

App Router의 서버 컴포넌트는 `async`를 지원한다. 컴포넌트가 Promise를 반환하면 Next.js가 알아서 기다렸다가 렌더링한다.

```tsx
// ✅ 권장 패턴
const Page = async () => {
  const posts = await fetch("https://api.example.com/posts");
  return <div>...</div>;
};
```

지금 당장 비동기 작업이 없어도, **서버 컴포넌트에서 데이터를 가져올 때는 `async`를 붙이는 습관**을 들이는 게 좋다. 나중에 비동기로 바꿀 때 자연스럽게 이어진다.

---

## async 없이 await을 쓰면?

```ts
// ❌ 에러: await is only valid in an async function
const Page = () => {
  const posts = await fetch("...");
};
```

`await`은 반드시 `async` 함수 안에서만 쓸 수 있다. 항상 세트다.

→ `await`은 [03-await.md](03-await.md) 참고
