<%*
// 1. 입력창 시퀀스
const title = await tp.system.prompt("글 제목을 입력하세요");
const subject = await tp.system.prompt("주제(연결할 노트)를 입력하세요");
const ref = await tp.system.prompt("참고(연결할 노트)를 입력하세요");
const desc = await tp.system.prompt("글 요약(description)을 적어주세요");
const tags = await tp.system.prompt("태그를 입력하세요 (쉼표로 구분)");


// 2. 파일 이름 자동 변경
await tp.file.rename(title);
%>---
title: "<% title %>"
subject: "[[<% subject %>]]"
reference: "[[<% ref %>]]"
date: "<% tp.date.now("YYYY-MM-DD HH:mm") %>"
description: "<% desc %>"
tags: [<% tags %>]
published: false

---

# 본문 시작