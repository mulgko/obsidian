# 완벽 개발 환경 구축 가이드 (2026)

> 재사용 가능한 컴포넌트 시스템 + Figma MCP + Obsidian + CodeRabbit 연동

**작성일:** 2026-02-11
**프로젝트:** display-flex, obslog, Obsidian vault

---

## 📋 목차

1. [프로젝트 개요](#프로젝트-개요)
2. [재사용 가능한 컴포넌트 시스템](#재사용-가능한-컴포넌트-시스템)
3. [2026년 베스트 프랙티스](#2026년-베스트-프랙티스)
4. [Figma MCP 연동](#figma-mcp-연동)
5. [Obsidian + Git + CodeRabbit](#obsidian--git--coderabbit)
6. [트러블슈팅](#트러블슈팅)
7. [최종 워크플로우](#최종-워크플로우)

---

## 프로젝트 개요

### 목표

여러 프로젝트에서 재사용 가능한 React 컴포넌트 라이브러리 구축:
- **Tailwind CSS + tailwind-variants** 사용
- **Variant만 변경**하여 프로젝트별 디자인 적용
- **Figma → 코드 자동 변환**
- **Obsidian에서 블로그 글 작성** → CodeRabbit 리뷰 → 자동 배포

### 프로젝트 구조

```
/Users/gimdogyeong/projects/
├── display-flex/          # 영화 스트리밍 앱
├── obslog/                # Next.js 블로그
└── ip1/                   # 다른 프로젝트

/Users/gimdogyeong/Documents/Obsidian/  # Obsidian vault
├── Projects/
│   └── Obslog/            # 블로그 노트
└── .coderabbit.yaml       # CodeRabbit 설정
```

---

## 재사용 가능한 컴포넌트 시스템

### 문제점

기존 방식:
```typescript
// ❌ className 충돌
<button className={`${buttonClasses} ${className}`}>
  {children}
</button>
```

### 해결: tailwind-merge + tailwind-variants

#### 1. CN 유틸리티 함수

```typescript
// src/design-system/utils/cn.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

#### 2. 개선된 Button 컴포넌트

```typescript
// src/design-system/components/button/variants.ts
import { tv, type VariantProps } from "tailwind-variants";

export const buttonVariants = tv({
  base: "inline-flex items-center justify-center font-medium transition-colors",
  variants: {
    variant: {
      primary: "bg-orange-600 text-white hover:bg-orange-700",
      secondary: "bg-gray-800 text-white hover:bg-gray-700",
      outline: "border-2 border-orange-600 text-orange-600",
    },
    size: {
      sm: "h-9 px-3 text-sm rounded-md",
      md: "h-10 px-4 py-2 text-base rounded-md",
      lg: "h-11 px-8 text-lg rounded-md",
    },
  },
  defaultVariants: {
    variant: "primary",
    size: "md",
  },
});

export type ButtonVariants = VariantProps<typeof buttonVariants>;
```

```typescript
// src/design-system/components/button/Button.tsx
"use client";
import { forwardRef } from "react";
import { cn } from "@/design-system/utils/cn";
import { buttonVariants, type ButtonVariants } from "./variants";

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    ButtonVariants {}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, children, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size }), className)}
        {...props}
      >
        {children}
      </button>
    );
  }
);

Button.displayName = "Button";
```

#### 3. 사용 예시

```tsx
// 기본 사용
<Button variant="primary" size="md">
  클릭하세요
</Button>

// className으로 오버라이드 (충돌 없음!)
<Button
  variant="primary"
  className="bg-blue-500 hover:bg-blue-600"
>
  커스텀 버튼
</Button>
```

### 장점

- ✅ **타입 안전성**: `VariantProps`로 타입 자동 추론
- ✅ **className 충돌 없음**: `cn()` 함수가 자동 병합
- ✅ **확장 가능**: `extend`로 기본 컴포넌트 확장
- ✅ **어디서든 사용 가능**: design-system 폴더만 복사

---

## 2026년 베스트 프랙티스

### Monorepo + Turborepo

```
my-workspace/
├── apps/
│   ├── project-a/      # 영화 스트리밍
│   ├── project-b/      # 전자상거래
│   └── project-c/      # 블로그
├── packages/
│   └── ui/             # 공유 컴포넌트
├── turbo.json
└── pnpm-workspace.yaml
```

**장점:**
- ✅ Remote Caching (빌드 8배 빠름)
- ✅ 한 번 작성, 어디서나 사용
- ✅ 프로젝트별 테마만 변경

### Tailwind v4

**CSS-First Config:**
```css
/* globals.css */
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.65 0.21 50);    /* orange-600 */
  --color-secondary: oklch(0.3 0.02 250);  /* gray-800 */
  --size-md: 2.5rem;
  --radius: 0.5rem;
}
```

**장점:**
- ✅ 빌드 100배 빠름
- ✅ CSS Variables 자동 변환
- ✅ 프로젝트별 @theme 덮어쓰기

---

## Figma MCP 연동

### 개요

Figma 디자인을 Claude가 직접 읽고 코드로 자동 변환.

### 설정

#### 1. Figma Personal Access Token 발급

```
Figma → Settings → Account → Personal Access Tokens
→ Generate new token

Scopes:
✓ File content (read)
✓ File variables (read)
```

#### 2. Claude Code MCP 설정

```json
// ~/.claude/mcp.json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@figma/mcp-server"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "figd_your_token_here"
      }
    }
  }
}
```

#### 3. Figma 파일 구조화

```
Figma 파일 구조:
├── Design System
│   ├── Colors (Variables)
│   ├── Typography (Text Styles)
│   └── Spacing (Variables)
├── Components
│   ├── Button (Variants: Primary, Secondary, Outline)
│   └── Badge
└── Pages
```

**Best Practices:**
- ✅ Variables 사용 (색상, spacing, radius)
- ✅ Auto Layout 모든 컴포넌트에 적용
- ✅ 명확한 네이밍 (`Button/Primary`, `Badge/Default`)

#### 4. 사용법

```
Claude Code에서:

"이 Figma 버튼을 React + Tailwind + tailwind-variants로 만들어줘
https://figma.com/file/abc123/MyApp?node-id=123-456

요구사항:
- TypeScript
- forwardRef 사용
- variants로 primary, secondary, outline 구현"
```

**결과: 픽셀 퍼펙트 컴포넌트 자동 생성!**

### 비용

- ✅ **Figma Token**: 완전 무료
- ✅ **MCP 서버**: 무료
- ✅ **Claude Code**: Sonnet 4.5 사용

---

## GitHub Actions 자동화

### Submodule 자동 업데이트

#### 구조

```
github.com/username/obslog-content  (마크다운 레포)
├── .coderabbit.yaml
└── posts/

github.com/username/obslog  (Next.js 레포)
├── content/  (submodule → obslog-content)
└── src/
```

#### 워크플로우

```
1. Obsidian에서 글 작성 & push
   ↓
2. 🤖 obslog-content Actions: obslog에 신호
   ↓
3. 🤖 obslog Actions: submodule 업데이트
   ↓
4. Vercel 자동 배포
```

#### GitHub Actions 설정

**obslog-content/.github/workflows/notify-parent.yml:**
```yaml
name: Notify Parent Repo

on:
  push:
    branches: [main]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger parent repo update
        run: |
          curl -X POST \
            -H "Accept: application/vnd.github+json" \
            -H "Authorization: token ${{ secrets.PARENT_REPO_TOKEN }}" \
            https://api.github.com/repos/username/obslog/dispatches \
            -d '{"event_type":"update-submodule"}'
```

**obslog/.github/workflows/update-submodule.yml:**
```yaml
name: Update Content Submodule

on:
  repository_dispatch:
    types: [update-submodule]

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Update submodule
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git submodule update --remote content
          git add content
          git commit -m "chore: Update content submodule"
          git push
```

### 비용

| 항목 | 무료 한도 | 우리 사용량 |
|------|-----------|-------------|
| **GitHub Actions (Public)** | 무제한 | 월 5분 |
| **GitHub Actions (Private)** | 월 2,000분 | 월 5분 |
| **Personal Access Token** | 무료 | 무료 |

**결론: 완전 무료!** ✅

---

## Obsidian + Git + CodeRabbit

### 초기 목표

Obsidian에서 main push → CodeRabbit 자동 리뷰

### 문제 발견

**CodeRabbit은 PR 전용!**
- Main branch push는 리뷰 안 함
- `.coderabbit.yaml`의 `review_on_push: true` 작동 안 함

### 해결: 브랜치 + PR 방식

#### 1. .coderabbit.yaml 설정

```yaml
# /Users/gimdogyeong/Documents/Obsidian/.coderabbit.yaml

language: ko  # 한국어 리뷰
early_access: true
enable_free_tier: true

reviews:
  review_on_push: true
  auto_review: true
  level: "detailed"

  path_filters:
    - "**/*.md"
    - "Projects/**"

  path_ignores:
    - ".obsidian/**"
    - ".trash/**"
    - ".DS_Store"

markdown:
  validate_frontmatter: true
  check_links: true
  spell_check: true
  spell_check_language: "ko-KR"

notifications:
  post_review_comment: true
  summary: true
```

#### 2. .gitignore 업데이트

```gitignore
# .obsidian 설정 파일 무시
.obsidian/cache
.obsidian/workspace
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.DS_Store
Thumbs.db
```

**이유:**
- `workspace.json`은 Obsidian 개인 설정
- Git에 commit할 필요 없음
- Obsidian이 실시간 수정하여 conflict 발생

#### 3. 워크플로우

```bash
# 1. 브랜치 생성
cd ~/Documents/Obsidian
git checkout -b post/new-article

# 2. Obsidian에서 글 작성

# 3. Commit & Push
git add .
git commit -m "feat: Add new article"
git push origin post/new-article

# 4. PR 생성
gh pr create --title "New article" --body "CodeRabbit please review" --fill

# 5. CodeRabbit 리뷰 확인 (1-2분)

# 6. PR 코멘트로 한국어 요청
@coderabbitai 한국어로 리뷰해주세요

# 7. 리뷰 확인 후 Merge
gh pr merge --auto --squash

# 8. main으로 돌아가기
git checkout main
git pull
```

### 한국어 리뷰 설정

**문제:** `.coderabbit.yaml`에 `language: ko` 설정했는데 영어로 리뷰

**해결:** PR 코멘트로 직접 요청
```
@coderabbitai 한국어로 리뷰해주세요
```

→ 즉시 한국어로 다시 리뷰해줌!

---

## 트러블슈팅

### 1. Git Conflict: workspace.json

**문제:**
```
error: Your local changes to the following files would be overwritten by checkout:
        .obsidian/workspace.json
```

**원인:**
- Obsidian이 실시간으로 `workspace.json` 수정
- Git이 변경사항 감지

**해결:**
```bash
# 방법 1: 변경사항 버리기
git restore .obsidian/workspace.json

# 방법 2: Stash
git stash

# 방법 3: Obsidian 종료 후 진행
# 1. Obsidian 종료 (Cmd + Q)
# 2. git stash && git pull
# 3. Obsidian 다시 열기
```

**근본 해결:**
```bash
# .gitignore에 추가
echo ".obsidian/workspace.json" >> .gitignore
echo ".obsidian/workspace-mobile.json" >> .gitignore

git add .gitignore
git commit -m "chore: Ignore workspace files"
git push
```

---

### 2. CodeRabbit Main Push 안 됨

**문제:** Obsidian에서 main push했는데 CodeRabbit 리뷰 없음

**원인:** CodeRabbit은 기본적으로 PR 전용

**해결:** 브랜치 + PR 방식 사용 (위 워크플로우 참고)

---

### 3. Figma MCP "Base path does not exist"

**문제:** Obsidian Git 플러그인에서 에러

**원인:** `.coderabbit.yaml`이 Git 레포 루트에 있어야 함

**해결:**
```bash
# Obsidian vault 위치 확인
pwd
# → /Users/gimdogyeong/Documents/Obsidian (OK!)

# .coderabbit.yaml이 여기에 있는지 확인
ls -la .coderabbit.yaml

# Git 레포 확인
ls -la .git
```

---

### 4. GitHub Actions Token 권한

**문제:** "Bad credentials" 또는 "Resource not accessible"

**해결:**
```
GitHub → Settings → Developer settings → Personal access tokens
→ Token 생성 시 권한 확인:
  ✓ repo (전체)
  ✓ workflow
```

---

## 최종 워크플로우

### A. Obsidian 블로그 글 작성

```bash
# 1. 브랜치 생성
cd ~/Documents/Obsidian
git checkout -b post/my-new-article

# 2. Obsidian에서 글 작성
# Projects/Obslog/my-new-article.md

# 3. Commit & Push
git add .
git commit -m "feat: Add article about React hooks"
git push origin post/my-new-article

# 4. PR 생성
gh pr create --fill

# 5. 한국어 리뷰 요청 (PR 코멘트)
@coderabbitai 한국어로 리뷰해주세요

# 6. 리뷰 확인 후 Merge

# 7. main으로
git checkout main
git pull
```

---

### B. Figma → 코드 (Claude Code)

```
Claude Code에서:

"이 Figma 컴포넌트를 React + TypeScript + tailwind-variants로 만들어줘
https://figma.com/file/abc123?node-id=123

요구사항:
- forwardRef 사용
- 모든 Variants 포함
- 접근성(a11y) 고려"
```

→ 자동으로 컴포넌트 생성!

---

### C. 여러 프로젝트에서 재사용

```typescript
// project-a (영화 앱)
import { Button } from "@/design-system/components/button";

<Button variant="primary">영화 보기</Button>  // 오렌지

// project-b (전자상거래)
// 같은 Button, 다른 테마 (CSS Variables)
<Button variant="primary">장바구니</Button>  // 파란색
```

---

## 핵심 정리

### 재사용 가능한 컴포넌트

- ✅ **tailwind-variants**: 타입 안전한 variant 시스템
- ✅ **cn() 함수**: className 충돌 없이 병합
- ✅ **CSS Variables**: 프로젝트별 테마 변경
- ✅ **Monorepo**: 한 번 작성, 어디서나 사용

### Figma MCP

- ✅ **디자인 → 코드 자동 생성**: 픽셀 퍼펙트
- ✅ **무료**: Token, MCP 서버 모두 무료
- ✅ **Variables 자동 추출**: 디자인 토큰 자동 반영

### Obsidian + CodeRabbit

- ✅ **브랜치 + PR 방식**: Main push는 작동 안 함
- ✅ **한국어 리뷰**: PR 코멘트로 요청
- ✅ **workspace.json 무시**: .gitignore 추가

### GitHub Actions

- ✅ **Submodule 자동 업데이트**: Obsidian push → 자동 배포
- ✅ **완전 무료**: Public 레포 무제한, Private도 충분

---

## 참고 자료

### 가이드 문서

- `/Users/gimdogyeong/projects/FIGMA_MCP_CLAUDE_GUIDE.md`
- `/Users/gimdogyeong/projects/GITHUB_ACTIONS_SETUP_GUIDE.md`
- `/Users/gimdogyeong/projects/GITHUB_ACTIONS_PRICING.md`

### 공식 문서

- [Tailwind Variants](https://www.tailwind-variants.org/)
- [Figma MCP Server](https://developers.figma.com/docs/figma-mcp-server/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [CodeRabbit](https://docs.coderabbit.ai/)
- [Turborepo](https://turbo.build/repo/docs)

### 커뮤니티

- [Shadcn/ui](https://ui.shadcn.com/)
- [Registry Directory](https://registry.directory)
- [Tailwind CSS v4](https://tailwindcss.com/blog/tailwindcss-v4)

---

## 다음 단계

### 1. Obslog 블로그 완성

- [ ] ROADMAP.md 따라 Phase 1-9 진행
- [ ] Obsidian vault ↔ obslog content 연동
- [ ] Vercel 배포

### 2. 컴포넌트 라이브러리 확장

- [ ] 더 많은 컴포넌트 추가 (Card, Input, Modal 등)
- [ ] Storybook 설정
- [ ] 문서화

### 3. Monorepo 구축

- [ ] Turborepo 설정
- [ ] 여러 프로젝트 통합
- [ ] 공유 UI 패키지

---

**작성:** 2026-02-11
**최종 수정:** 2026-02-11
**상태:** ✅ 완료

---

## 변경 이력

| 날짜 | 변경 내용 |
|------|-----------|
| 2026-02-11 | 초기 문서 작성 |
| 2026-02-11 | CodeRabbit 한국어 설정 추가 |
| 2026-02-11 | workspace.json 트러블슈팅 추가 |

---

**🎉 이제 모든 준비가 완료되었습니다!**