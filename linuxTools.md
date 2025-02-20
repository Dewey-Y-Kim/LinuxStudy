### 쉘 스크립트를 하위폴더, depth 1깊이에서 실행하도록 하는 쉘 스크립트

```
#!/bin/sh

# 실행할 쉘 스크립트 파일 확인
if [ -z "$1" ]; then
    echo "사용법: $0 <스크립트파일>"
    exit 1
fi

script_file="$1"

# 실행할 스크립트가 존재하는지 확인
if [ ! -f "$script_file" ]; then
    echo "오류: 파일 '$script_file'을 찾을 수 없습니다."
    exit 1
fi

# 현재 폴더 내의 모든 하위 디렉토리 순회
for dir in */; do
    if [ -d "$dir" ]; then
        echo "📂 이동: $dir"
        (cd "$dir" && sh "$script_file")
    fi
done
```
### 폴더내 모든 파일을 폴더명으로 변경해주는 스크립트
```
#!/bin/sh

# 폴더 이름 가져오기
folder_name=$(basename "$PWD")

# 현재 폴더 내의 모든 파일을 처리
for file in *; do
    # 파일인지 확인 (디렉토리는 제외)
    if [ -f "$file" ]; then
        # 기존 파일명에서 마지막 숫자 부분 추출
#        new_suffix=$(echo "$file" | sed 's/.*-\([0-9]*\)$/\1/')
	new_suffix=$(echo "$file" | awk -F'-' '{print $NF}')
        # 새로운 파일명 생성
        new_name="${folder_name}-${new_suffix}"

        # 파일 이름 변경
        mv "$file" "$new_name"
    fi
done
```
### 폴더명으로 압축하는 스크립트
```
#!/bin/bash

# 현재 디렉토리의 폴더 이름 가져오기
folder_name=$(basename "$PWD")

# 현재 디렉토리 내 모든 파일을 압축
zip -r "${folder_name}.zip" *

echo "Compressed all files into ${folder_name}.zip"
```

### 폴더 내의 mkv 혹은 mp4에 같은 이름의 srt를 통합해주는 스크립트 with MKVmerge(MKVtoolnixGUI)
```
#!/bin/bash

# 변환할 폴더 (기본값: 현재 폴더)
TARGET_DIR="."

# 로그 파일 경로
LOG_FILE="$HOME/mkvmerge.log"

# mkvmerge 존재 여부 확인
if ! command -v mkvmerge &> /dev/null; then
    echo "오류: mkvmerge가 설치되어 있지 않습니다. MKVToolNix를 설치하세요."
    exit 1
fi

# 로그 파일 초기화 (기존 로그를 유지하려면 제거)
echo "===== $(date) MKV 변환 로그 시작 =====" >> "$LOG_FILE"

# 폴더 내 모든 mp4 파일 처리
for mp4_file in "$TARGET_DIR"/*.mp4; do
    # mp4 파일이 존재하는지 확인
    [ -e "$mp4_file" ] || continue
    
    # 확장자 제거하여 파일 이름 추출
    filename=$(basename -- "$mp4_file" .mp4)
    srt_file="$TARGET_DIR/$filename.srt"
    mkv_file="$TARGET_DIR/$filename.mkv"

    # 같은 이름의 SRT 파일이 있는지 확인
    if [ -e "$srt_file" ]; then
        echo "▶ $filename.mp4 + $filename.srt → $filename.mkv 변환 중..."

        # mkvmerge 실행 (자막 강제 실행 설정)
        mkvmerge -o "$mkv_file" \
            --forced-track 0:yes \
            --language 0:kor "$srt_file" \
            "$mp4_file"

        echo "✅ 변환 완료: $mkv_file"
    else
        # SRT 파일이 없을 경우 로그 기록
        echo "❌ [$filename] SRT 파일 없음 - 변환 불가" | tee -a "$LOG_FILE"
    fi
done

echo "🎉 모든 변환 작업이 완료되었습니다!"
``` 
