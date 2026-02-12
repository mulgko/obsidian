
1. git / obsidian 설치
2. git에서 .gitignore 먼저 작성
```
# obsidian
.obsidian/cache
.obsidian/workspace
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.DS_Store
Thumbs.db
.obsidian/graph.json
```
4. terminal에서 obsidian vault 찾아서 git init
5. git remote add origin https://github.com/USERNAME/obsidian.git
6. 나머지는 commit / push / pull 기존 git처럼 사용하기.