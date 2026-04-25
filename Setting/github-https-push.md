# GitHub HTTPS Push 인증 문제 해결

## 증상
```
remote: Repository not found.
fatal: repository 'https://github.com/...' not found
```
- private 레포에 HTTPS push 시 인증 프롬프트 없이 바로 실패
- `credential.helper=osxkeychain` 설정되어 있지만 keychain에 github 항목 없음
- 인증 없이 private 레포 접근 → GitHub가 404 반환

## 원인
osxkeychain에 cached credentials가 없으면 git이 프롬프트를 띄우지 않고 anonymous 요청을 보냄. GitHub는 인증되지 않은 private 레포 요청에 "Repository not found"를 반환.

## 해결 방법

### 1. URL에 username 명시해서 push
```bash
git push https://USERNAME@github.com/ORG/REPO.git BRANCH
```
Password 프롬프트에 **Personal Access Token(PAT)** 입력.

### 2. PAT 발급
GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
- `repo` 스코프 하나만 체크
- 나머지 무시

### 3. Remote에 초기 커밋이 있을 경우 (fetch first 에러)
```
! [rejected] main -> main (fetch first)
```
새 레포 초기 설정이라면 force push:
```bash
git push https://USERNAME@github.com/ORG/REPO.git BRANCH --force
```

## 키체인 캐시 지우기 (필요 시)
Spotlight → "Keychain Access" → github.com 항목 삭제
