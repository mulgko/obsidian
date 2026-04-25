---
title: remark, remark-html
subject: "[[npm]]"
reference: ""
date: 2026-02-12 09:17
description: 마크다운을 변환하는 라이브러리
tags:
  - npm
  - remark
series: ""
seriesOrder:
published: false
---

상세역할: 마크다운 문법등을 브라우저가 이해할 수 있도록 HTML로 변환

 마크다운 텍스트
 
↓                                                                                                              

  remark. ← 마크다운을 읽어서 "트리 구조(AST)"로 파싱

↓

AST (Abstract Syntax Tree)

↓

remark-html  ← AST를 HTML 문자열로 변환하는 "플러그인"

↓

HTML 문자열

remark 자체는 마크다운을 파싱하는 엔진이야. HTML을 만들지는 않음
remark-html은 remark에 붙이는 플러그인으로, AST → HTML 변환을 담당

플러그인 구조인 이유는? remark가 범용 파서라서 HTML 말고도 다른 형태로 변환할 수 있게 설계
예: AST → PDF,
AST → Slack 메시지 등)

```typescript
import { remark } from "remark";                                                 import remarkHtml from "remark-html";                                                                            
export async function markdownToHtml(content: string): Promise<string> {                                           
	const result = await remark().use(remarkHtml).process(content); 
	// 메서드 체이닝 패턴, 각 단계가 뭘 반환하는지 머릿속으로 따라가는 습관                          // remark() => Processor 객체 
	// .use(reamrkHtml) => 플러그인 추가된 Processor 객체                        
	// .process(content) => Promise<VFile> (VFile = remark의 결과 객체)
	// result.toString() => HTML 문자열
	return result.toString();                                                                                    
}     
```

