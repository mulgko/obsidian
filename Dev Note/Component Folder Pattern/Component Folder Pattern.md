---
title: Component Folder Pattern
subject: "[[Dev Note]]"
reference: "[[Frontend Architecture]]"
date: 2026-02-13 11:45
description: 컴포넌트 폴더 구조 패턴 및 배럴 익스포트 설명
tags:
  - architecture
  - frontend
  - component-structure
  - barrel-pattern
  - 개념
published: false
---

# Component Folder Pattern & Barrel Pattern

"컴포넌트마다 폴더를 만들고 index.ts 등을 두는 구조"는 **Component Folder Pattern** (또는 Feature Folder)과 **Barrel Export Pattern**이 결합된 형태입니다.

---

## 1. 컴포넌트 폴더 패턴 (Component Folder Pattern)

각 컴포넌트와 관련된 모든 파일(로직, 스타일, 테스트, 스토리북 등)을 하나의 폴더에 모아 관리하는 패턴입니다. **Co-location(위치 통합)** 원칙의 핵심입니다.

### 구조 예시
```
src/components/Button/
  ├── index.ts           # Barrel Export (외부 노출용)
  ├── Button.tsx         # 컴포넌트 본체
  ├── Button.styles.ts   # 스타일 (CSS, Tailwind 등)
  ├── Button.test.tsx    # 유닛 테스트
  ├── Button.stories.tsx # 스토리북
  └── useButton.ts       # (선택) 복잡한 로직 분리
```

### 장점
- **파일 찾기 쉬움**: 관련 파일이 한 곳에 모여있음
- **유지보수성 향상**: 컴포넌트 수정 시 관련 파일 한 번에 관리
- **독립성 강화**: 컴포넌트별 응집도 높아짐
- **삭제/이동 용이**: 폴더째 관리 가능

### 단점
- **폴더 depth 증가**: 구조가 깊어질 수 있음
- **간단한 컴포넌트에 과함**: 10줄 컴포넌트에 폴더 구조는 over-engineering

---

## 2. 배럴 익스포트 패턴 (Barrel Export Pattern)

폴더 내부의 복잡한 구조를 숨기고, 외부에서는 깔끔하게 import 할 수 있게 해주는 패턴입니다.

### `index.ts` 내용
```typescript
export { Button } from './Button';
export type { ButtonProps } from './Button';
```

### Import 비교
```typescript
// ❌ Barrel 미사용: 경로가 지저분하고 내부 구조가 드러남
import { Button } from '@/components/Button/Button';

// ✅ Barrel 사용: 깔끔하고 명확함 (캡슐화)
import { Button } from '@/components/Button';
```

### ⚠️ Barrel Pattern 주의사항 (2026년 업데이트)

```typescript
// ❌ 나쁜 예: 모든 것을 re-export
export * from './Button';
export * from './Input';
// → 트리쉐이킹 방해, 순환 참조 위험

// ✅ 좋은 예: 명시적 export
export { Button } from './Button';
export type { ButtonProps } from './Button';
// → 번들러가 최적화하기 쉬움
```

---

## 3. 다양한 컴포넌트 구조 방법론

### 방법 1: Flat Structure (평면 구조)
```
src/components/
  ├── Button.tsx
  ├── Input.tsx
  ├── Card.tsx
  └── Modal.tsx
```

**적합한 경우:**
- 소규모 프로젝트 (< 50 컴포넌트)
- 빠른 프로토타이핑
- 팀원 모두 주니어일 때

**장점:** 단순함, 빠른 파일 접근
**단점:** 파일 많아지면 관리 어려움, 관련 파일 분리됨

---

### 방법 2: Component Folder (기본 방식)
```
src/components/Button/
  ├── index.ts
  ├── Button.tsx
  ├── Button.styles.ts
  ├── Button.test.tsx
  └── Button.stories.tsx
```

**적합한 경우:**
- 중규모 프로젝트 (50-200 컴포넌트)
- 컴포넌트별 독립성이 중요할 때
- 테스트/스토리북 함께 관리

**장점:** Co-location, 독립성
**단점:** 폴더 depth 증가

---

### 방법 3: Atomic Design Pattern
```
src/components/
  ├── atoms/          # 가장 작은 단위 (Button, Input)
  │   └── Button/
  ├── molecules/      # 원자 조합 (SearchBar)
  │   └── SearchBar/
  ├── organisms/      # 복잡한 UI (Header, Footer)
  │   └── Header/
  └── templates/      # 페이지 레이아웃
      └── MainLayout/
```

**적합한 경우:**
- 디자인 시스템 구축
- 재사용성이 매우 중요한 프로젝트
- 디자이너와 협업이 활발한 팀

**장점:** 명확한 계층 구조, 디자인 시스템과 궁합 좋음
**단점:** 분류 기준 애매할 수 있음, 작은 프로젝트엔 과함

---

### 방법 4: Feature-Sliced Design (FSD)
```
src/
  ├── shared/         # 공통 컴포넌트
  │   └── ui/
  │       └── Button/
  ├── entities/       # 비즈니스 엔티티
  │   └── user/
  ├── features/       # 기능 단위
  │   └── auth/
  └── pages/          # 페이지
      └── home/
```

**적합한 경우:**
- 대규모 프로젝트 (200+ 컴포넌트)
- 복잡한 비즈니스 로직
- 장기 운영 프로젝트

**장점:** 비즈니스 로직 분리 명확, 순환 참조 방지
**단점:** 학습 곡선 높음, 초기 설정 복잡

**참고:** [[Frontend Architecture]]

---

### 방법 5: Colocation with Next.js App Router (2026년 트렌드)
```
app/
  ├── (routes)/
  │   └── dashboard/
  │       ├── page.tsx
  │       ├── _components/     # Route 전용 컴포넌트
  │       │   └── DashboardChart/
  │       └── layout.tsx
  └── _components/             # 전역 공유 컴포넌트
      └── Button/
```

**특징:**
- Next.js 15+ App Router 권장 방식
- `_components`로 route별 컴포넌트 분리
- Server Components와 잘 맞음
- `_` prefix로 라우트에서 제외

---

## 4. 2026년 추천 방식

### ✅ 추천 1: Hybrid Approach (하이브리드)

```typescript
src/
  ├── components/
  │   ├── ui/                  # UI 라이브러리 스타일 (Barrel 사용)
  │   │   ├── button/
  │   │   │   ├── index.ts     # ✅ Barrel 사용
  │   │   │   ├── Button.tsx
  │   │   │   └── Button.test.tsx
  │   │   └── input/
  │   │       └── index.ts
  │   └── features/            # 비즈니스 컴포넌트 (Barrel 미사용)
  │       ├── UserProfile.tsx  # ❌ index.ts 없음 (명시적 import)
  │       └── PostList.tsx
  └── lib/
```

**Import 예시:**
```typescript
// UI 컴포넌트: 깔끔한 import
import { Button, Input } from '@/components/ui/button';

// Feature 컴포넌트: 명시적 import
import { UserProfile } from '@/components/features/UserProfile';
```

**왜 이 방식?**
- UI 키트는 Barrel로 편의성 제공
- 비즈니스 로직은 명시적 경로로 순환 참조 방지
- 트리쉐이킹 효율적
- shadcn/ui 등 인기 라이브러리가 사용하는 방식

---

### ✅ 추천 2: Turborepo + Shared UI (Monorepo)

```
apps/
  ├── web/
  └── admin/
packages/
  └── ui/                      # 공유 UI 라이브러리
      ├── src/
      │   ├── button/
      │   │   ├── index.ts
      │   │   └── Button.tsx
      │   └── index.ts         # 전체 Barrel
      └── package.json
```

**2026년 대세:**
- Vercel Turborepo, Nx 사용 급증
- 여러 앱이 UI 패키지 공유
- 독립적 versioning 가능
- 팀 간 협업 효율적

---

### ✅ 추천 3: RSC 고려한 구조 (React Server Components)

```typescript
// app/dashboard/_components/UserCard/
├── UserCard.tsx              # 'use client' 표시 여부 명확히
├── UserCard.server.tsx       # Server Component
├── UserCard.client.tsx       # Client Component
└── index.ts                  # 필요시 Barrel
```

**2026년 핵심:**
- Server/Client Component 구분 명확히
- `.server.tsx`, `.client.tsx` 네이밍 컨벤션
- Server Component 기본, 필요시만 Client
- Streaming, Suspense 활용

---

## 5. 실무 추천 가이드

### 프로젝트 규모별 추천

| 프로젝트 규모 | 추천 방식 | Barrel 사용 | 예시 |
|--------------|----------|-----------|------|
| 소규모 (< 50 컴포넌트) | Flat + Folder | 최소화 | 개인 프로젝트, MVP |
| 중규모 (50-200) | Component Folder | UI만 사용 | 스타트업 서비스 |
| 대규모 (200+) | FSD or Monorepo | 패키지별 사용 | 엔터프라이즈 |

### 팀 경험별 추천

- **주니어 위주**: Component Folder (단순하고 명확)
- **시니어 위주**: FSD (복잡도 관리 가능)
- **디자인 시스템 팀**: Atomic Design + Monorepo

### 프레임워크별 추천

- **Next.js App Router**: Colocation with `_components`
- **Remix**: Route-based colocation
- **React (Vite)**: Hybrid Approach
- **라이브러리 개발**: Monorepo + Barrel Pattern

---

## 6. 핵심 원칙 (2026년 기준)

1. **Co-location 우선** (관련 파일은 가까이)
2. **Barrel은 선택적** (UI 라이브러리에만)
3. **명시적 > 암시적** (경로 명확히)
4. **Server Component 우선** (RSC 활용)
5. **순환 참조 방지** (명시적 import로 해결)

---

## 7. 최종 추천 구조

**2026년 기준 가장 균형 잡힌 방식:**

```
src/
  ├── components/
  │   ├── ui/              # Barrel 사용 ✅
  │   │   └── button/
  │   │       └── index.ts
  │   └── features/        # Barrel 미사용 ❌
  │       └── UserProfile.tsx
  ├── lib/
  └── app/                 # Next.js App Router
      └── (routes)/
          └── _components/ # Route별 컴포넌트
```

---

## 더 알아보기

- [[Frontend Architecture]] - FSD, Clean Architecture
- **FSD 공식 문서**: https://feature-sliced.design/
- **Vercel Turborepo**: https://turbo.build/
- **shadcn/ui**: 2026년 가장 인기 있는 Component 구조 참고
- **React Server Components**: Next.js 15 공식 문서
