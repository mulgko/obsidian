# posts.ts 테스트 가이드

## 실행 방법

```bash
# 1회 실행
npm test

# 파일 저장 시 자동 재실행 (개발 중)
npx vitest
```

### 통과 시 출력

```text
 ✓ src/app/lib/__tests__/posts.test.ts (14 tests) 14ms

 Test Files  1 passed (1)
       Tests  14 passed (14)
    Duration  199ms
```

---

## 파일 구조

```text
src/app/lib/__tests__/posts.test.ts   ← 테스트 코드
src/app/lib/posts.ts                  ← 테스트 대상 코드
content/posts/                        ← 실제 데이터 (mock 없이 그대로 사용)
```

---

## 테스트 코드 구조 설명

### 기본 뼈대

```ts
describe("묶음 이름", () => {
  it("테스트 케이스 설명", () => {
    // 검증 코드
  });
});
```

- `describe` — 관련 테스트를 그룹으로 묶는 컨테이너
- `it` — 테스트 케이스 하나
- `expect(값).toBe(기댓값)` — 실제 결과가 기댓값과 같은지 검증

---

### expect 매처 종류

```ts
expect(value).toBe(true)           // 값이 정확히 true인가
expect(value).toBeNull()           // null인가
expect(value).not.toBeNull()       // null이 아닌가
expect(arr).toEqual([])            // 배열/객체 깊은 비교
expect(obj).toHaveProperty("slug") // 해당 key가 있는가
expect(arr).toContain("react")     // 배열에 값이 포함되어 있는가
expect(fn).not.toThrow()           // 함수 실행 시 에러가 없는가
```

---

## 각 함수별 테스트 설명

### `getAllPosts()` — 반환 타입 + 정렬 검증

```ts
// 타입 검증: 배열인지, 각 요소가 slug/frontmatter를 갖는지
expect(Array.isArray(posts)).toBe(true);
expect(post).toHaveProperty("slug");

// 정렬 검증: 인접한 두 요소를 비교해서 내림차순인지 확인
for (let i = 0; i < posts.length - 1; i++) {
  expect(posts[i].frontmatter.createdAt >= posts[i + 1].frontmatter.createdAt).toBe(true);
}
```

### `getPostBySlug()` — 정상 케이스 + 엣지 케이스

```ts
// 정상: 실제 존재하는 slug로 테스트 (첫 번째 포스트 slug 재활용)
const firstSlug = getAllPosts()[0]?.slug;
const post = getPostBySlug(firstSlug);
expect(post).not.toBeNull();

// 엣지 케이스: 없는 slug → null
const result = getPostBySlug("slug-that-does-not-exist-xyz");
expect(result).toBeNull();
```

### `getAllTags()` — 중복 제거 검증

```ts
const tags = getAllTags(); // { name: string, count: number }[]
const uniqueNames = new Set(tags.map((t) => t.name));
expect(tags.length).toBe(uniqueNames.size); // 길이가 같으면 중복 없음
```

### `savePost()` + `deletePost()` — 파일 시스템 직접 확인

```ts
savePost({ slug: TEST_SLUG, ... });
expect(fs.existsSync(testFilePath)).toBe(true); // 파일이 생겼는가

deletePost(TEST_SLUG);
expect(fs.existsSync(testFilePath)).toBe(false); // 파일이 없어졌는가
```

---

## 테스트 후 정리 (afterEach)

```ts
afterEach(() => {
  if (fs.existsSync(testFilePath)) {
    fs.unlinkSync(testFilePath); // 각 테스트 후 임시 파일 삭제
  }
});
```

`afterEach`는 `it` 블록이 끝날 때마다 실행됩니다.
테스트가 실패해도 정리가 보장됩니다.

---

## 핵심 원칙

| 원칙 | 적용 방법 |
|------|---------|
| 읽기 함수는 실제 데이터로 | mock 없이 `content/posts/` 그대로 사용 |
| 쓰기 함수는 임시 파일로 | `__test-post__` slug로 격리 |
| 항상 엣지 케이스 포함 | 없는 slug, 없는 tag 등 |
| 테스트 후 원상복구 | `afterEach`로 파일 정리 |
