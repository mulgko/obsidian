---
title: Modern Frontend Terms
subject: "[[Dev Note]]"
reference: "[[Component Folder Pattern]]"
date: 2026-02-13 11:58
description: 현대 프론트엔드 개발의 핵심 개념 (Barrel, Monorepo, Co-location)
tags:
  - barrel-pattern
  - monorepo
  - turborepo
  - co-location
  - 개념
series: ""
seriesOrder:
published: false
---

# 현대 프론트엔드 핵심 용어 정리

[[Component Folder Pattern]] 등 최신 아키텍처 문서에서 자주 언급되는 핵심 개념들에 대한 상세 설명입니다.

## 1. Barrel Pattern (배럴 패턴)

### 개념
여러 모듈을 하나의 `index.ts` 파일에서 다시 내보내는(re-export) 패턴입니다. 술통(Barrel)에 여러 물건을 담아두듯, 폴더 내부의 여러 파일을 하나의 진입점으로 묶어서 제공합니다.

### 예시
```typescript
// features/auth/index.ts
export * from './LoginButton';
export * from './UserProfile';
export * from './useAuth';

// 사용하는 곳
import { LoginButton, useAuth } from '@/features/auth'; // 깔끔함
```

### 장점
- **캡슐화**: 내부 구조를 숨기고 깔끔한 외부 인터페이스 제공.
- **Import 경로 단축**: `../../features/auth/LoginButton` 대신 `../../features/auth` 사용 가능.

### 단점 및 주의사항 (2026 트렌드)
- **Tree Shaking 문제**: 일부 번들러 설정에서는 `index.ts`의 모든 모듈을 로드하여 불필요한 코드가 번들에 포함될 수 있음.
- **순환 참조 (Circular Dependency)**: 서로 다른 모듈이 서로의 `index.ts`를 참조하면 런타임 에러 발생 위험 높음.
- **성능 저하**: 개발 서버 구동 시 수천 개의 Barrel 파일을 읽느라 속도가 느려질 수 있음.
> 💡 **Tip**: 라이브러리 제작 시에는 유용하지만, 앱 내부 비즈니스 로직에는 과도한 사용을 지양하는 추세입니다.

---

## 2. Monorepo & Turborepo (모노레포와 터보레포)

### Monorepo (모노레포)
여러 개의 프로젝트(앱, 패키지)를 **하나의 Git 저장소(Repository)** 에서 관리하는 전략입니다.
- 예: 웹사이트(`apps/web`), 관리자 페이지(`apps/admin`), 공유 UI 라이브러리(`packages/ui`), 공통 유틸(`packages/utils`)을 한 곳에 둠.

### Turborepo (터보레포)
Vercel에서 만든 **모노레포용 고성능 빌드 시스템**입니다.
- **캐싱(Caching)**: 한 번 빌드한 내용은 캐시하여, 변경되지 않은 부분은 0초 만에 빌드 완료 처리.
- **병렬 실행**: 의존성을 분석하여 가능한 작업을 동시에 실행.
- **원격 캐싱**: 팀원 A가 빌드한 캐시를 팀원 B도 사용하여 빌드 속도 비약적 향상.

### 사용하는 이유
- **코드 공유 용이**: UI 컴포넌트나 비즈니스 로직을 패키지로 분리하여 여러 앱에서 즉시 사용.
- **단일 버전 관리**: 모든 앱이 동일한 버전의 라이브러리나 설정을 공유하기 쉬움.
- **DevOps 효율화**: CI/CD 파이프라인 하나로 모든 프로젝트 검증 가능.

---

## 3. Co-location (위치 통합/근접 배치)

### 개념
**"변하는 것들은 함께 두라"** 는 소프트웨어 공학 원칙을 프론트엔드 파일 구조에 적용한 것입니다. 특정 기능을 수정할 때 필요한 모든 파일(로직, 스타일, 테스트, 문서)을 **물리적으로 같은 폴더**에 배치합니다.

### 과거 vs 현재
- **과거 (Layered)**: 기술 종류별로 분리
  ```text
  src/
    components/Button.tsx
    styles/Button.css
    tests/Button.test.ts
    hooks/useButton.ts
  ```
  -> 버튼 하나 수정하려면 4개의 폴더를 오가야 함.

- **현재 (Co-located)**: 기능/컴포넌트별로 통합
  ```text
  src/components/Button/
    Button.tsx
    Button.css
    Button.test.ts
    useButton.ts
  ```
  -> 버튼 수정 시 이 폴더 하나만 보면 됨.

### Next.js App Router와 Co-location
Next.js의 App Router는 이 원칙을 강력하게 지원합니다.
- `app/dashboard/` 폴더 안에 `page.tsx`뿐만 아니라, 해당 페이지에서만 쓰이는 `Chart.tsx`, `helper.ts` 등을 함께 둬도 라우팅에 영향을 주지 않습니다. (과거 `pages/` 폴더는 모든 파일을 라우트로 인식했음)

### 장점
- **유지보수성**: 관련 코드가 한곳에 있어 문맥 파악이 빠름.
- **삭제 용이성**: 기능 삭제 시 해당 폴더만 지우면 됨 (좀비 코드 방지).
- **AI 친화적**: AI에게 폴더 하나만 던져주면 완벽한 컨텍스트 이해 가능.
