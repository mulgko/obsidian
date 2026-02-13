---
title: tailwind-typography
subject: "[[npm]]"
reference:
date: 2026-02-12 09:17
description: Tailwind CSS를 위한 타이포그래피 플러그인
tags:
  - npm
  - tailwind
  - typography
published: false
---

npm i @tailwindcss/typography



### @tailwindcss/typography

(디자인 치트키)

- **상세 역할:** 
    
    ```
    prose
    ```
    
    라는 마법의 클래스 하나로 마크다운 HTML에 완벽한 타이포그래피 스타일을 입혀줍니다.
- **왜 사용하나요?** 마크다운을 HTML로 변환하면 아무 스타일 없는 쌩(!) 텍스트가 나옵니다. 
    
    ```
    <h2>
    ```
    
    는 작고, 여백은 하나도 없죠. 이걸 일일이 테두리 주고 여백 주려면 노가다가 심합니다.
- **특징:** 
    
    ```
    <article className="prose">
    ```
    
    라고 감싸기만 하면, 제목 크기, 줄 간격, 리스트 점 모양 등이 마치 잡지처럼 예쁘게 자동 세팅됩니다. (개인적으로 가장 좋아하는 도구입니다.)