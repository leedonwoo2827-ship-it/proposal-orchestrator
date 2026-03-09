### 1. 디렉토리 구조

```
proposal-orchestrator/
├── .claude-plugin/
│   └── plugin.json              # 플러그인 매니페스트 (필수)
├── .mcp.json                    # MCP 커넥터 설정 (선택)
├── commands/                    # 슬래시 커맨드 (사용자가 직접 호출)
│   ├── create-proposal.md
│   ├── generate-images.md
│   └── watch-status.md
├── skills/                      # 스킬 (Claude가 자동 참조)
│   ├── orchestration-workflow.md
│   ├── proposal-style-guide.md
│   └── image-prompt-design.md
└── agents/                      # 서브에이전트 (선택)
    └── proposal-agent.md
```

### 2. 각 파일 작성법

#### `.claude-plugin/plugin.json` (필수)

```json
{
  "name": "proposal-orchestrator",
  "version": "1.0.0",
  "description": "제안서 작성 + 이미지 생성을 자연어로 오케스트레이션하는 에이전트",
  "author": {
    "name": "Your Name"
  },
  "capabilities": [
    "skill",
    "command",
    "connector"
  ]
}
```

#### `.mcp.json` (이미 등록된 커넥터를 참조)

이미 hwpx-writer, text2img-mcp 등을 **별도 커넥터로 등록**하셨다면, 플러그인 내 `.mcp.json`은 생략하거나 최소화할 수 있습니다. 플러그인 안에 넣고 싶다면:

```json
{
  "mcpServers": {
    "hwpx-writer": {
      "command": "npx",
      "args": ["hwpx-writer-mcp"],
      "env": {}
    },
    "text2img-mcp": {
      "command": "npx",
      "args": ["text2img-mcp"],
      "env": {}
    }
  }
}
```

> 단, 이미 개별 커넥터로 등록되어 있으므로 `.mcp.json`은 **생략해도 무방**합니다. 플러그인의 스킬/커맨드에서 해당 도구들을 자연스럽게 참조하면 됩니다.

#### `commands/create-proposal.md` (슬래시 커맨드 예시)

```markdown
# /proposal-orchestrator:create-proposal

## Description
양식과 참조제안서를 기반으로 제안서를 생성하고, 요청된 이미지를 함께 만듭니다.

## Usage
/proposal-orchestrator:create-proposal

## Parameters
- 양식 파일 또는 텍스트
- 참조 제안서 (선택)
- 목표 글자 수 (예: 50000, 100000)
- 이미지 개수 및 용도 (예: 표지 1장, 슬라이드 4장)
- 톤/스타일 (예: 금융, 공공, 교육)

## What It Does
1. project_id를 생성하고 사용자에게 알림
2. 양식 분석 → 섹션 구조/목차 도출
3. 참조제안서 분석 → 핵심 포인트 추출
4. 목표 글자 수에 맞춰 섹션별 본문 작성
5. 이미지 프롬프트 설계 및 이미지 생성
6. HWPX 파일 생성 (요청 시)
7. 결과 요약 및 다음 단계 제안
```

#### `skills/orchestration-workflow.md` (핵심 스킬)

여기에 설계 문서의 **시스템 프롬프트 초안** (섹션 7)을 포함합니다:

```markdown
# 제안서 오케스트레이션 워크플로

## Purpose
사용자의 자연어 요청을 해석하여 제안서 작성, 이미지 생성, 
진행 상황 보고를 단계적으로 수행한다.

## When to Use
- 사용자가 제안서 작성을 요청할 때
- 이미지 생성이 필요할 때
- 프로젝트 진행 상황을 확인하고 싶을 때

## 입력 파싱 규칙
사용자 메시지에서 다음 정보를 추출하라:
- 작업 유형: 제안서 전체 생성 / 일부 섹션 보완 / 이미지 생성 / 워치
- project_id 또는 설명
- 텍스트 옵션: 글자 수, 톤/스타일
- 이미지 옵션: 개수, 용도(표지/슬라이드/요약), 스타일
- 출력 형식: HWPX 필요 여부

## 오케스트레이션 규칙
1. project_id가 없으면 새로 생성하고 사용자에게 알린다
2. 제안서 전체 생성:
   양식 분석 → 참조 분석 → 본문 작성 → 이미지 생성 → 최종 출력
3. 이미지 생성:
   관련 텍스트 확보 → 이미지 프롬프트 설계 → 생성
4. 워치/점검:
   상태 조회 → 완료/미완료 보고 → 보완 작업 제안

## 대화 스타일
- 각 단계를 "1단계, 2단계…"처럼 짧게 설명하면서 진행
- MCP 툴 호출 전 중요한 설정을 사용자에게 확인
- 결과 반환 시 다음 선택지를 함께 제안

## 주의사항
- 의도가 불명확하면 반드시 되물어본다
- 불완전한 데이터는 감추지 말고 확인한다
- 중간 결과를 공유하여 사용자 개입을 허용한다
```

#### `agents/proposal-agent.md` (서브에이전트, 선택)

```markdown
---
name: proposal-agent
description: 제안서 작성 및 이미지 생성을 오케스트레이션하는 전문 에이전트. 
  사용자가 제안서 관련 작업을 요청할 때 자동으로 활성화된다.
---

너는 "제안서 + 이미지 자동화 비서"다.
사용자는 한글로 자연스럽게 요청하며, 너는 등록된 MCP 툴들을 
오케스트레이션하여 제안서 작성, 이미지 생성, 진행 상황 보고를 수행한다.

[시스템 프롬프트 초안 내용을 여기에 포함]
```

### 3. ZIP으로 패키징

디렉토리를 만든 후, 그대로 zip으로 압축합니다:

```bash
# 디렉토리 최상위에서
cd proposal-orchestrator
zip -r ../proposal-orchestrator.zip .
```

**주의사항:**

- zip 파일 안에 `proposal-orchestrator/` 폴더가 한 겹 더 들어가지 않도록, 디렉토리 **안에서** 압축하거나 직접 내용물을 선택하세요
- 최대 **50MB** 이하
- 코드 없이 **마크다운 + JSON 파일만**으로 구성 가능

### 4. 업로드

1. Claude Desktop → 사용자 지정 → 플러그인 탐색
2. "+" 버튼 또는 "로컬 플러그인 업로드" 클릭
3. 만든 `proposal-orchestrator.zip` 파일을 드래그 앤 드롭 또는 파일 선택
4. "업로드" 클릭

---

## 실제 작업 순서 요약

|단계|할 일|
|---|---|
|1|폴더 구조 생성 (`proposal-orchestrator/`)|
|2|`.claude-plugin/plugin.json` 작성|
|3|`skills/` 에 오케스트레이션 로직, 스타일 가이드 등 마크다운 작성|
|4|`commands/` 에 슬래시 커맨드 마크다운 작성|
|5|(선택) `agents/` 에 서브에이전트 정의|
|6|(선택) `.mcp.json` 에 커넥터 설정 (이미 등록했으면 생략 가능)|
|7|zip으로 압축 후 업로드|

이미 스킬과 커넥터를 개별 등록하셨으므로, 이 플러그인에서는 **오케스트레이션 로직**(어떤 순서로 어떤 도구를 호출할지)에 집중하시면 됩니다. 스킬 마크다운에 설계 문서의 워크플로 내용을 넣으면 Claude가 자동으로 참조하게 됩니다.


---

Update Todos

- [x] Create plugin directory structure
- [x] Create .claude-plugin/plugin.json manifest
- [x] Create skills markdown files
- [x] Create commands markdown files
- [x] Create agents markdown file
- [x] Package as ZIP file

모든 파일 생성과 ZIP 패키징이 완료되었습니다.

---

## 생성된 파일 구조

```
d:/00work/260303-cl/
├── proposal-orchestrator.zip          ← 업로드용 (12KB)
└── proposal-orchestrator/             ← 원본 소스
    ├── .claude-plugin/
    │   └── plugin.json                 # 매니페스트
    ├── skills/
    │   ├── orchestration-workflow.md   # 핵심 오케스트레이션 로직
    │   ├── proposal-style-guide.md    # 제안서 스타일/글자수 배분
    │   ├── image-prompt-design.md     # 이미지 프롬프트 설계 가이드
    │   └── watch-monitor.md           # 워치/모니터링 기능
    ├── commands/
    │   ├── create-proposal.md         # /제안서 전체 생성
    │   ├── generate-images.md         # /이미지 생성
    │   ├── watch-status.md            # /진행 상황 점검
    │   └── export-hwpx.md            # /HWPX 출력
    └── agents/
        └── proposal-agent.md          # 오케스트레이션 서브에이전트
```

## 업로드 방법

1. Claude Desktop → 사용자 지정 → 플러그인 탐색
2. "로컬 플러그인 업로드" 클릭
3. [proposal-orchestrator.zip](vscode-webview://14anrp961fh3qkh8ledtjl9slrm78k07qib0f3e2q9ofbrbkf4n6/proposal-orchestrator.zip) 파일을 드래그 앤 드롭
4. "업로드" 클릭

---

## 나중에 고칠 때 (수정/업데이트 방법)

### 방법 1: 소스 수정 후 재업로드 (권장)

```
1. proposal-orchestrator/ 폴더 안의 파일을 직접 수정
2. plugin.json의 version을 올림 (예: "1.0.0" → "1.1.0")
3. 다시 ZIP으로 압축
4. Claude Desktop에서 기존 플러그인 삭제 후 새 ZIP 업로드
```

**버전 업 규칙:**

|변경 종류|버전 변경|예시|
|---|---|---|
|오타 수정, 문구 다듬기|PATCH|1.0.0 → 1.0.1|
|새 커맨드 추가, 스킬 보강|MINOR|1.0.0 → 1.1.0|
|구조 변경, 호환성 깨짐|MAJOR|1.0.0 → 2.0.0|

### 방법 2: 빠른 재패키징 명령어

PowerShell에서 한 줄로 재압축할 수 있습니다:

```powershell
# 기존 zip 삭제 후 재생성
Remove-Item "d:\00work\260303-cl\proposal-orchestrator.zip" -Force
Compress-Archive -Path "d:\00work\260303-cl\proposal-orchestrator\*" -DestinationPath "d:\00work\260303-cl\proposal-orchestrator.zip"
```

### 방법 3: 자주 수정하는 경우

|수정 대상|파일 위치|
|---|---|
|오케스트레이션 로직 변경|`skills/orchestration-workflow.md`|
|글자수 배분/톤 가이드 수정|`skills/proposal-style-guide.md`|
|이미지 프롬프트 템플릿 수정|`skills/image-prompt-design.md`|
|워치 보고 형식 수정|`skills/watch-monitor.md`|
|슬래시 커맨드 절차 수정|`commands/*.md`|
|에이전트 시스템 프롬프트 수정|`agents/proposal-agent.md`|
|플러그인 이름/설명 변경|`.claude-plugin/plugin.json`|

**핵심 포인트**: `proposal-orchestrator/` 폴더를 원본으로 유지하세요. 수정할 때는 항상 이 폴더의 파일을 고친 뒤 ZIP으로 다시 압축하면 됩니다.