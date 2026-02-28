---
title: Promise
subject: "[[Javascript Async]]"
reference: ""
date: 2026-02-26 09:59
description: ""
tags:
  - async
  - promise
  - frontend
  - await
series: ""
seriesOrder:
published: false
---

# Promise

**"나중에 결과를 줄게"라는 약속 객체.**

`fetch()`, DB 조회 같은 비동기 작업은 결과를 즉시 줄 수 없다. 그래서 JavaScript는 결과 대신 **Promise**를 먼저 반환한다.

```ts
const result = fetch("https://api.example.com/posts");
console.log(result); // Promise { <pending> } ← 아직 데이터 없음
```

---

## Promise의 3가지 상태

| 상태        | 의미                    |
| ----------- | ----------------------- |
| `pending`   | 아직 결과를 기다리는 중 |
| `fulfilled` | 성공적으로 완료됨       |
| `rejected`  | 실패함 (에러 발생)      |

---

## Promise를 직접 만들면

```ts
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("완료!"); // 1초 후 fulfilled 상태로 전환
  }, 1000);
});
```

---

## 결과를 받으려면?

Promise는 약속 객체이므로, 실제 값을 꺼내려면 `.then()`을 쓰거나 `await`을 써야 한다.

```ts
// .then() 방식 (구식)
fetch("https://api.example.com/posts")
  .then((response) => response.json())
  .then((data) => console.log(data));

// await 방식 (현대적, 권장)
const response = await fetch("https://api.example.com/posts");
const data = await response.json();
console.log(data);
```

→ `await`은 [03-await.md](03-await.md) 참고

---

## 왜 Promise가 필요한가?

JavaScript는 싱글 스레드 — 한 번에 하나의 작업만 처리한다. 오래 걸리는 작업(네트워크, DB)을 동기로 처리하면 그동안 다른 모든 작업이 멈춘다.

Promise + 비동기 처리를 통해 **기다리는 동안 다른 작업을 계속** 할 수 있다.
