# /proposal-orchestrator:export-hwpx

작성된 제안서를 HWPX 파일로 내보내는 커맨드입니다.

## 핵심 원칙

> 한 번의 응답에서 hwpx-writer MCP를 1회만 호출하라.
> 이미지가 많으면 경로를 미리 정리한 뒤 한 번에 전달하라.

## 실행 절차

### 1단계: 프로젝트 확인
- project_dir 확인
- output/ 폴더와 images/ 폴더 상태 확인
- 미완성 섹션이 있으면 사용자에게 알림

### 2단계: 출력 옵션 확인
- **출력 파일명**: 기본값은 프로젝트명 기반
- **이미지 포함 여부**: images/ 폴더의 이미지를 문서에 삽입할지

### 3단계: HWPX 생성
- hwpx-writer의 `convert_text_to_hwpx` 호출
- `project_dir` 파라미터로 output/ 폴더에 자동 저장
- `image_paths` 파라미터로 이미지 삽입 (쉼표 구분 절대경로)

### 색상/서식 처리
- `{{red:텍스트}}` → hwpx-writer가 적색 글자로 자동 변환
- `{{green:텍스트}}` → hwpx-writer가 녹색 글자로 자동 변환
- 마커 없는 텍스트 → 기본 검정색
- 별도 변환 작업 불필요. hwpx-writer가 마커를 인식하여 자동 처리한다.

### 4단계: 결과 안내
- 생성된 파일 경로와 크기 안내

## 사용 예시
```
/proposal-orchestrator:export-hwpx
지금까지 작성한 내용으로 HWPX 파일 만들어줘. 이미지도 같이 넣어줘.
```
