---
title: Typeof JS vs TS
subject: "[[Frontend]]"
reference: ""
date: 2026-02-20 10:48
description: JS typeof(런타임 값 판별)와 TS typeof(컴파일타임 타입 추출)의 차이 및 실무 패턴
tags:
  - typeof
  - js
  - ts
  - 개념
series: ""
seriesOrder:
published: false
---

# Typeof JS vs TS

`typeof`는 JS와 TS에서 이름은 같지만 작동 시점과 목적이 완전히 다르다.

---

## 1. JS의 `typeof` — 런타임 연산자

코드가 실제로 **실행될 때** 변수에 담긴 **값의 종류**를 문자열로 반환한다.

```js
typeof "hello"    // "string"
typeof 42         // "number"
typeof true       // "boolean"
typeof undefined  // "undefined"
typeof null       // "object" ← 유명한 버그, null인데 object를 반환
typeof []         // "object" ← 배열도 object로 나옴, 주의
typeof {}         // "object"
typeof function(){} // "function"
```

반환 가능한 값은 `"string"`, `"number"`, `"boolean"`, `"object"`, `"undefined"`, `"function"`, `"symbol"`, `"bigint"` 총 8가지다.

**언제 쓰나?** 외부 API나 사용자 입력처럼, **실행 중에 들어오는 값**이 어떤 타입인지 모를 때 방어 코드로 쓴다.

```ts
function process(value: string | number) {
  if (typeof value === "string") {
    // 이 블록 안에서는 value가 string임을 JS가 런타임에 보장
    console.log(value.toUpperCase());
  } else {
    // 이 블록 안에서는 value가 number임이 확정
    console.log(value.toFixed(2));
  }
}
```

---

## 2. TS의 `typeof` — 컴파일타임 연산자

코드가 **실행되기 전, 빌드 단계**에서 이미 존재하는 **변수나 객체의 구조**를 그대로 복사해 타입을 만든다. 컴파일 후에는 흔적도 없이 사라져서 번들 크기에 영향을 주지 않는다.

```ts
const user = {
  name: "dk",
  age: 27,
};

type User = typeof user;
// 결과: { name: string; age: number; }
```

직접 `type User = { name: string; age: number; }`를 쓰지 않아도 되고, `user` 객체가 바뀌면 `User` 타입도 자동으로 반영된다.

---

## 3. 핵심 비교

| | JS `typeof` | TS `typeof` |
|---|---|---|
| **작동 시점** | 런타임 (실행 중) | 컴파일타임 (빌드 전) |
| **반환값** | 8가지 문자열 중 하나 | 변수 구조를 복사한 타입 |
| **주 용도** | 실행 중 값 검증 | 타입 중복 정의 방지 |
| **번들 포함** | ✅ JS 파일에 포함 | ❌ 컴파일 후 사라짐 |

---

## 4. 실무 패턴

### 데이터에서 타입 추출 (Single Source of Truth)

상수 배열에서 타입을 뽑아 쓰는 가장 흔한 패턴이다. 배열 항목이 바뀌면 타입도 자동으로 바뀐다.

```ts
export const THEMES = [
  { id: "light", label: "Light" },
  { id: "dark",  label: "Dark"  },
  { id: "glass", label: "Glass" },
] as const;

// THEMES 배열에서 자동으로 타입 생성
type Theme   = (typeof THEMES)[number];        // { id: "light", label: "Light" } | ...
type ThemeId = Theme["id"];                    // "light" | "dark" | "glass"
```

`as const`를 붙이면 TS가 `string` 대신 `"light"` 같은 리터럴 타입으로 정확하게 추론한다.

### 외부 라이브러리 객체에서 타입 따오기

라이브러리가 타입을 export하지 않을 때, 객체에서 직접 추출할 수 있다.

```ts
import { router } from "some-library";

type RouterType = typeof router;

function navigate(config: RouterType) { ... }
```

---

## 5. 결론

두 연산자는 역할이 다르므로 함께 쓴다.

- 실행 중 외부 값 검증이 필요하면 → **JS `typeof`** (런타임 가드)
- 이미 있는 변수에서 타입을 재사용하려면 → **TS `typeof`** (타입 추출, DRY 원칙)
