# Proposal Orchestrator 설치법

Claude Desktop에서 제안서 자동화 플러그인을 설치하고 작업 환경을 구성하는 가이드입니다.

---

#### mcp   : filesystem은 클로드 공식 커넥터로 연결합니다. 
https://github.com/leedonwoo2827-ship-it/text2img-mcp
https://github.com/leedonwoo2827-ship-it/hwpx-writer

#### 스킬
https://github.com/leedonwoo2827-ship-it/proposal-writer
https://github.com/leedonwoo2827-ship-it/company-data
https://github.com/leedonwoo2827-ship-it/rfp-analysis

Donwnload Zip 으로 다운을 받고, zip파일 뒤에 -master가 있다면 지워둡니다. 스킬은 스킬이름과 zip파일명이 동일해야 
합니다. 

```
{
  "preferences": {
    "coworkScheduledTasksEnabled": true,
    "sidebarMode": "chat",
    "coworkWebSearchEnabled": true
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "C:\\Users\\ubion\\Documents\\proposal"
      ]
    },
    "text2img-mcp": {
      "command": "D:\\mcp\\text2img-mcp\\.venv\\Scripts\\python.exe",
      "args": ["D:\\mcp\\text2img-mcp\\server.py"],
      "env": {
        "GEMINI_API_KEY": "your-api-key"
      }
    },
    "hwpx-writer": {
      "command": "D:\\mcp\\hwpx_writer\\.venv\\Scripts\\python.exe",
      "args": ["D:\\mcp\\hwpx_writer\\server.py"]
    }
  }
}
```



## 1. 사전 준비 체크리스트

플러그인 설치 전에 아래 스킬과 커넥터가 모두 등록되어 있어야 합니다.

### 스킬 (사용자 지정 → 스킬)

| 이름 | 역할 | 상태 |
|------|------|------|
| proposal-writing | 제안서 본문 작성 전문 지식 | ☐ 등록 완료 |
| company-data | 회사/산업 데이터 참조 | ☐ 등록 완료 |
| rfp-analysis | RFP/제안요청서 분석 | ☐ 등록 완료 |

### 커넥터 (사용자 지정 → 커넥터)

| 이름 | 역할 | 상태 |
|------|------|------|
| filesystem | 로컬 파일 읽기/쓰기 | ☐ 등록 완료 |
| hwpx-writer | HWPX 문서 생성 | ☐ 등록 완료 |
| text2img-mcp | 이미지 생성 | ☐ 등록 완료 |

---

## 2. 작업 폴더 구성

프로젝트마다 **날짜-폴더**를 만들고, 그 안에 3개 입력 폴더를 두는 구조입니다.

```
C:\Users\leedonwoo\Documents\proposals\
│
├── 20260303-스마트시티\               ← 프로젝트 1
│   ├── rawdata\                       ← 원본 로데이터
│   │   ├── 회사실적_2025.xlsx
│   │   ├── 시장분석_보고서.pdf
│   │   └── ...
│   ├── references\                    ← 참조 제안서
│   │   ├── A사_클라우드전환_제안서.pdf
│   │   ├── B공단_유사사업_제안서.pdf
│   │   └── ...
│   ├── templates\                     ← 양식 파일
│   │   ├── 공공_표준양식_2026.hwpx
│   │   ├── RFP_스마트시티.pdf
│   │   └── ...
│   └── output\                        ← 결과물 (자동 생성됨)
│       ├── 제안서.hwpx
│       └── images\
│
├── 20260305-디지털전환\               ← 프로젝트 2
│   ├── rawdata\
│   ├── references\
│   ├── templates\
│   └── output\
│
└── 20260310-클라우드이전\             ← 프로젝트 3
    ├── rawdata\
    ├── references\
    ├── templates\
    └── output\
```

### 폴더 이름 규칙

```
YYYYMMDD-프로젝트명
```

예시: `20260303-스마트시티`, `20260305-디지털전환`, `20260310-클라우드이전`

### 각 하위 폴더의 역할

| 폴더 | 넣는 파일 | 예시 |
|------|-----------|------|
| `rawdata\` | 수치 데이터, 통계, 내부 보고서 등 가공 전 원본 자료 | 매출 엑셀, 시장 통계 CSV, 기술 백서 |
| `references\` | 과거 수주 제안서, 경쟁사 제안서, 벤치마크용 샘플 | 기존 제안서 PDF/HWPX |
| `templates\` | 발주처/고객이 제공하는 제안서 양식, RFP 본문 | RFP 양식, 평가 기준표 |
| `output\` | 생성된 결과물 (Claude가 자동으로 생성/관리) | 제안서.hwpx, 이미지 파일 |

### 새 프로젝트 시작 시 폴더 만들기

PowerShell에서 한 번에 만들 수 있습니다:

```powershell
# 예: 2026년 3월 3일 스마트시티 프로젝트
mkdir C:\Users\leedonwoo\Documents\proposals\20260303-스마트시티\rawdata, C:\Users\leedonwoo\Documents\proposals\20260303-스마트시티\references, C:\Users\leedonwoo\Documents\proposals\20260303-스마트시티\templates
```

> `output\` 폴더는 Claude가 결과물 생성 시 자동으로 만듭니다.

---

## 3. filesystem 커넥터 경로 설정

filesystem 커넥터에 `C:\Users\leedonwoo\Documents\proposals` 경로를 추가해야 합니다.

### 설정 파일 위치

```
C:\Users\leedonwoo\AppData\Roaming\Claude\claude_desktop_config.json
```

### 변경 전 (현재 상태)

현재 filesystem은 Obsidian Vault만 접근 가능합니다:

```json
"filesystem": {
  "command": "npx",
  "args": [
    "-y",
    "@modelcontextprotocol/server-filesystem",
    "C:\\Users\\leedonwoo\\Documents\\Obsidian Vault"
  ]
}
```

### 변경 후

`C:\\Users\\leedonwoo\\Documents\\proposals` 경로를 **한 줄 추가**합니다:

```json
"filesystem": {
  "command": "npx",
  "args": [
    "-y",
    "@modelcontextprotocol/server-filesystem",
    "C:\\Users\\leedonwoo\\Documents\\Obsidian Vault",
    "C:\\Users\\leedonwoo\\Documents\\proposals"
  ]
}
```

> 기존 Obsidian Vault 경로는 그대로 유지됩니다.
> 홈 디렉터리(C:\Users\leedonwoo) 안이므로 Cowork 접근 제한에 걸리지 않습니다.
> **변경 후 Claude Desktop을 재시작**해야 적용됩니다.

### 확인 방법

Claude Desktop 재시작 후, 채팅창에서 아래를 입력합니다:

```
Documents\proposals 폴더에 접근할 수 있어?
```

Claude가 폴더 내용을 읽을 수 있으면 정상입니다.

---

## 4. 플러그인 업로드

### 업로드 순서

1. Claude Desktop 실행
2. 좌측 하단 프로필 아이콘 → **사용자 지정** 클릭
3. 좌측 메뉴에서 **플러그인 탐색** 클릭
4. 상단 **개인** 탭 선택 → **+** 버튼 클릭
5. **로컬 플러그인 업로드** 창이 열림
6. `proposal-orchestrator.zip` 파일을 드래그 앤 드롭 (또는 "파일 찾아보기")
7. **업로드** 클릭

### 업로드 후 확인

- 사용자 지정 → 스킬 목록에 `proposal-orchestrator` 관련 항목이 보이면 성공
- 채팅창에서 `/proposal-orchestrator:` 를 입력했을 때 명령어 자동완성이 뜨면 정상

---

## 5. 설치 확인 테스트

채팅창에서 아래를 입력하여 정상 동작을 확인하세요:

```
/proposal-orchestrator:watch-status
현재 환경이 정상인지 확인해줘.
```

Claude가 아래와 비슷하게 응답하면 정상입니다:

```
현재 활성화된 프로젝트가 없습니다.
새 제안서를 만들려면 양식 파일과 함께 요청해 주세요.

[환경 확인]
- filesystem 커넥터: 연결됨
- hwpx-writer 커넥터: 연결됨
- text2img-mcp 커넥터: 연결됨
```

---

## 6. 플러그인 업데이트

플러그인 파일을 수정한 후 다시 적용하는 방법입니다.

### 순서

1. `proposal-orchestrator\` 폴더 안의 파일을 수정
2. `.claude-plugin\plugin.json`에서 `version`을 올림:
   - 오타 수정 → `"1.0.0"` → `"1.0.1"` (PATCH)
   - 기능 추가 → `"1.0.0"` → `"1.1.0"` (MINOR)
   - 구조 변경 → `"1.0.0"` → `"2.0.0"` (MAJOR)
3. ZIP 재생성:

```powershell
Remove-Item "d:\00work\260303-cl\proposal-orchestrator.zip" -Force
Compress-Archive -Path "d:\00work\260303-cl\proposal-orchestrator\*" -DestinationPath "d:\00work\260303-cl\proposal-orchestrator.zip"
```

4. Claude Desktop → 사용자 지정 → 기존 플러그인 삭제
5. 새 ZIP 파일 다시 업로드

### 수정 대상별 파일 위치

| 바꾸고 싶은 것 | 수정할 파일 |
|----------------|-------------|
| 작업 진행 순서/규칙 | `skills\orchestration-workflow.md` |
| 글자 수 배분/분야별 톤 | `skills\proposal-style-guide.md` |
| 이미지 프롬프트 스타일 | `skills\image-prompt-design.md` |
| 워치 보고 형식 | `skills\watch-monitor.md` |
| 제안서 생성 절차 | `commands\create-proposal.md` |
| 이미지 생성 절차 | `commands\generate-images.md` |
| 진행 점검 절차 | `commands\watch-status.md` |
| HWPX 출력 절차 | `commands\export-hwpx.md` |
| 에이전트 시스템 프롬프트 | `agents\proposal-agent.md` |
| 플러그인 이름/설명/버전 | `.claude-plugin\plugin.json` |

---

## 7. 문제 해결

| 증상 | 원인 | 해결 방법 |
|------|------|-----------|
| 명령어가 안 보임 | 플러그인 미설치 또는 비활성 | 사용자 지정에서 플러그인 활성화 확인 |
| 이미지가 생성 안 됨 | text2img-mcp 커넥터 미연결 | 커넥터 목록에서 text2img-mcp 상태 확인 |
| HWPX 출력 실패 | hwpx-writer 커넥터 미연결 | 커넥터 목록에서 hwpx-writer 상태 확인 |
| 파일을 못 읽음 | filesystem 커넥터 미연결 또는 경로 미허용 | 커넥터 경로 설정 확인 (3번 항목 참고) |
| 제안서 품질이 낮음 | 스킬 미등록 | proposal-writing 등 3개 스킬 등록 확인 |
| 업로드 시 오류 | ZIP 내부에 폴더가 한 겹 더 있음 | 폴더 안에서 압축했는지 확인 |
| 업데이트가 반영 안 됨 | version을 안 올림 | plugin.json의 version 번호 확인 |
