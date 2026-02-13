---
title: "Frontend design pattern"
subject: "[[Dev Note]]"
reference: ""
date: "2026-02-12 15:42"
description: "프론트엔드 디자인 패턴 가이드"
tags:
  - design
  - pattern
  - frontend
  - 개념
series: ""
seriesOrder:
published: false
---

# 디자인 패턴

반복적으로 발생하는 문제 상황에 대한 해결책을 제시하는 일반적인 설계 템플릿. 생성, 구조, 행위 세 가지 범주로 나뉘어진다.

![Frontend Design Patterns Comparison](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F61af5cdf-2a56-4ae8-ad09-eb703c26989d_1280x1755.jpeg)

## 1. MV* 패턴의 진화 (전통적 아키텍처)

프론트엔드 초기에는 UI 로직과 데이터 로직을 분리하는 것이 가장 큰 과제였습니다.

### MVC (Model-View-Controller)
- **Model**: 데이터와 비즈니스 로직.
- **View**: 사용자에게 보여지는 UI.
- **Controller**: 사용자 입력을 받아 Model을 업데이트하고 View를 제어.
- **특징**: View와 Model 사이의 의존성이 높고, 프로젝트가 커질수록 Controller가 비대해지는 'Massive Controller' 문제가 발생함.

### MVP (Model-View-Presenter)
- MVC의 Controller 대신 **Presenter**가 등장.
- Presenter는 View와 Model 사이의 완전한 중재자 역할을 하며, View는 오직 Presenter를 통해서만 데이터를 주고받음.
- **특징**: View와 Model의 의존성을 끊었지만, View와 Presenter 사이의 의존성이 강해지는 경향이 있음.

### MVVM (Model-View-ViewModel)
- 핵심은 **Data Binding**. 
- **ViewModel**: View를 위한 상태를 추적하고 업데이트. 데이터 바인딩을 통해 ViewModel의 상태가 변하면 View가 자동으로 업데이트됨.
- **특징**: 유지보수가 쉽고 코드가 간결해짐. React, Vue 등 현대 프레임워크의 근간이 됨.

---

## 2. 현대적 상태 관리 및 데이터 흐름

앱이 복잡해지면서 "상태(State)"를 어떻게 관리할 것인가가 중요해졌습니다.

### FLUX Architecture (Redux, Zustand)
- **Unidirectional Data Flow**: Action -> Dispatcher -> Store -> View의 단방향 흐름.
- **특징**: 데이터 흐름이 예측 가능해져 디버깅이 쉬워짐. 대규모 앱의 복잡한 상태 관리에 적합.

![Flux/Redux Unidirectional Data Flow](https://redux.js.org/assets/images/ReduxDataFlowDiagram-49fa8c3968371d9ef6f2a1486bd40a26.gif)

### Atomic State Management (Recoil, Jotai)
- 상태를 가장 작은 단위인 **Atom**으로 쪼갬.
- **특징**: 필요한 컴포넌트만 해당 Atom을 구독하므로 성능 최적화가 쉽고, 직관적인 사용이 가능함.

---

## 3. 서버 상태와 통신 (Data Fetching)

단순한 클라이언트 상태 관리를 넘어, 서버 데이터와의 동기화가 핵심이 되었습니다.

### React Query / SWR (Server State)
- 클라이언트 상태(UI 토글 등)와 **서버 상태**(API 응답 데이터)를 엄격히 분리.
- **특징**: 캐싱, 업데이트, 에러 핸들링, 로딩 상태 관리 등을 선언적으로 처리. 프론트엔드 개발의 패러다임을 바꿈.

### GraphQL (Declarative Fetching)
- REST API의 한계(Over-fetching, Under-fetching)를 해결.
- **특징**: 클라이언트가 필요한 데이터의 구조를 직접 정의하여 요청. 타입 안전성이 높고 백엔드 의존성을 줄임.
