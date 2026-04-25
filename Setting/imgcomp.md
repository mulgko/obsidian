# imgcomp - 이미지 압축 스크립트 설치

## 설치 현황

| 항목 | 상태 | 버전 |
|------|------|------|
| pngquant | ✅ 설치됨 | 3.0.3 |
| jpegoptim | ✅ 설치됨 | 1.5.6 |
| `~/.local/bin/imgcomp` | ✅ 생성됨 | — |
| `~/.zshrc` PATH 등록 | ✅ 완료 | — |

## 1. Homebrew 설치 (미설치 시)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/homebrew/install/HEAD/install.sh)"
```

## 2. 필요 도구 설치

```bash
brew install pngquant jpegoptim
```

- `pngquant` — PNG 손실 압축 (색상 팔레트 최적화)
- `jpegoptim` — JPG/JPEG 품질 기반 압축

## 3. 스크립트 디렉토리 생성 및 PATH 등록

```bash
mkdir -p ~/.local/bin
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

## 4. 스크립트 생성

```bash
nano ~/.local/bin/imgcomp
```

에디터가 열리면 아래 코드를 붙여넣기.

```bash
#!/bin/bash

PNGQUANT="/opt/homebrew/bin/pngquant"
JPEGOPTIM="/opt/homebrew/bin/jpegoptim"

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

if [ $# -eq 0 ]; then
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${YELLOW}📦 이미지 압축 도구 v1.0${NC}"
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
    echo "사용법: imgcomp [파일/폴더 드래그]"
    echo ""
    echo -e "${YELLOW}💡 PNG, JPG, JPEG 지원${NC}"
    exit 0
fi

TARGET="$1"
TARGET=$(echo "$TARGET" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')

[ ! -x "$PNGQUANT" ]  && PNGQUANT=$(which pngquant 2>/dev/null)
[ ! -x "$JPEGOPTIM" ] && JPEGOPTIM=$(which jpegoptim 2>/dev/null)

echo -e "${YELLOW}압축 모드를 선택하세요:${NC}"
echo "  1) 보통 (PNG 75-90 + JPG 80%) - 균형"
echo "  2) 강함 (PNG 60-80 + JPG 70%) - 추천"
echo "  3) 최강 (PNG 40-65 + JPG 60%) - 최소 용량"
echo "  4) 화질우선 (PNG 85-95 + JPG 85%)"
echo ""
read -p "선택 (1-4, 엔터=2): " choice

case $choice in
    1) PNG_QUALITY="75-90"; JPG_QUALITY=80 ;;
    3) PNG_QUALITY="40-65"; JPG_QUALITY=60 ;;
    4) PNG_QUALITY="85-95"; JPG_QUALITY=85 ;;
    *) PNG_QUALITY="60-80"; JPG_QUALITY=70 ;;
esac

print_size() {
    local before=$1 after=$2
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
}

compress() {
    local file="$1"
    local name=$(basename "$file")
    local ext=$(echo "${file##*.}" | tr '[:upper:]' '[:lower:]')
    local before=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)

    echo -e "${BLUE}🔄 ${name}${NC}"

    if [ "$ext" = "png" ]; then
        if [ -z "$PNGQUANT" ]; then
            echo -e "${RED}❌ pngquant 미설치 (brew install pngquant)${NC}"
            return 1
        fi
        local temp="${file}.TMP.png"
        "$PNGQUANT" --quality="$PNG_QUALITY" --speed 1 --force --output "$temp" "$file" 2>/dev/null
        if [ -f "$temp" ]; then
            local after=$(stat -f%z "$temp" 2>/dev/null || stat -c%s "$temp" 2>/dev/null)
            if [ "$after" -lt "$before" ]; then
                mv "$temp" "$file"
                print_size "$before" "$after"
                return 0
            else
                rm "$temp"
                echo -e "${YELLOW}⚠️  이미 최적화됨${NC}"
                return 1
            fi
        else
            echo -e "${RED}❌ 실패 (품질 범위 초과일 수 있음)${NC}"
            return 1
        fi

    elif [ "$ext" = "jpg" ] || [ "$ext" = "jpeg" ]; then
        if [ -z "$JPEGOPTIM" ]; then
            echo -e "${RED}❌ jpegoptim 미설치 (brew install jpegoptim)${NC}"
            return 1
        fi
        "$JPEGOPTIM" --max="$JPG_QUALITY" --strip-all --quiet "$file" 2>/dev/null
        local after=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
        if [ "$after" -lt "$before" ]; then
            print_size "$before" "$after"
            return 0
        else
            echo -e "${YELLOW}⚠️  이미 최적화됨${NC}"
            return 1
        fi
    else
        echo -e "${RED}❌ 지원하지 않는 형식${NC}"
        return 1
    fi
}

if [ -f "$TARGET" ]; then
    ext=$(echo "${TARGET##*.}" | tr '[:upper:]' '[:lower:]')
    if [[ "$ext" =~ ^(png|jpg|jpeg)$ ]]; then
        compress "$TARGET"
    else
        echo -e "${RED}❌ PNG/JPG/JPEG 파일이 아닙니다${NC}"
        exit 1
    fi

elif [ -d "$TARGET" ]; then
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${YELLOW}📁 $(basename "$TARGET")${NC}"
    echo -e "${YELLOW}🎨 PNG ${PNG_QUALITY} / JPG ${JPG_QUALITY}%${NC}"
    echo -e "${BLUE}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"

    count=0
    success=0
    total_before=0
    total_after=0

    for file in "$TARGET"/*.png "$TARGET"/*.PNG \
                "$TARGET"/*.jpg "$TARGET"/*.JPG \
                "$TARGET"/*.jpeg "$TARGET"/*.JPEG; do
        [ -f "$file" ] || continue
        ((count++))

        before=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
        total_before=$((total_before + before))

        if compress "$file"; then
            ((success++))
        fi

        after=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
        total_after=$((total_after + after))
    done

    if [ $count -eq 0 ]; then
        echo -e "${RED}❌ PNG/JPG/JPEG 파일 없음${NC}"
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
chmod +x ~/.local/bin/imgcomp
source ~/.zshrc
```

## 6. 사용법

```bash
imgcomp [이미지파일 드래그 or 폴더 드래그]
```

이후 압축 모드 선택 (엔터 = 강함 모드 기본값):

```
1) 보통 (PNG 75-90 + JPG 80%) - 균형
2) 강함 (PNG 60-80 + JPG 70%) - 추천
3) 최강 (PNG 40-65 + JPG 60%) - 최소 용량
4) 화질우선 (PNG 85-95 + JPG 85%)
```

> ⚠️ 원본을 덮어씁니다. 중요한 파일은 미리 백업하세요.
> JPG/JPEG의 경우 EXIF 메타데이터(위치정보, 카메라 정보 등)가 자동 삭제됩니다. 보존하려면 `--strip-all`을 `--strip-com`으로 변경하세요.

## 재설치 시 참고

새 맥에서 처음 설치하거나 초기화 후 재설치할 때 순서대로 실행:

```bash
# 1. 도구 설치
brew install pngquant jpegoptim

# 2. 디렉토리 생성 및 PATH 등록
mkdir -p ~/.local/bin
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc

# 3. 스크립트 생성 후 실행권한 부여
nano ~/.local/bin/imgcomp   # 위 스크립트 붙여넣기
chmod +x ~/.local/bin/imgcomp
source ~/.zshrc

# 4. 동작 확인
imgcomp
```
