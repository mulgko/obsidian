---
title: Mount
subject: ""
reference: ""
date: 2026-03-05 17:05
description: ""
tags:
  - frontend
  - mount
series: ""
seriesOrder:
published: false
---
 
# Mount

# 다크모드 새로고침 시 테마 전환 모션이 보이는 문제

  

## 문제

다크모드 상태에서 새로고침하면, 잠깐 라이트 모드(해 아이콘)가 보였다가 다크 모드(달 아이콘)로 전환되는 모션이 노출됨.

  

## 원인

`ThemeToggle` 컴포넌트가 초기값을 `isDark = false`로 시작하기 때문.

```

1. 컴포넌트 렌더링 → isDark = false → 해 아이콘 표시

2. useEffect 실행 → data-theme 읽어서 isDark = true로 변경

3. 달 아이콘으로 전환되는 모션이 노출됨

```

`useEffect`는 브라우저에서 hydration이 끝난 후 실행되기 때문에,
그 사이에 잘못된 초기 상태가 화면에 잠깐 보이게 됨.

  

## 해결

`mounted` 상태를 추가해서, 테마를 확인하기 전까지는 토글을 렌더링하지 않도록 처리.

```tsx

const [mounted, setMounted] = useState(false);

  

useEffect(() => {

const applied = document.documentElement.getAttribute("data-theme");

setIsDark(applied === "dark");

setMounted(true);

}, []);

  

if (!mounted) return <div className="w-11 h-6" />;

```

  
- `mounted`가 `false`인 동안은 같은 크기의 빈 `div`를 렌더링

- 레이아웃이 튀지 않으면서 잘못된 아이콘도 노출되지 않음

- `useEffect` 실행 후 `mounted = true`가 되면 올바른 테마 상태로 토글을 렌더링