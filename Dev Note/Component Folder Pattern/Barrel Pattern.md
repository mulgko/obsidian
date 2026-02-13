---
title: Barrel Pattern
subject: "[[Component Folder Pattern]]"
reference:
date: 2026-02-13 12:00
description: 배럴 패턴(Barrel Pattern)의 개념, 장단점 및 주의사항
tags:
  - barrel-pattern
  - architecture
  - frontend
  - 개념
---

# Barrel Pattern (배럴 패턴)

## 개념
여러 모듈을 하나의 `index.ts` 파일에서 다시 내보내는(re-export) 패턴입니다. 술통(Barrel)에 여러 물건을 담아두듯, 폴더 내부의 여러 파일을 하나의 진입점으로 묶어서 제공합니다.

## 예시
```typescript
// features/auth/index.ts
export * from './LoginButton';
export * from './UserProfile';
export * from './useAuth';

// 사용하는 곳
import { LoginButton, useAuth } from '@/features/auth'; // 깔끔함
```

## 장점
- **캡슐화**: 내부 구조를 숨기고 깔끔한 외부 인터페이스 제공.
- **Import 경로 단축**: `../../features/auth/LoginButton` 대신 `../../features/auth` 사용 가능.

## 단점 및 주의사항 (2026 트렌드)
- **Tree Shaking 문제**: 일부 번들러 설정에서는 `index.ts`의 모든 모듈을 로드하여 불필요한 코드가 번들에 포함될 수 있음.
- **순환 참조 (Circular Dependency)**: 서로 다른 모듈이 서로의 `index.ts`를 참조하면 런타임 에러 발생 위험 높음.
- **성능 저하**: 개발 서버 구동 시 수천 개의 Barrel 파일을 읽느라 속도가 느려질 수 있음.

> 💡 **Tip**: 라이브러리 제작 시에는 유용하지만, 앱 내부 비즈니스 로직에는 과도한 사용을 지양하는 추세입니다.
