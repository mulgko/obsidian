---
title: Pagination
subject: ""
reference: ""
date: 2026-02-25 16:43
description: ""
tags:
  - frontend
  - pagination
series: ""
seriesOrder:
published: false
---

# Pagination

프론트엔드 작업을 하다가 보면, 꼭 페이지네이션 작업을 해야할 때가 있다. 그때마다 헷갈려서 만든 참고서 

```typescript

// 각각 페이지: 현재 바라보고 있어야하는 페이지, tags: 필터링된 태그들, pageSize: 한 페이지에 들어가야하는 컨텐츠 수
export function getPaginatedPosts(page: number, tags?: string[], pageSize = 4) {



// 전체 포스트 가져와 준다.
const allPosts = getAllPosts();

// 필터링 해야하기 때문에 따로 변수 선언 해준다.
let filteredPosts = allPosts;


// 만약 태그가 있고, 태그 갯수가 0보다 클 때
if (tags && tags.length > 0) {

// 필터링된 포스트는 필터 함수를 통해 태그가 포함된 포스트들로만 이루어져 반환된다. 
filteredPosts = filteredPosts.filter((post) => {
return post.frontmatter.tags?.some((tag) => tags.includes(tag));
});

}

  
// 총 페이지의 갯수: 태그가 없으면 전체를 가져오고, 태그가 있으면 태그가 포함된 포스팅만 가져온다. Math.ceil 하는 이유는 필터된 포스트의 갯수가 1이고, page의 size가 4일 때 값은 0.25가 된다. 그치만 floor, round는 전부 페이지가 나오지 않게 된다. 왜냐면 ceil처럼 '올림'이 아니기 때문이다. ceil 0.25라는 값이 나와도 1페이지를 보여주게 된다. 그래서 설령 페이지의 사이즈보다 필터링된 포스트의 내용이 없더라도 화면에 렌더링 되게 된다. 
const totalPages = Math.ceil(filteredPosts.length / pageSize);

const start = (page - 1) * pageSize;

const end = start + pageSize;

const posts = filteredPosts.slice(start, end);

  

return { posts, totalPages, currentPage: page };

}
```

| page | start | end | 가져오는 포스트 인덱스 |
| ---- | ----- | --- | ------------ |
| 1    | 0     | 4   | 0, 1, 2, 3   |
| 2    | 4     | 8   | 4, 5, 6, 7   |
| 3    | 8     | 12  | 8, 9, 10, 11 |
