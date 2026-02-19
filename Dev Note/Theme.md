---
title: Theme
subject: "[[Dev Note]]"
reference: "[[LocalStorage vs Cookie]]"
date: 2026-02-19 23:10
description: ""
tags:
  - theme
  - cookie
series: ""
seriesOrder:
published: false
---


# 테마 시스템 구현 원리 및 상세 로직

  

이 문서는 `ThemeDropdown.tsx` 컴포넌트와 `layout.tsx`를 연동하여 구현된 테마 전환 시스템의 4가지 핵심 로직을 설명합니다.


---

  

### 1. `useEffect`: 서버와 클라이언트의 상태 동기화

  

서버 사이드 렌더링(SSR) 환경에서 발생할 수 있는 테마 불일치 문제를 해결합니다.

  

```typescript

useEffect(() => {

// 서버에서 이미 data-theme을 설정했으므로 DOM에서 읽어 상태 동기화

const applied = document.documentElement.getAttribute(

"data-theme",

) as ThemeId | null;

if (applied && themes.some((t) => t.id === applied)) {

setCurrent(applied);

}

}, []);

```

  

- **배경**: `layout.tsx`에서 서버가 쿠키를 읽어 `<html>` 태그에 `data-theme`을 미리 적용해둡니다.

- **역할**: 클라이언트 컴포넌트가 마운트될 때 실제 HTML에 적용된 속성값을 읽어와서 React의 `current` 상태를 동기화합니다. 이를 통해 UI(체크 표시 등)가 실제 적용된 테마와 항상 일치하게 됩니다.

  

---

  

### 2. 사용자 경험(UX)을 위한 편의 기능

  

드롭다운 메뉴의 인터랙션을 자연스럽게 처리합니다.

  

```typescript

const handleClickOutside = (e: MouseEvent) => {

if (ref.current && !ref.current.contains(e.target as Node)) {

setOpen(false);

}

};

const handleEscape = (e: KeyboardEvent) => {

if (e.key === "Escape") setOpen(false);

};

```

  

- **바깥 클릭 시 닫기**: 드롭다운이 열린 상태에서 메뉴 외부의 영역을 클릭하면 `ref`를 감지하여 메뉴를 닫습니다.

- **ESC 키 바인딩**: 사용자가 키보드의 ESC 키를 누르면 즉시 메뉴가 닫히도록 접근성과 편의성을 제공합니다.

  

---

  

### 3. `handleSelect`: 테마 변경 및 저장

  

사용자가 테마를 선택했을 때의 복합적인 상태 변화를 처리합니다.

  

```typescript

const handleSelect = (id: ThemeId) => {

setCurrent(id); // 1. React UI 상태 변경

document.documentElement.setAttribute("data-theme", id); // 2. 즉시 CSS 적용

document.cookie = `theme=${id};path=/;max-age=31536000;SameSite=Lax`; // 3. 브라우저 저장

setOpen(false); // 4. 드롭다운 닫기

};

```

  

- **즉시성**: `data-theme` 속성을 변경하여 별도의 새로고침 없이 CSS 변수(`var(--bg-color)` 등)가 즉시 바뀌도록 합니다.

- **지속성**: `document.cookie`에 테마 값을 저장하여, 이후 페이지를 새로고침하거나 브라우저를 다시 열어도 서버가 이 값을 읽어 테마를 유지할 수 있게 합니다.

  

---

  

### 4. 전체적인 흐름 정리 (Synergy)

  

서버(Next.js)와 클라이언트(React)가 협력하여 '테마 깜빡임' 없는 경험을 제공합니다.

  

1. **사용자 선택**: 사용자가 테마를 변경하면 **쿠키**가 업데이트됩니다.

2. **서버 렌더링**: 다음 페이지 요청 시, 서버(`layout.tsx`)가 쿠키를 읽어 **HTML 생성 단계**에서 이미 테마가 적용된 클래스/속성을 주입하여 보냅니다.

3. **깜빡임 방지**: 브라우저는 자바스크립트가 로드되기 전부터 올바른 테마 색상으로 페이지를 그리므로 속칭 '화이트 플래시' 현상이 발생하지 않습니다.

4. **최종 동기화**: `ThemeDropdown` 컴포넌트가 로드되면서 현재 테마 상태를 UI에 반영하며 준비를 마칩니다.