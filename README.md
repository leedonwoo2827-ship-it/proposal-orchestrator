# Proposal Orchestrator - AI 제안서 자동화 플러그인

제안서 작성 + 이미지 생성을 자연어로 오케스트레이션하는 Claude Desktop 플러그인입니다.
양식 분석, 참조 문서 활용, 본문 작성, 이미지 생성, HWPX 출력까지 단계별 자동화를 제공합니다.

---

## 설치법

### 1. 사전 준비 체크리스트

플러그인 설치 전에 아래 스킬과 커넥터가 모두 등록되어 있어야 합니다.

#### 스킬 (사용자 지정 → 스킬)

| 이름 | 역할 | GitHub |
|------|------|--------|
| proposal-writing | 제안서 본문 작성 전문 지식 | https://github.com/leedonwoo2827-ship-it/proposal-writer |
| company-data | 회사/산업 데이터 참조 | https://github.com/leedonwoo2827-ship-it/company-data |
| rfp-analysis | RFP/제안요청서 분석 | https://github.com/leedonwoo2827-ship-it/rfp-analysis |

> Download ZIP으로 다운받고, zip파일 뒤에 `-master`가 있다면 지워둡니다. 스킬은 스킬이름과 zip파일명이 동일해야 합니다.

#### 커넥터 (사용자 지정 → 커넥터)

| 이름 | 역할 |
|------|------|
| filesystem | 로컬 파일 읽기/쓰기 (Claude 공식 커넥터) |
| hwpx-writer | HWPX 한글 문서 생성 |
| text2img-mcp | AI 이미지 생성 (Gemini API) |

MCP 서버 GitHub:
- https://github.com/leedonwoo2827-ship-it/text2img-mcp
- https://github.com/leedonwoo2827-ship-it/hwpx-writer

#### Claude Desktop 설정 파일 예시

`claude_desktop_config.json` 위치: `C:\Users\{사용자}\AppData\Roaming\Claude\claude_desktop_config.json`

```json
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
        "C:\\Users\\ubion\\Documents\\proposals"
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

---

### 2. 작업 폴더 구성

프로젝트마다 **날짜-폴더**를 만들고, 그 안에 3개 입력 폴더를 두는 구조입니다.

```
C:\Users\{사용자}\Documents\proposals\
│
├── 20260303-스마트시티\               ← 프로젝트 1
│   ├── rawdata\                       ← 원본 로데이터
│   ├── references\                    ← 참조 제안서
│   ├── templates\                     ← 양식 파일
│   ├── output\                        ← HWPX 결과물 (자동 생성)
│   └── images\                        ← 생성 이미지 (자동 생성)
│
├── 20260305-디지털전환\               ← 프로젝트 2
└── 20260310-클라우드이전\             ← 프로젝트 3
```

#### 폴더 이름 규칙

```
YYYYMMDD-프로젝트명
```

#### 각 하위 폴더의 역할

| 폴더 | 넣는 파일 | 예시 |
|------|-----------|------|
| `rawdata\` | 수치 데이터, 통계, 내부 보고서 등 원본 자료 | 매출 엑셀, 시장 통계, 기술 백서 |
| `references\` | 과거 수주 제안서, 경쟁사 제안서, 벤치마크 샘플 | 기존 제안서 PDF/HWPX |
| `templates\` | 발주처/고객이 제공하는 제안서 양식, RFP 본문 | RFP 양식, 평가 기준표 |
| `output\` | 생성된 HWPX 문서 (Claude가 자동 관리) | 제안서.hwpx |
| `images\` | 생성된 이미지 (Claude가 자동 관리) | 표지.png, 인포그래픽.png |

#### 새 프로젝트 시작 시 폴더 만들기

```powershell
mkdir C:\Users\{사용자}\Documents\proposals\20260303-스마트시티\rawdata,
      C:\Users\{사용자}\Documents\proposals\20260303-스마트시티\references,
      C:\Users\{사용자}\Documents\proposals\20260303-스마트시티\templates
```

> `output\`과 `images\` 폴더는 Claude가 결과물 생성 시 자동으로 만듭니다.

---

### 3. filesystem 커넥터 경로 설정

filesystem 커넥터에 proposals 경로를 추가합니다:

```json
"filesystem": {
  "command": "npx",
  "args": [
    "-y",
    "@modelcontextprotocol/server-filesystem",
    "C:\\Users\\{사용자}\\Documents\\proposals"
  ]
}
```

> 변경 후 Claude Desktop을 재시작해야 적용됩니다.

---

### 4. 플러그인 업로드

1. Claude Desktop 실행
2. 좌측 하단 프로필 아이콘 → **사용자 지정** 클릭
3. 좌측 메뉴에서 **플러그인 탐색** 클릭
4. 상단 **개인** 탭 선택 → **+** 버튼 클릭
5. `proposal-orchestrator.zip` 파일을 드래그 앤 드롭
6. **업로드** 클릭

---

### 5. 설치 확인

채팅창에서 아래를 입력하여 정상 동작을 확인하세요:

```
/proposal-orchestrator:watch-status
현재 환경이 정상인지 확인해줘.
```

---

## 사용법

### 명령어 목록

| 명령어 | 기능 |
|--------|------|
| `/proposal-orchestrator:create-proposal` | 제안서 전체 생성 |
| `/proposal-orchestrator:generate-images` | 이미지 생성 |
| `/proposal-orchestrator:watch-status` | 진행 상황 점검 |
| `/proposal-orchestrator:export-hwpx` | HWPX 파일 출력 |

> 명령어 없이 자연어로 말해도 Claude가 알아서 실행합니다.

---

### 1. 제안서 전체 생성

#### 기본 사용법

```
/proposal-orchestrator:create-proposal
C:\Users\{사용자}\Documents\proposals\20260303-스마트시티 폴더로 작업해줘.
templates 폴더의 양식으로, references 폴더의 참조제안서를 참고해서
10만자 제안서 만들어줘.
rawdata 폴더의 데이터도 활용해줘.
표지 이미지 1장, 슬라이드 이미지 4장도 같이 만들어줘.
분야는 IT/디지털 전환, 톤은 전문적으로.
```

#### 파일을 직접 첨부하는 경우

파일을 채팅창에 드래그 앤 드롭하고 말합니다:

```
이 양식이랑 참조제안서로 10만자 제안서 만들어줘.
표지 이미지 1장, 핵심 슬라이드용 이미지 4장도 같이 만들어줘.
```

#### Claude가 진행하는 과정

```
1단계: 요청 확인 → 사용자 확인 대기
2단계: 프로젝트 생성 → Project ID 발급
3단계: 양식 분석 → 섹션/목차 도출 → 사용자 확인 대기
4단계: rawdata + 참조제안서 분석
5단계: 본문 작성 (섹션별 진행 보고)
6단계: 이미지 생성 (프롬프트 확인 후 생성)
7단계: HWPX 출력 → output 폴더에 저장
8단계: 결과 요약 → 수정 제안
```

#### 중간에 개입하기

```
여기서 멈춰. 3번 섹션 목차를 좀 바꾸고 싶어.
```

```
rawdata 폴더에 파일 하나 더 추가했어. 추가자료.xlsx 도 참고해줘.
```

---

### 2. 이미지만 생성

```
/proposal-orchestrator:generate-images
이 제안서 요약을 기반으로 썸네일 이미지 3개 만들어 줘.
분위기는 금융/투자 느낌으로.
```

#### 기존 프로젝트에 추가

```
아까 만든 제안서에 인포그래픽 이미지 2장 추가해줘.
시장 분석이랑 수익 모델 섹션에 넣을 거야.
```

#### 이미지 프롬프트 조정

Claude는 이미지 생성 전에 프롬프트를 보여줍니다. 여기서 조정 가능:

```
색상을 남색+금색으로 바꿔줘. 텍스트 넣을 공간을 좀 더 넓혀줘.
```

---

### 3. 진행 상황 점검

```
/proposal-orchestrator:watch-status
지금까지 만든 제안서 구조와 이미지 목록을 정리해서 보여줘.
```

---

### 4. 부분 수정/보완

```
시장 분석 섹션의 분량을 2만자로 늘려줘. 경쟁사 비교표도 추가해줘.
```

```
결론 부분의 톤을 좀 더 자신감 있게 바꿔줘.
```

---

### 5. HWPX 파일 출력

```
/proposal-orchestrator:export-hwpx
지금까지 작성한 내용으로 HWPX 파일 만들어줘. 이미지도 같이 넣어줘.
```

---

### 6. 자연어 요청 매핑

| 이렇게 말하면 | Claude가 하는 일 |
|---------------|-----------------|
| "제안서 만들어줘" | 전체 생성 워크플로 시작 |
| "이미지 3장 만들어줘" | 이미지 생성 |
| "어디까지 했어?" | 진행 상황 점검 |
| "HWPX로 저장해줘" | HWPX 출력 |
| "시장 분석 부분 보완해줘" | 해당 섹션 부분 수정 |
| "표지 이미지 다시 만들어줘" | 이미지 재생성 |
| "빠진 섹션 확인해줘" | 워치 + 보완 제안 |

---

## 결과물의 색상과 서식

### 출처별 글자 색상

| 글자 색상 | 의미 |
|-----------|------|
| **적색 (빨간색)** | rawdata에서 가져온 텍스트 (매출 데이터, 통계, 팩트) |
| **녹색 (초록색)** | 참조 제안서에서 가져온 텍스트 (기존 표현, 구조, 사례) |
| **검정색 (기본)** | AI가 새로 작성한 텍스트 |

> 이 색상 구분으로 검토 시 어느 내용이 원본 데이터 기반인지, 참조 문서에서 온 것인지, AI가 생성한 것인지 한눈에 파악할 수 있습니다.

---

## 팁

### rawdata가 많을 때
- 파일명을 명확하게 지으세요 (예: `2025_매출실적_분기별.xlsx`)
- 특정 파일만 지정할 수도 있습니다:
  ```
  rawdata 폴더의 시장분석_보고서.pdf 랑 산업통계.xlsx 만 참고해줘.
  ```

### 대용량 제안서 (10만자 이상)
- 섹션별로 나누어 순차 작성됩니다
- 각 섹션 완료 시 진행 보고를 받습니다
- 중간에 "여기서 멈춰" 또는 "이 부분 수정해줘"로 개입 가능

### 여러 프로젝트 관리
- 프로젝트마다 날짜-폴더를 만들어 관리합니다
- 새 대화에서 이어 작업하려면:
  ```
  20260303-스마트시티 프로젝트 이어서 진행해줘.
  ```

### 참조 제안서 활용 극대화
- 수주에 성공한 제안서를 넣으면 효과적입니다
- 같은 분야의 참조를 넣으면 톤이 맞습니다
- 여러 개를 넣으면 Claude가 공통 패턴을 추출합니다

---

## 플러그인 업데이트

1. 폴더 안의 파일 수정
2. `.claude-plugin/plugin.json`에서 `version` 올림
3. ZIP 재생성 후 Claude Desktop에서 기존 플러그인 삭제 → 새 ZIP 업로드

### 수정 대상별 파일 위치

| 바꾸고 싶은 것 | 수정할 파일 |
|----------------|-------------|
| 작업 진행 순서/규칙 | `skills/orchestration-workflow.md` |
| 글자 수 배분/분야별 톤 | `skills/proposal-style-guide.md` |
| 이미지 프롬프트 스타일 | `skills/image-prompt-design.md` |
| 워치 보고 형식 | `skills/watch-monitor.md` |
| 제안서 생성 절차 | `skills/create-proposal/SKILL.md` |
| 이미지 생성 절차 | `skills/generate-images/SKILL.md` |
| 진행 점검 절차 | `skills/watch-status/SKILL.md` |
| HWPX 출력 절차 | `skills/export-hwpx/SKILL.md` |
| 에이전트 시스템 프롬프트 | `agents/proposal-agent.md` |
| 플러그인 이름/설명/버전 | `.claude-plugin/plugin.json` |

---

## 문제 해결

| 증상 | 원인 | 해결 방법 |
|------|------|-----------|
| 명령어가 안 보임 | 플러그인 미설치 또는 비활성 | 사용자 지정에서 플러그인 활성화 확인 |
| 이미지가 생성 안 됨 | text2img-mcp 커넥터 미연결 | 커넥터 목록에서 상태 확인 |
| HWPX 출력 실패 | hwpx-writer 커넥터 미연결 | 커넥터 목록에서 상태 확인 |
| 파일을 못 읽음 | filesystem 경로 미허용 | 커넥터 경로 설정 확인 |
| 제안서 품질이 낮음 | 스킬 미등록 | 3개 스킬 등록 확인 |
| 업로드 시 오류 | ZIP 내부에 폴더가 한 겹 더 있음 | 폴더 안에서 압축했는지 확인 |
| 업데이트가 반영 안 됨 | version을 안 올림 | plugin.json의 version 번호 확인 |
