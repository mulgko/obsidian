# gifcomp - GIF 압축 스크립트 설치

## 1. Homebrew 설치 (미설치 시)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/homebrew/install/HEAD/install.sh)"
```

## 2. gifsicle 설치

```bash
brew install gifsicle
```

## 3. 스크립트 디렉토리 생성 및 PATH 등록

```bash
mkdir -p ~/.local/bin
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

## 4. 스크립트 생성

```bash
nano ~/.local/bin/gifcomp
```

에디터가 열리면 아래 코드를 붙여넣기.

```bash
#!/bin/bash

GIFSICLE="/opt/homebrew/bin/gifsicle"

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

if [ $# -eq 0 ]; then
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${YELLOW}📦 GIF 압축 도구 v3.0${NC}"
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
    echo "사용법: gifcomp [파일/폴더 드래그]"
    echo ""
    echo -e "${YELLOW}💡 드래그 후 엔터로 압축 모드 선택!${NC}"
    exit 0
fi

TARGET="$1"
TARGET=$(echo "$TARGET" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')

if [ -z "$2" ]; then
    echo -e "${YELLOW}압축 모드를 선택하세요:${NC}"
    echo "  1) 보통 (lossy 60 + 128색) - 균형"
    echo "  2) 강함 (lossy 80 + 64색) - 추천"
    echo "  3) 최강 (lossy 100 + 32색) - 최소 용량"
    echo "  4) 화질우선 (lossy 없음 + 256색)"
    echo ""
    read -p "선택 (1-4, 엔터=2): " choice

    case $choice in
        1) COLORS=128; LOSSY=60 ;;
        3) COLORS=32; LOSSY=100 ;;
        4) COLORS=256; LOSSY=0 ;;
        *) COLORS=64; LOSSY=80 ;;
    esac
else
    COLORS="$2"
    COLORS=$(echo "$COLORS" | xargs)
    LOSSY=80
fi

compress() {
    local file="$1"
    local colors="$2"
    local lossy="$3"
    local temp="${file}.TMP.gif"
    local name=$(basename "$file")

    local before=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)

    echo -e "${BLUE}🔄 ${name}${NC}"

    if [ "$lossy" -gt 0 ]; then
        "$GIFSICLE" -O3 --lossy="$lossy" --colors "$colors" "$file" --output "$temp" 2>/dev/null
    else
        "$GIFSICLE" -O3 --colors "$colors" "$file" --output "$temp" 2>/dev/null
    fi

    if [ -f "$temp" ]; then
        local after=$(stat -f%z "$temp" 2>/dev/null || stat -c%s "$temp" 2>/dev/null)

        if [ $after -lt $before ]; then
            mv "$temp" "$file"
            local saved=$((before - after))
            local percent=$((saved * 100 / before))

            if [ $before -gt 1048576 ]; then
                local bMB=$(awk "BEGIN {printf \"%.1f\", $before/1048576}")
                local aMB=$(awk "BEGIN {printf \"%.1f\", $after/1048576}")
                echo -e "${GREEN}✅ ${bMB}MB → ${aMB}MB (-${percent}%)${NC}"
            else
                local bKB=$((before / 1024))
                local aKB=$((after / 1024))
                echo -e "${GREEN}✅ ${bKB}KB → ${aKB}KB (-${percent}%)${NC}"
            fi
            return 0
        else
            rm "$temp"
            echo -e "${YELLOW}⚠️  이미 최적화됨${NC}"
            return 1
        fi
    else
        rm -f "$temp"
        echo -e "${RED}❌ 실패${NC}"
        return 1
    fi
}

if [ -f "$TARGET" ]; then
    if [[ "$TARGET" =~ \.(gif|GIF)$ ]]; then
        compress "$TARGET" "$COLORS" "$LOSSY"
    else
        echo -e "${RED}❌ GIF 파일이 아닙니다${NC}"
        exit 1
    fi

elif [ -d "$TARGET" ]; then
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${YELLOW}📁 $(basename "$TARGET")${NC}"
    if [ "$LOSSY" -gt 0 ]; then
        echo -e "${YELLOW}🎨 Lossy ${LOSSY} + ${COLORS}색${NC}"
    else
        echo -e "${YELLOW}🎨 ${COLORS}색 (무손실)${NC}"
    fi
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"

    count=0
    success=0
    total_before=0
    total_after=0

    for file in "$TARGET"/*.gif "$TARGET"/*.GIF; do
        [ -f "$file" ] || continue
        ((count++))

        before=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
        total_before=$((total_before + before))

        if compress "$file" "$COLORS" "$LOSSY"; then
            ((success++))
        fi

        after=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
        total_after=$((total_after + after))
    done

    if [ $count -eq 0 ]; then
        echo -e "${RED}❌ GIF 파일 없음${NC}"
        exit 1
    fi

    if [ $success -gt 0 ] && [ $total_after -lt $total_before ]; then
        saved=$((total_before - total_after))
        percent=$((saved * 100 / total_before))

        if [ $total_before -gt 1048576 ]; then
            bMB=$(awk "BEGIN {printf \"%.1f\", $total_before/1048576}")
            aMB=$(awk "BEGIN {printf \"%.1f\", $total_after/1048576}")
            sMB=$(awk "BEGIN {printf \"%.1f\", $saved/1048576}")
            echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
            echo -e "${GREEN}📊 ${success}/${count}개 완료${NC}"
            echo -e "${GREEN}📦 ${bMB}MB → ${aMB}MB${NC}"
            echo -e "${GREEN}💾 -${percent}% (${sMB}MB 절약)${NC}"
        else
            bKB=$((total_before / 1024))
            aKB=$((total_after / 1024))
            sKB=$((saved / 1024))
            echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
            echo -e "${GREEN}📊 ${success}/${count}개 완료${NC}"
            echo -e "${GREEN}📦 ${bKB}KB → ${aKB}KB${NC}"
            echo -e "${GREEN}💾 -${percent}% (${sKB}KB 절약)${NC}"
        fi
        echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    else
        echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
        echo -e "${YELLOW}📊 ${count}개 처리 (압축 효과 없음)${NC}"
        echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    fi

else
    echo -e "${RED}❌ 파일/폴더 없음${NC}"
    exit 1
fi
```

저장: `Ctrl+O` → `Enter` → `Ctrl+X`

## 5. 실행 권한 부여 및 설정 적용

```bash
chmod +x ~/.local/bin/gifcomp
source ~/.zshrc
```

## 6. 사용법

```bash
gifcomp [gif파일 드래그 or 폴더 드래그]
```

이후 압축 모드 선택 (엔터 = 강함 모드 기본값):

```
1) 보통 (lossy 60 + 128색) - 균형
2) 강함 (lossy 80 + 64색)  - 추천
3) 최강 (lossy 100 + 32색) - 최소 용량
4) 화질우선 (lossy 없음 + 256색)
```

> ⚠️ 원본을 덮어씁니다. 중요한 파일은 미리 백업하세요.
