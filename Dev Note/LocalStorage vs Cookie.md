---
title: LocalStorage & Cookie
subject: "[[Dev Note]]"
reference: "[[Theme]]"
date: 2026-02-19 17:48
description: ""
tags:
  - local-storage
  - cookie
  - 개념
series: ""
seriesOrder:
published: false
---

# LocalStorage & Cookie

obslog 블로그의 테마(Light/Dark/Glassmorphism)를 새로고침 후에도 유지하려고 했다.
처음엔 `localStorage`로 저장했더니 Next.js에서 hydration 오류가 발생했고,
이를 해결하면서 `localStorage`와 `Cookie`의 근본적인 차이를 이해하게 됐다.

---

## 핵심 차이 한 줄 요약

| | localStorage | Cookie |
|---|---|---|
| **서버 접근** | ❌ 불가 | ✅ 가능 (HTTP 요청에 자동 포함) |
| **브라우저 접근** | ✅ 가능 | ✅ 가능 |
| **용량** | ~5MB | ~4KB |
| **만료** | 직접 삭제 전까지 영구 | 만료일 설정 가능 |
| **자동 전송** | ❌ | ✅ 매 요청마다 서버로 전송 |

---

## 왜 hydration 오류가 발생했나

### localStorage를 썼을 때의 흐름

```
1. 브라우저가 서버에 페이지 요청
2. 서버: localStorage를 모름 → <html> 렌더링 (data-theme 없음)
3. 브라우저: HTML 수신 후 인라인 스크립트 실행
         → localStorage에서 "dark" 읽어서 data-theme="dark" 주입
4. React hydration 시작
         → 서버 HTML: data-theme 없음
         → 현재 DOM:  data-theme="dark"
         → "서버랑 달라!" → Hydration Error 💥
```

### Cookie를 썼을 때의 흐름

```
1. 브라우저가 서버에 페이지 요청
         → 요청 헤더에 cookie: theme=dark 자동 포함
2. 서버: 쿠키에서 "dark" 읽음
         → <html data-theme="dark"> 렌더링
3. 브라우저: HTML 수신
         → 이미 data-theme="dark"가 붙어있음
4. React hydration 시작
         → 서버 HTML: data-theme="dark"
         → 현재 DOM:  data-theme="dark"
         → "일치!" → 오류 없음 ✅
```

쿠키는 HTTP 요청 시 자동으로 서버에 전달되기 때문에,
서버가 첫 렌더링 시점부터 올바른 값을 알 수 있다.
이것이 핵심 차이다.

---

## localStorage 상세

### 특징
- 브라우저의 `window` 객체에 속함 → **서버에서 접근 불가**
- 같은 origin(도메인+포트) 내에서만 공유
- 탭/창을 닫아도 데이터 유지
- 명시적으로 삭제하기 전까지 영구 보존

### API
```js
// 저장
localStorage.setItem('key', 'value');

// 읽기
localStorage.getItem('key');

// 삭제
localStorage.removeItem('key');

// 전체 삭제
localStorage.clear();
```

### 적합한 사용 사례
- 서버 렌더링과 무관한 순수 클라이언트 데이터
- 장바구니 임시 저장 (비로그인 상태)
- 에디터 임시 저장 (자동저장 초안)
- 사용자 UI 상태 (접힌 사이드바, 정렬 기준 등)
- 오프라인 웹앱 데이터 캐시

### 주의사항
- SSR/SSG 환경에서 `typeof window !== 'undefined'` 체크 필요
- XSS 공격으로 탈취 가능 → 민감한 정보(토큰 등) 저장 금지
- 서버가 값을 알아야 하는 경우엔 부적합

---

## Cookie 상세

### 특징
- HTTP 요청마다 서버에 **자동으로 전송**됨
- 서버에서 `Set-Cookie` 헤더로 설정 가능
- 클라이언트에서 `document.cookie`로 읽기/쓰기 가능
- 만료일(`max-age`, `expires`) 설정 가능
- `HttpOnly` 플래그로 JS 접근 차단 가능 (XSS 방어)
- `Secure` 플래그로 HTTPS에서만 전송 가능

### API (클라이언트)
```js
// 저장
document.cookie = 'theme=dark;path=/;max-age=31536000;SameSite=Lax';

// 읽기 (파싱이 불편함)
document.cookie; // "theme=dark; user=dk"
```

### API (Next.js 서버 컴포넌트)
```ts
import { cookies } from 'next/headers';

const cookieStore = await cookies();
const theme = cookieStore.get('theme')?.value ?? 'light';
```

### 적합한 사용 사례
- **인증 토큰** (HttpOnly + Secure 설정으로 보안 강화)
- **서버 렌더링에 영향을 주는 사용자 설정** (테마, 언어, 지역)
- 세션 관리
- A/B 테스트 그룹 분류
- 트래킹/분석

### 주의사항
- 용량이 4KB로 제한 → 대용량 데이터 저장 부적합
- 매 요청마다 전송되므로 불필요하게 많은 데이터를 담으면 성능 저하
- `HttpOnly` 없이 저장한 토큰은 XSS에 취약

---

## 언제 뭘 쓸까?

```
서버가 이 값을 알아야 하는가?
├── Yes → Cookie
│         예: 테마, 언어 설정, 인증 토큰, 세션
│
└── No  → localStorage
          예: 에디터 초안, UI 상태, 오프라인 캐시
```

### 실전 예시

| 기능 | 선택 | 이유 |
|---|---|---|
| 로그인 토큰 | Cookie (HttpOnly) | 서버 인증 필요 + XSS 방어 |
| 테마 설정 | Cookie | SSR 시 서버가 알아야 hydration 오류 방지 |
| 언어 설정 | Cookie | SSR 시 서버가 알아야 올바른 언어로 렌더링 |
| 장바구니 (비로그인) | localStorage | 서버 불필요, 대용량 가능 |
| 에디터 자동저장 | localStorage | 서버 불필요, 빠른 읽기/쓰기 |
| 최근 본 상품 | localStorage | 서버 불필요, 용량 여유 |

---

## obslog에서의 적용

```ts
// layout.tsx (서버 컴포넌트)
const cookieStore = await cookies();
const theme = cookieStore.get("theme")?.value ?? "light";

return <html data-theme={theme}>...</html>;
```

```ts
// ThemeDropdown.tsx (클라이언트 컴포넌트)
const handleSelect = (id: ThemeId) => {
  document.documentElement.setAttribute("data-theme", id);
  document.cookie = `theme=${id};path=/;max-age=31536000;SameSite=Lax`;
};
```

서버가 쿠키를 읽어 첫 HTML 렌더링 시 `data-theme`을 주입하므로
클라이언트 hydration 시점에 DOM이 이미 일치 → 오류 없음.
