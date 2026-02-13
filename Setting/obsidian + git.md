---
title: obsidian + git
subject: "[[setting]]"
reference: ""
date: 2026-02-12 09:17
description: Git과 Obsidian 연동 가이드
tags:
  - setting
  - git
  - obsidian
---

1. git / obsidian 설치
2. git에서 .gitignore 먼저 작성
```gitignore
# obsidian
.obsidian/cache
.obsidian/workspace
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.DS_Store
Thumbs.db
.obsidian/graph.json
```
3. terminal에서 obsidian vault 찾아서 git init
4. git remote add origin https://github.com/USERNAME/obsidian.git
5. 나머지는 commit / push / pull 기존 git처럼 사용하기.