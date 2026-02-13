---
title: Monorepo & Turborepo
subject: "[[Barrel Pattern]]"
reference:
date: 2026-02-13 12:00
description: 모노레포(Monorepo)와 터보레포(Turborepo)의 개념 및 장점
tags:
  - monorepo
  - turborepo
  - architecture
  - frontend
  - 개념
published: false
---

# Monorepo & Turborepo

## Monorepo (모노레포)
여러 개의 프로젝트(앱, 패키지)를 **하나의 Git 저장소(Repository)** 에서 관리하는 전략입니다.
- 예: 웹사이트(`apps/web`), 관리자 페이지(`apps/admin`), 공유 UI 라이브러리(`packages/ui`), 공통 유틸(`packages/utils`)을 한 곳에 둠.

## Turborepo (터보레포)
Vercel에서 만든 **모노레포용 고성능 빌드 시스템**입니다.
- **캐싱(Caching)**: 한 번 빌드한 내용은 캐시하여, 변경되지 않은 부분은 0초 만에 빌드 완료 처리.
- **병렬 실행**: 의존성을 분석하여 가능한 작업을 동시에 실행.
- **원격 캐싱**: 팀원 A가 빌드한 캐시를 팀원 B도 사용하여 빌드 속도 비약적 향상.

## 사용하는 이유
- **코드 공유 용이**: UI 컴포넌트나 비즈니스 로직을 패키지로 분리하여 여러 앱에서 즉시 사용.
- **단일 버전 관리**: 모든 앱이 동일한 버전의 라이브러리나 설정을 공유하기 쉬움.
- **DevOps 효율화**: CI/CD 파이프라인 하나로 모든 프로젝트 검증 가능.
