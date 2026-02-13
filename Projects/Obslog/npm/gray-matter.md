---
title: gray-matter
subject: "[[npm]]"
reference:
date: 2026-02-12 09:17
description: 마크다운 파일의 메타데이터를 추출하는 라이브러리
tags:
  - npm
  - metadata
---

gray-matter는 마크다운 파일 내에서 생성된 메타데이터를 추출해 자바스크립트 객체로 변환해주는 라이브러리다. 

### 1. Frontmatter란?

마크다운 파일 최상단에 

```
---
```

로 감싸진 영역을 말합니다. 보통 YAML 형식을 파일의 메타데이터를 저장할 때 사용합니다.

markdown

---

title: "나의 첫 블로그 포스트"

date: "2024-02-10"

tags: ["Next.js", "React"]

---
# 여기서부터는 마크다운 본문입니다.

이 내용은 블로그의 본문으로 렌더링됩니다.



### 2.  gray-matter의 역할

이 패키지는 위의 마크다운 파일을 읽어서 아래와 같이 **데이터(data)**와 **본문(content)**으로 깔끔하게 분리해줍니다.

const matter = require('gray-matter');
const fileContent = fs.readFileSync('post.md', 'utf8');

const { data, content } = matter(fileContent);

console.log(data); 
// 결과: { title: "나의 첫 블로그 포스트", date: "2024-02-10", tags: ["Next.js", "React"] }

console.log(content);
// 결과: "\n# 여기서부터는 마크다운 본문입니다.\n이 내용은 블로그의 본문으로 렌더링됩니다."


### 3. DB의 컬럼 역할 

일반적인 웹 서비스는 글 제목, 작성일, 태그 정보를 데이터 베이스(My SQL, Mongo Db 등)의 각 컬럼에 저장합니다. 하지만 지금 만들고 있는 블로그는 별도의 데이터베이스를 제공하지 않는 파일 기반 블로그입니다. 따라서 다음과 같은 매칭이 일어납니다. 

전통적인 DB: posts 테이블의 title, date, tags 컬럼
파일 기반 블로그: 마크다운 파일 내 Frontmatter의 title, date, tags 속성

즉, gray-matter를 사용하면 별도의 DB 서버 없이도 **마크다운 파일 안에 데이터베이스처럼 구조화된 정보**를 담고 관리할 수 있게 되는 것입니다.


--- 
'---'
title,
date,
tags
이 사이를 메타데이터로 만들어서 객체로 담아주는 역할
'---'

--- 
