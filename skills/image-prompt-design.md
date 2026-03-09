# 이미지 프롬프트 설계 가이드

## Purpose
제안서에 포함할 이미지를 text2img-mcp로 생성할 때,
효과적인 프롬프트를 설계하기 위한 가이드라인을 제공한다.

## When to Use
- 제안서용 이미지를 생성할 때
- 이미지 프롬프트를 자동으로 설계할 때
- 사용자가 이미지 스타일/분위기를 지정할 때

## 용도별 프롬프트 템플릿

### 표지 이미지 (cover)
- 목적: 제안서의 첫인상, 주제를 시각적으로 전달
- 스타일: 클린, 프로페셔널, 심플
- 프롬프트 요소:
  - 주제 키워드 (예: "디지털 전환", "스마트 시티")
  - 분위기 (예: "modern, professional, clean")
  - 색상 톤 (예: "blue and white corporate tones")
  - 구도 (예: "centered composition, wide angle")
- 예시: "A professional cover image for a digital transformation proposal, modern minimalist design, blue and white corporate color scheme, abstract geometric patterns representing technology and innovation, clean layout with space for title text"

### 본문 슬라이드 이미지 (slide)
- 목적: 핵심 내용을 시각적으로 보조
- 스타일: 인포그래픽, 다이어그램, 개념도
- 프롬프트 요소:
  - 설명할 개념 (예: "시스템 아키텍처", "프로세스 흐름")
  - 데이터 시각화 유형 (예: "flowchart", "infographic")
  - 색상 일관성 (표지와 동일한 톤 유지)
- 예시: "An infographic showing a 5-step business process workflow, flat design icons, arrows connecting each step, corporate blue color palette, clean white background, professional business illustration style"

### 요약 이미지 (summary)
- 목적: 핵심 성과/결론을 한 눈에 전달
- 스타일: 대시보드형, KPI 카드형
- 프롬프트 요소:
  - 핵심 수치/지표
  - 대시보드 레이아웃
  - 성과를 나타내는 시각적 요소

### 결론/비전 이미지 (conclusion)
- 목적: 미래 비전, 기대 효과를 시각적으로 표현
- 스타일: 미래지향적, 긍정적, 역동적
- 프롬프트 요소:
  - 미래 키워드 (예: "growth", "innovation", "success")
  - 상승/성장 이미지 (예: "upward arrows", "growing graph")

## 분위기별 키워드 사전

### 금융/투자
- "financial charts, stock market visualization, wealth management, gold and navy blue, trust and stability"

### 공공/행정
- "government building, public service, community, national flag colors, official and trustworthy"

### IT/기술
- "circuit board patterns, cloud computing, data visualization, neon blue and dark background, futuristic"

### 교육
- "learning environment, books and technology, bright colors, students and teachers, warm and inviting"

### 건설/엔지니어링
- "architectural blueprint, construction site, modern building, steel and glass, precision and scale"

## 프롬프트 작성 원칙
1. **영어로 작성**: 이미지 생성 모델은 영어 프롬프트에 최적화됨
2. **구체적 서술**: "nice image" 대신 구체적인 요소 나열
3. **스타일 명시**: "flat design", "3D render", "photorealistic" 등
4. **네거티브 요소**: 원치 않는 요소 명시 (예: "no text overlay", "no people")
5. **일관성 유지**: 같은 제안서 내 이미지들은 동일한 색상 톤과 스타일 유지
