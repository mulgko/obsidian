---
title: Role
subject: "[[Frontend]]"
reference: ""
date: 2026-02-20 13:50
description: ""
tags:
  - role
  - aria
series: ""
seriesOrder:
published: false
---

# Role

role은 HTML요소가 어떤 역할을 하는지 스크린 리더에게 알려주는 ARIA 속성입니다.
div는 의미가 없지만, role="listbox"를 붙이면, 이 div는 선택 가능한 목록이에요 라고 알려줄 수 있습니다.

| Role       | 의미                        |
| ---------- | ------------------------- |
| listbox    | 선택 가능한 옵션 목록              |
| option     | listbox 안의 개별 항목          |
| dialog     | 모달/다이얼로그                  |
| navigation | 네비게이션 영역                  |
| button     | 버튼 (이미 <button> 태그이면 불필요) |
