---
title: stringify
subject: "[[Dev Note/Dev Note|Dev Note]]"
reference: "[[Phase 3 [마크다운 포스트]]]"
date: 2026-02-23 14:30
description: ""
tags:
  - matter
  - stringify
  - 개념
series: ""
seriesOrder:
published: false
---
# Markdown CRUD: savePost() 구현 흐름

마크다운 기반 블로그에서 포스트를 저장하거나 수정하는 과정은 **데이터를 문자열로 직렬화(Serialization)**하고 **파일로 기록**하는 과정입니다.

  

## 1. savePost() 함수 구현

`savePost`는 객체 형태의 데이터를 마크다운 파일 형식으로 변환하여 저장합니다.

  

```typescript

export function savePost(post: Post): void {

// 1. frontmatter를 YAML 문자열로 변환 (데이터 + 본문 -> 문자열)

const fileContents = matter.stringify(post.content, post.frontmatter);

// 결과물 예시:

// ---

// title: "Next.js 입문"

// date: "2024-01-01"

// tags:

// - javascript

// - react

// published: true

// ---

//

// 본문 내용...

  

// 2. 파일 경로 조립

const fullPath = path.join(postDirectory, `${post.slug}.md`);

  

// 3. 파일 쓰기 (문자열 -> 파일)

fs.writeFileSync(fullPath, fileContents, "utf8");

}

```

  

---

  

## 2. 핵심 메서드 이해

  

### matter.stringify()

`gray-matter` 라이브러리에서 제공하는 `matter()`의 반대 기능을 수행합니다.

- **`matter()`**: 파일 내용(문자열) → `data`(frontmatter) + `content`(본문) 추출
- **`matter.stringify()`**: `data` + `content` → 마크다운 형식의 문자열로 병합

  

```javascript

const fileContents = matter.stringify(content, data);

```

  

### fs.writeFileSync()

Node.js의 파일 시스템 메서드로, 파일에 데이터를 기록합니다.

- **파일이 없을 때**: 지정된 이름으로 새 파일을 생성 (새 포스트 작성)
- **파일이 있을 때**: 기존 내용을 모두 지우고 새로 덮어씀 (기존 포스트 수정)

  

```javascript

fs.writeFileSync(fullPath, fileContents, "utf8");

```

  

---

  

## 3. 전체 데이터 흐름 (CRUD 그림)

  

마크다운 파일 제어의 전체적인 흐름은 다음과 같습니다.

  

| 동작 | 메서드 (읽기/변환) | 메서드 (쓰기/직렬화) |

| :------------------ | :------------------------------ | :------------------------------- |

| **파일 <-> 문자열** | `fs.readFileSync()` (파일 읽기) | `fs.writeFileSync()` (파일 쓰기) |

| **문자열 <-> 객체** | `matter()` (파싱) | `matter.stringify()` (직렬화) |

  

### 시각화

  

```

[ 읽기 흐름 ]

파일(.md) ──(readFileSync)──> 문자열 ──(matter)──> data / content 객체

  

[ 쓰기 흐름 ]

data / content 객체 ──(matter.stringify)──> 문자열 ──(writeFileSync)──> 파일(.md)

```

