---
title: Next.js AppRouter에서의 작동방식
subject: "[[Rendering]]"
reference:
date: 2026-02-12 16:03
description: ""
tags:
  - next
  - 개념
  - rsc
---
Next.js App Router에서는 이 모든 것을 **서버 컴포넌트(RSC)**라는 개념으로 통합했습니다.

- 이제 우리가 따로 "이건 SSG로 할게"라고 선언하지 않아도, Next.js는 똑똑하게 판단합니다.
- 데이터를 fetching 하는 함수에 
    
    ```
    cache: 'force-cache'
    ```
    
    를 쓰면 **SSG**가 되고,
- ```
    cache: 'no-store'
    ```
    
    를 쓰거나 
    
    ```
    cookies()
    ```
    
     같은 함수를 호출하면 자동으로 **SSR**로 동작합니다.
- ```
    "use client"
    ```
    
    를 쓰면 그 컴포넌트와 그 하위는 **CSR** 방식으로 작동하며 브라우저에서 '상호작용'을 담당합니다.