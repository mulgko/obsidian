
- 참고: [[npm]]

- 태그: #npm #css #utility


### 

```
clsx
```

 (깔끔한 스타일 코드)

- **상세 역할:** 조건에 따라 CSS 클래스를 붙였다 뗐다 할 때 코드를 깔끔하게 만들어주는 유틸리티
- **실제 예시:**
    
    javascript
    
    // clsx가 없다면? (지저분함)
    
    const className = "btn " + (isActive ? "btn-active" : "btn-disabled") + " " + size;
    
    // clsx를 쓴다면? (가독성 폭발)
    
    const className = clsx("btn", isActive ? "btn-active" : "btn-disabled", size);
    
- **왜 사용하나요?** Tailwind CSS를 쓰다 보면 클래스명이 길어지고 조건부 스타일이 많아지는데, 이를 관리하기 위한 필수 도구


