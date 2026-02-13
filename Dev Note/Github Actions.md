---
title: Github Actions
subject: "[[Dev Note]]"
reference: ""
date: 2026-02-12 15:45
description: 깃허브 액션에 대해서
tags:
  - 개념
  - githubActions
  - git
---

Github Actions는 Github에서 제공하는 CI/CD플랫폼. 코드 저장소에서 push, pr 등이 발생했을 때 내가 정해놓은 작업들이 자동으로 실행되는 것. 

- Workflow: 가장 상위 개념으로 자동화된 전체 프로세스 .github/workflow/.yml 형식의 파일로 저장 
- Events: 워크플로우를 실행시키는 트리거 역할. 코드를 push했을 때, 매일 자정, 혹은 수동으로 버튼을 눌렀을 때 등
- Jobs: 워크플로우 안에서 실행되는 독립적인 단위. 여러 작업이 병렬로 실행될 수도 있고, 순차적으로 실행될 수도 있습니다. 
- Steps: 하나의 작업 안에서 실행되는 개별 테스크. 명령어를 실행하거나 다른 사람들이 만들어놓은 Actions를 사용할 수도 있다. 
- Runners: 작업이 실제로 실행되는 서버. Github가 가상 컴퓨터(서버)를 빌려줍니다. 