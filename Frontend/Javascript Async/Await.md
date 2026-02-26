---
title: Await
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

# Await

**Promise가 완료될 때까지 기다렸다가 실제 값을 꺼내는 키워드.**

```ts
// await 없으면
const result = fetch("https://api.example.com/posts");
console.log(result); // Promise { <pending> } ← 약속 객체, 데이터 아님

// await 있으면
const result = await fetch("https://api.example.com/posts");
console.log(result); // Response { ... } ← 실제 응답 데이터
```

---

## 비유

```
// await 없이 주문
"피자 주문!" → 📦 상자(Promise)만 받음 (안에 피자 없음)

// await으로 주문
"피자 주문!" → ⏳ 기다림 → 🍕 피자가 도착하면 꺼냄
```

---

## 규칙: async 함수 안에서만 쓸 수 있다

```ts
// ❌ 에러
const Page = () => {
  const data = await fetch("..."); // SyntaxError
};

// ✅ OK
const Page = async () => {
  const data = await fetch("..."); // 정상 작동
};
```

`await`은 반드시 `async` 함수 안에서만 사용 가능하다. 항상 세트다.

→ `async`는 [02-async.md](02-async.md) 참고

---

## 여러 번 쓸 수 있다

```ts
const Page = async () => {
  const response = await fetch("https://api.example.com/posts"); // 1. 요청 기다림
  const data = await response.json(); // 2. JSON 파싱 기다림
  return data;
};
```

각각의 `await`마다 해당 Promise가 완료될 때까지 기다린다.

---

## 에러 처리

Promise가 실패(rejected)하면 `await`에서 에러가 던져진다. `try/catch`로 잡는다.

```ts
const Page = async () => {
  try {
    const data = await fetch("https://api.example.com/posts");
  } catch (error) {
    console.error("데이터 받기 실패:", error);
  }
};
```

---

## 요약

|             | 설명                                                                |
| ----------- | ------------------------------------------------------------------- |
| **Promise** | "나중에 결과 줄게"라는 약속 객체 → [01-promise.md](01-promise.md) |
| **async**   | "이 함수는 비동기 작업을 한다" 선언 → [02-async.md](02-async.md)  |
| **await**   | Promise가 끝날 때까지 기다렸다가 값을 꺼냄                          |
