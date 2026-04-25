---
title: DangerouslySetInnerHTML
subject: ""
reference: ""
date: 2026-03-16 11:42
description: ""
tags:
  - frontend
  - dangerously-setInner-html
series: ""
seriesOrder:
published: false
---

# DangerouslySetInnerHTML

dangerouslySetInnerHTML은 HTML 문자열을 그대로 DOM에 삽입하는 React의 prop인데, 이름"dangerously"가 붙은 이유가  바로 XSS(Cross-Site Scripting) 취약점 때문

만약 마크다운 내용에 이런 게 들어있다면?

```typescript
<script>document.cookie를 탈취하는 코드</script>
```

그게 그대로 실행될 수 있어요. 그래서 React가 이름을 일부러 무섭게 지어서 "너 지금 위험한 짓 하는 거 알지?" 라고 경고 