---
title: Some
subject: ""
reference: ""
date: 2026-02-25 16:04
description: ""
tags:
  - frontend
  - some
  - every
series: ""
seriesOrder:
published: false
---

# Some

.som() 메서드는 하나라도 만족하면 true 값을 반환한다.

```typescript
["react", "next"].some((tag) => tag === "react") // → true

["react", "next"].some((tag) => tag === "vue") // → false
```


.every() 메서드와 비교하면 차이가 더욱 명확합니다.

```typescript
["react", "next"].every((tag) => tag === "react") // → false

["react", "next"].every((tag) => tag === "vue") // → false
```
