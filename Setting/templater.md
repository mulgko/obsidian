<%*
// 최소 입력만 받기
const title = await tp.system.prompt("글 제목을 입력하세요");
const tags = await tp.system.prompt("태그를 입력하세요 (쉼표로 구분)");

// 파일명에서 사용 불가능한 문자 제거
const safeFileName = title.replace(/[\\/:*?"<>|]/g, '-');
await tp.file.rename(safeFileName);
%>---
title: "<% title %>"
subject: ""
reference: ""
date: "<% tp.date.now("YYYY-MM-DD HH:mm") %>"
description: ""
tags: [<% tags %>]
series: ""
seriesOrder:
published: false
---

# <% title %>