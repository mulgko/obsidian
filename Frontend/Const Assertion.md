---
title: Const Assertion
subject: "[[Frontend]]"
reference: ""
date: 2026-02-20 09:19
description: ""
tags:
  - const
  - ts
  - const-assertion
  - readonly
  - 개념
  - typeof
series: ""
seriesOrder:
published: false
---

# Const Assertion


```typescript
export const THEMES = [

{ id: "light", label: "Light" },

{ id: "dark", label: "Dark" },

{ id: "glass", label: "Glassmorphism" },

] as const;
```

1. Readonly로 잠금: 일반적으로 JS의 const는 변수 재할당만 막을뿐, 배열 내부의 값을 바꾸는 것은 허용 합니다. 하지만 as const를 붙이면 배열 내부의 값도 절대 수정할 수 없는 '읽기 전용'(Readonly)상태가 됩니다.

2. 타입을 구체적인 '값'으로 고정 (Literal Type): 가장 핵심적인 이유
     - as const가 없을 때 TS는 id를 단순히 string으로 취급합니다.
     - as const가 있을 때 TS는 id를 그 글자 외에는 값으로 취급하질 않습니다. 

3. 어떻게 활용할까
```typescript
// 1. 배열 선언
export const THEMES = [...] as const;

// 2. 배열에서 ID 값들만 뽑아서 타입으로 만들기 (자동 추출)
export type ThemeId = (typeof THEMES)[number]["id"];

// 결과: type ThemeId = "light" | "dark" | "glass"

```
이렇게 하면, 나중에 테마가 추가되어도 배열만 수정하면, 연결된 타입까지 자동으로 업데이트 되기 때문에 코드 관리가 편해집니다. 


