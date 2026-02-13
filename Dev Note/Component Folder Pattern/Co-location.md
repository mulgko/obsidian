---
title: Co-location
subject: "[[Barrel Pattern]]"
reference:
date: 2026-02-13 12:00
description: Co-location(위치 통합)의 개념과 Next.js App Router에서의 적용
tags:
  - co-location
  - architecture
  - frontend
  - 개념
---

# Co-location (위치 통합/근접 배치)

## 개념
**"변하는 것들은 함께 두라"** 는 소프트웨어 공학 원칙을 프론트엔드 파일 구조에 적용한 것입니다. 특정 기능을 수정할 때 필요한 모든 파일(로직, 스타일, 테스트, 문서)을 **물리적으로 같은 폴더**에 배치합니다.

## 과거 vs 현재

### 과거 (Layered)
기술 종류별로 분리
```
src/
  components/Button.tsx
  styles/Button.css
  tests/Button.test.ts
  hooks/useButton.ts
```
-> 버튼 하나 수정하려면 4개의 폴더를 오가야 함.

### 현재 (Co-located)
기능/컴포넌트별로 통합
```
src/components/Button/
  Button.tsx
  Button.css
  Button.test.ts
  useButton.ts
```
-> 버튼 수정 시 이 폴더 하나만 보면 됨.

## Next.js App Router와 Co-location
Next.js의 App Router는 이 원칙을 강력하게 지원합니다.
- `app/dashboard/` 폴더 안에 `page.tsx`뿐만 아니라, 해당 페이지에서만 쓰이는 `Chart.tsx`, `helper.ts` 등을 함께 둬도 라우팅에 영향을 주지 않습니다. (과거 `pages/` 폴더는 모든 파일을 라우트로 인식했음)

## 장점
- **유지보수성**: 관련 코드가 한곳에 있어 문맥 파악이 빠름.
- **삭제 용이성**: 기능 삭제 시 해당 폴더만 지우면 됨 (좀비 코드 방지).
- **AI 친화적**: AI에게 폴더 하나만 던져주면 완벽한 컨텍스트 이해 가능.
