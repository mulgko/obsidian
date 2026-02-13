---
title: Obsidian Submodule 선택적 발행 가이드
subject: "[[Obslog]]"
reference: "[[ROADMAP]], [[Obsidian-Blog-Structure]]"
date: 2026-02-13
description: Obsidian 저장소를 submodule로 넣고 원하는 파일만 블로그에 발행하는 방법
tags:
  - obsidian
  - blog
  - submodule
  - publishing
---

# Obsidian Submodule 선택적 발행 가이드

Obsidian 저장소를 **submodule**로 넣되, **원하는 파일만** 블로그에 발행하는 방법입니다.

---

## 1. 프로젝트 구조

### Before (Obsidian 저장소)

```bash
Obsidian/
├── Dev Note/
│   ├── Component Folder Pattern.md
│   ├── Frontend Architecture.md      # 발행하고 싶음 ✅
│   └── Modern Frontend Terms.md
├── Projects/
│   ├── Obslog/
│   └── Personal/
├── Setting/
│   └── obsidian + git.md
└── Daily/
    └── 2024-01-15.md
```

### After (Obslog 블로그 프로젝트)

```bash
obslog/                           # 블로그 프로젝트
├── content/                      # Git submodule
│   └── obsidian/                 # Obsidian 저장소 (submodule)
│       ├── Dev Note/
│       ├── Projects/
│       ├── Setting/
│       └── Daily/
│
├── src/
│   ├── app/
│   ├── components/
│   └── lib/
│       └── posts.ts              # ⭐ 여기서 필터링!
│
└── next.config.ts
```

---

## 2. 선택적 발행 전략 (3가지 방법)

### 방법 A: `published: true` 필드 (가장 추천) ⭐

**개념:**
- 발행하고 싶은 파일에만 `published: true` 추가
- `lib/posts.ts`에서 `published: true`인 것만 읽기

**장점:**
- 가장 명시적이고 확실함
- 파일 위치 상관없이 발행 제어
- 실수로 발행될 위험 없음

**Obsidian 파일 예시:**

```markdown
<!-- Dev Note/Frontend Architecture.md -->
---
title: "Frontend Architecture 가이드"
tags: [frontend, architecture]
published: true                    # ⭐ 이것만 추가!
createdAt: "2024-01-15"
---

# 내용...
```

```markdown
<!-- Dev Note/Component Folder Pattern.md -->
---
title: "Component Folder Pattern"
tags: [frontend, pattern]
published: false                   # ❌ 발행 안 함
createdAt: "2024-01-10"
---

# 내용...
```

**`lib/posts.ts` 구현:**

```typescript
import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';

// Obsidian 저장소 경로 (submodule)
const CONTENT_DIR = path.join(process.cwd(), 'content', 'obsidian');

export function getAllPosts() {
  const allFiles = getAllMarkdownFiles(CONTENT_DIR);

  const posts = allFiles
    .map(filePath => {
      const content = fs.readFileSync(filePath, 'utf-8');
      const { data, content: markdown } = matter(content);

      return {
        slug: generateSlug(filePath),
        frontmatter: data,
        content: markdown,
      };
    })
    // ⭐ published: true만 필터링
    .filter(post => post.frontmatter.published === true)
    // 날짜 내림차순
    .sort((a, b) =>
      new Date(b.frontmatter.createdAt) - new Date(a.frontmatter.createdAt)
    );

  return posts;
}

// 재귀적으로 모든 .md 파일 찾기
function getAllMarkdownFiles(dir: string): string[] {
  const files: string[] = [];

  const items = fs.readdirSync(dir);

  for (const item of items) {
    const fullPath = path.join(dir, item);
    const stat = fs.statSync(fullPath);

    if (stat.isDirectory()) {
      // .obsidian, .git 등 무시
      if (!item.startsWith('.')) {
        files.push(...getAllMarkdownFiles(fullPath));
      }
    } else if (item.endsWith('.md')) {
      files.push(fullPath);
    }
  }

  return files;
}

// 파일 경로에서 slug 생성
function generateSlug(filePath: string): string {
  // content/obsidian/Dev Note/Frontend Architecture.md
  // → dev-note-frontend-architecture

  const relativePath = path.relative(CONTENT_DIR, filePath);
  const slug = relativePath
    .replace(/\.md$/, '')
    .replace(/\//g, '-')
    .replace(/\s+/g, '-')
    .toLowerCase();

  return slug;
}
```

---

### 방법 B: 특정 폴더만 읽기

**개념:**
- `lib/posts.ts`에서 특정 폴더만 스캔
- 예: `Dev Note/`, `Projects/` 폴더만 읽고, `Setting/`, `Daily/` 무시

**장점:**
- frontmatter 없어도 됨
- 폴더 구조로 명확히 구분

**단점:**
- 폴더 안에 있으면 무조건 발행됨
- 세밀한 제어 어려움

**`lib/posts.ts` 구현:**

```typescript
const ALLOWED_FOLDERS = ['Dev Note', 'Projects'];

export function getAllPosts() {
  const allFiles = getAllMarkdownFiles(CONTENT_DIR);

  const posts = allFiles
    .filter(filePath => {
      // ⭐ 특정 폴더에 있는 파일만
      return ALLOWED_FOLDERS.some(folder =>
        filePath.includes(path.join(CONTENT_DIR, folder))
      );
    })
    .map(filePath => {
      // ... 파일 읽기
    });

  return posts;
}
```

---

### 방법 C: 태그 기반 필터링

**개념:**
- 특정 태그가 있는 것만 발행
- 예: `blog` 태그가 있으면 발행

**장점:**
- 태그로 간단히 제어
- 여러 태그 조합 가능

**단점:**
- 태그 빠뜨리면 발행 안 됨
- 실수 가능성 높음

**Obsidian 파일 예시:**

```markdown
---
title: "Frontend Architecture"
tags: [frontend, architecture, blog]    # ⭐ blog 태그 추가
---
```

**`lib/posts.ts` 구현:**

```typescript
export function getAllPosts() {
  const allFiles = getAllMarkdownFiles(CONTENT_DIR);

  const posts = allFiles
    .map(filePath => {
      const content = fs.readFileSync(filePath, 'utf-8');
      const { data } = matter(content);
      return { filePath, frontmatter: data };
    })
    // ⭐ 'blog' 태그가 있는 것만
    .filter(post =>
      post.frontmatter.tags?.includes('blog')
    );

  return posts;
}
```

---

## 3. 최종 추천: Hybrid 방식 ⭐

**방법 A (published) + 방법 C (태그) 조합**

```typescript
export function getAllPosts() {
  const allFiles = getAllMarkdownFiles(CONTENT_DIR);

  const posts = allFiles
    .map(filePath => {
      const content = fs.readFileSync(filePath, 'utf-8');
      const { data, content: markdown } = matter(content);

      return {
        slug: generateSlug(filePath),
        frontmatter: data,
        content: markdown,
      };
    })
    .filter(post => {
      const { frontmatter } = post;

      // ⭐ 1차: published: true 필수
      if (frontmatter.published !== true) return false;

      // ⭐ 2차: tags 배열이 있어야 함
      if (!frontmatter.tags || !Array.isArray(frontmatter.tags)) return false;

      // ⭐ 3차: (선택) 최소 1개 태그 필요
      if (frontmatter.tags.length === 0) return false;

      return true;
    })
    .sort((a, b) =>
      new Date(b.frontmatter.createdAt) - new Date(a.frontmatter.createdAt)
    );

  return posts;
}
```

**Obsidian 파일 예시:**

```markdown
<!-- ✅ 발행됨 -->
---
title: "Frontend Architecture"
tags: [frontend, architecture]
published: true
createdAt: "2024-01-15"
---

<!-- ❌ 발행 안 됨 (published: false) -->
---
title: "개인 메모"
tags: [memo]
published: false
---

<!-- ❌ 발행 안 됨 (published 없음) -->
---
title: "Daily Note"
tags: [daily]
---

<!-- ❌ 발행 안 됨 (tags 없음) -->
---
title: "To Do"
published: true
---
```

---

## 4. Submodule 설정

### 4-1. Submodule 추가

```bash
cd obslog                        # 블로그 프로젝트 루트
mkdir -p content
cd content

# Obsidian 저장소를 submodule로 추가
git submodule add https://github.com/your-username/obsidian.git obsidian

# 또는 로컬 경로
git submodule add ../Obsidian obsidian
```

### 4-2. .gitmodules 확인

```ini
[submodule "content/obsidian"]
    path = content/obsidian
    url = https://github.com/your-username/obsidian.git
```

### 4-3. 서브모듈 업데이트

```bash
# 서브모듈 최신화
cd content/obsidian
git pull origin main

# 또는 프로젝트 루트에서
git submodule update --remote
```

---

## 5. Obsidian에서 발행 워크플로우

### 템플릿 설정 (Templater)

```markdown
<!-- Templates/blog-post.md -->
---
title: "<% tp.file.cursor(1) %>"
description: "<% tp.file.cursor(2) %>"
tags: []
published: false              # ⭐ 기본값: 발행 안 함
createdAt: "2026-02-13"
updatedAt: "2026-02-13"
---

# <% tp.file.cursor(1) %>

<% tp.file.cursor(3) %>
```

### 발행 프로세스

```
1. Obsidian에서 글 작성
   ↓
2. 발행 결정
   ├─ published: false → true 변경
   └─ tags 추가
   ↓
3. Git commit & push (Obsidian)
   ↓
4. 블로그 프로젝트에서 submodule 업데이트
   ├─ git submodule update --remote
   └─ git add content/obsidian
   ↓
5. Vercel 자동 빌드 → 블로그 반영
```

---

## 6. 고급: 발행 상태 관리

### Dataview로 발행 가능한 글 보기

```dataviewjs
// published: true이지만 태그 없는 글
TABLE title, tags, published
FROM ""
WHERE published = true AND (!tags OR length(tags) = 0)

// 발행 예정 (published: false)
TABLE title, tags
FROM ""
WHERE published = false AND tags AND length(tags) > 0
SORT file.mtime DESC
LIMIT 10
```

### Obsidian 명령어 단축키

**빠른 발행 토글 (QuickAdd 플러그인)**

```javascript
// published: false ↔ true 토글
module.exports = async (params) => {
  const file = app.workspace.getActiveFile();
  if (!file) return;

  const content = await app.vault.read(file);
  const newContent = content.replace(
    /published: (true|false)/,
    (match, p1) => `published: ${p1 === 'true' ? 'false' : 'true'}`
  );

  await app.vault.modify(file, newContent);
};
```

---

## 7. 실전 예시

### 현재 Obsidian 구조

```bash
Obsidian/
├── Dev Note/
│   ├── Component Folder Pattern.md    (published: false)
│   ├── Frontend Architecture.md       (published: true) ✅
│   └── Modern Frontend Terms.md       (published: true) ✅
│
├── Projects/
│   ├── Obslog/
│   │   └── ROADMAP.md                 (published: true) ✅
│   └── Personal/
│       └── secret.md                  (published: false)
│
├── Setting/
│   └── obsidian + git.md              (published: false)
│
└── Daily/
    └── 2024-01-15.md                  (published: false)
```

### 블로그에 보이는 글

```bash
블로그 포스트 목록:
├── Frontend Architecture          (Dev Note/)
├── Modern Frontend Terms          (Dev Note/)
└── ROADMAP                        (Projects/Obslog/)

# Setting/, Daily/는 아예 안 보임
```

---

## 8. 추가 최적화

### 8-1. 캐싱 (성능)

```typescript
// lib/posts.ts
let cachedPosts: Post[] | null = null;

export function getAllPosts(forceRefresh = false) {
  if (!forceRefresh && cachedPosts) {
    return cachedPosts;
  }

  // ... 파일 읽기
  cachedPosts = posts;
  return posts;
}
```

### 8-2. 에러 처리

```typescript
function getAllMarkdownFiles(dir: string): string[] {
  try {
    const items = fs.readdirSync(dir);
    // ...
  } catch (error) {
    console.warn(`Failed to read directory: ${dir}`, error);
    return [];
  }
}
```

### 8-3. Frontmatter 검증

```typescript
function validateFrontmatter(data: any): boolean {
  if (!data.title || typeof data.title !== 'string') return false;
  if (!data.createdAt) return false;
  if (!Array.isArray(data.tags)) return false;

  return true;
}

export function getAllPosts() {
  // ...
  .filter(post => validateFrontmatter(post.frontmatter))
}
```

---

## 9. 체크리스트

- [ ] Obsidian 저장소를 submodule로 추가
- [ ] `lib/posts.ts`에서 재귀 탐색 구현
- [ ] `published: true` 필터링 구현
- [ ] Obsidian 템플릿에 `published: false` 기본값 설정
- [ ] 샘플 포스트 3개에 `published: true` 추가
- [ ] `npm run dev`로 블로그 확인
- [ ] Dataview 쿼리로 발행 가능한 글 확인
- [ ] Git submodule 업데이트 테스트
- [ ] Vercel 배포 테스트

---

## 10. FAQ

**Q1. published 필드를 빠뜨리면?**
- A: 기본값이 없으므로 발행 안 됨 (안전)

**Q2. 폴더 구조를 바꾸면?**
- A: slug 생성 로직만 수정하면 됨

**Q3. 파일명이 한글이면?**
- A: slug 생성 시 한글 → 영문 변환 로직 추가
  ```typescript
  function slugify(text: string): string {
    return text
      .replace(/[ㄱ-ㅎ|ㅏ-ㅣ|가-힣]/g, '') // 한글 제거
      .replace(/\s+/g, '-')
      .toLowerCase();
  }
  ```

**Q4. 여러 Obsidian vault를 연동하려면?**
- A: submodule 여러 개 추가
  ```bash
  content/
  ├── obsidian-work/
  └── obsidian-personal/
  ```

---

## 11. 결론

**최종 추천 방식:**

```
Obsidian Submodule + published: true 필터링 + 태그 검증
```

**장점:**
1. ✅ 심플함 - `published: true`만 추가
2. ✅ 안전함 - 기본값이 발행 안 됨
3. ✅ 유연함 - 폴더 구조 변경해도 OK
4. ✅ 명시적 - 어떤 글이 발행될지 명확

**시작 단계:**
1. Submodule 추가
2. `lib/posts.ts` 구현 (재귀 + 필터링)
3. Obsidian 템플릿 설정
4. 3개 글에 `published: true` 추가
5. 블로그 확인

이 방식으로 **Obsidian 전체를 연동하되, 원하는 글만 심플하게 발행**할 수 있습니다! 🚀
