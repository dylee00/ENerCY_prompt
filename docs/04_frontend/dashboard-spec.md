# 대시보드 UI 명세

## 문서 목적

이 문서는 ENerCY 프로젝트 메인 대시보드(`DashboardView.vue`)의
레이아웃 구조, API 데이터 매핑, LLM 분석 리포트 패널 위치를 정의한다.

관련 스킬 문서: `.codex/skills/frontend-development/SKILL.md`
데이터 출처: `GET /api/v1/dashboard`

---

## 핵심 표시 요소 (필수 4요소)

대시보드는 아래 4가지 정보를 사용자가 한눈에 파악할 수 있도록 설계한다.

| 필수 요소 | API 필드 | 표시 위치 |
|-----------|---------|-----------|
| 현재 판단 시점 | `currentDecisionTime` | 상단 헤더 영역 |
| 최대 예측 유가 | `forecastSummary.predictedMaxPrice` | 좌측 상단 카드 |
| 최소 예측 유가 | `forecastSummary.predictedMinPrice` | 좌측 상단 카드 |
| 구매 추천 결과 | `recommendation.action`, `recommendation.label` | 좌측 중앙 카드 |

※ `predictedMaxPrice` / `predictedMinPrice`는 **실질 타겟 구간(Day 16~23) 기준**으로 산출된 값이다.

---

## 레이아웃 구조

대시보드는 **2단 레이아웃**으로 구성한다.

```
┌────────────────────────────────────────────────────────────────┐
│  [앱 바] ENerCY  |  oilType 표시  |  모델 상태 배지            │
├─────────────────────────────┬──────────────────────────────────┤
│                             │                                  │
│  [A] 판단 시점 헤더          │                                  │
│                             │                                  │
│  [B] 현재 유가 카드          │   [E] LLM 분석 리포트 패널       │
│                             │                                  │
│  [C] 최대/최소 예측 유가 카드 │   (recommendation.reasonSummary │
│                             │    표시 영역)                     │
│  [D] 구매 추천 배지 카드     │                                  │
│                             │                                  │
│  [F] 예측 차트 (축약)        │                                  │
│                             │                                  │
└─────────────────────────────┴──────────────────────────────────┘
```

- **좌측 영역 (2/3)**: 필수 4요소 카드 + 예측 차트 축약본
- **우측 영역 (1/3)**: LLM 분석 리포트 전용 패널

Vuetify 구현 기준: `v-row` + `v-col cols="8"` (좌) / `v-col cols="4"` (우)

---

## 영역별 컴포넌트 및 API 바인딩 상세

### [앱 바] 상단 네비게이션

| UI 요소 | 컴포넌트 | API 필드 | 비고 |
|---------|----------|---------|------|
| oilType 표시 | 텍스트 | `oilType` | 표시 텍스트 및 **가격 강조 분기**에 사용. 그 외 비즈니스 로직 판단에는 사용 금지 |
| 모델 상태 배지 | `StatusBadge.vue` | `monitoring.modelStatus` | `HEALTHY` → 녹색, 그 외 → 경고색 |
| 재학습 경고 | `v-banner` | `monitoring.retrainingRequired` | `true`일 때만 표시 |

---

### [A] 판단 시점 헤더

| UI 요소 | 컴포넌트 | API 필드 | 표시 형식 |
|---------|----------|---------|-----------|
| 판단 시점 텍스트 | `DecisionTimeCard.vue` | `currentDecisionTime` | `YYYY년 MM월 DD일 기준` |
| 데이터 생성 시각 | 부제목 텍스트 | `generatedAt` | `YYYY-MM-DD HH:mm 갱신` |

---

### [B] 현재 유가 카드

| UI 요소 | 컴포넌트 | API 필드 | 표시 형식 |
|---------|----------|---------|-----------|
| 현재 유가 수치 | `CurrentPriceCard.vue` | `currentPrice` | `$XX.XX / bbl` |

---

### [C] 최대/최소 예측 유가 카드

`PriceSummaryCard.vue` 컴포넌트는 두 카드를 렌더링한다.
어느 카드를 강조할지는 **백엔드가 제공하는 UI 표시용 메타 플래그**에 의존한다.
프론트엔드는 `oilType` 값을 직접 비교하여 강조 대상을 결정하지 않는다.

| UI 요소 | API 필드 | 비고 |
|---------|---------|------|
| 최대 예측 유가 수치 | `forecastSummary.predictedMaxPrice` | 항상 표시 |
| 최소 예측 유가 수치 | `forecastSummary.predictedMinPrice` | 항상 표시 |
| 강조 스타일 전환 기준 | `oilType` | Crude Oil → `predictedMinPrice` 강조, Gasoline·Heating Oil → `predictedMaxPrice` 강조 (PA-001 예외 허용) |

**설계 원칙**: `oilType` 값에 따라 강조 카드를 결정한다.
이는 Logic-less UI 원칙의 **유일한 예외 항목**으로 관리한다.
- `oilType === 'Crude Oil'`: 최소 예측 유가 카드(`predictedMinPrice`) 강조
- `oilType === 'Gasoline'` 또는 `'Heating Oil'`: 최대 예측 유가 카드(`predictedMaxPrice`) 강조

---

### [D] 구매 추천 배지 카드

| UI 요소 | 컴포넌트 | API 필드 | 조건 |
|---------|----------|---------|------|
| 추천 배지 | `RecommendationBadge.vue` | `recommendation.action` | `BUY` → `success` 색상, `HOLD` → `warning` 색상 |
| 추천 문구 | 텍스트 | `recommendation.label` | 배지 옆에 표시 |
| 임계치 통과 여부 | 보조 텍스트 | `recommendation.thresholdStatus` | `PASSED` / `FAILED` 텍스트 표시 |

---

### [E] LLM 분석 리포트 패널

이 패널은 향후 LLM 분석 기능 확장을 위해 **반드시 설계에 포함**한다.
현재는 `recommendation.reasonSummary` 내용을 표시하며,
추후 LLM 생성 리포트 전문으로 교체될 수 있는 구조로 설계한다.

| UI 요소 | 컴포넌트 | API 필드 | 비고 |
|---------|----------|---------|------|
| 패널 타이틀 | 정적 텍스트 | — | `"분석 리포트"` 고정 문구 |
| 리포트 본문 | `LlmReportPanel.vue` | `recommendation.reasonSummary` | 텍스트 스크롤 영역 |
| 확장 슬롯 | `v-card` 내 named slot | — | 향후 LLM 리포트 전문 삽입 예정 |

**설계 주의사항**: `LlmReportPanel.vue` 컴포넌트 내부에 `fullReport` named slot을
미리 정의하여, LLM 리포트 전문 교체 시 이 컴포넌트만 수정하면 되는 구조를 유지한다.

---

### [F] 예측 차트 (축약본)

대시보드에는 `ForecastChart.vue`의 축약 버전을 표시한다.
전체 차트는 `/forecast` 페이지에서 확인한다.

| UI 요소 | 컴포넌트 | 데이터 출처 | 비고 |
|---------|----------|------------|------|
| 예측 시계열 미니 차트 | `ForecastChart.vue` | `useForecastStore` | 높이 제한 버전 |
| "상세 보기" 버튼 | `v-btn` | — | `/forecast` 라우트로 이동 |

---

## API 데이터 매핑 전체 요약

```
GET /api/v1/dashboard 응답
│
├── generatedAt              → [A] 갱신 시각 텍스트
├── currentDecisionTime      → [A] 판단 시점 헤더
├── oilType                  → [앱 바] 표시 텍스트 전용 (UI 분기 금지)
├── currentPrice             → [B] 현재 유가 카드
├── forecastSummary
│   ├── predictedMaxPrice    → [C] 최대 예측 유가 수치 (Day 16~23 기준)
│   └── predictedMinPrice    → [C] 최소 예측 유가 수치 (Day 16~23 기준)
├── oilType                  → [C] 가격 카드 강조 분기 기준 (Crude Oil → min, Gasoline·Heating Oil → max)
├── recommendation
│   ├── action               → [D] 배지 색상 분기
│   ├── label                → [D] 배지 텍스트
│   ├── reasonSummary        → [E] LLM 리포트 패널 본문
│   └── thresholdStatus      → [D] 임계치 통과 여부 텍스트
└── monitoring
    ├── modelStatus          → [앱 바] 모델 상태 배지
    └── retrainingRequired   → [앱 바] 재학습 경고 배너
```

---

## 협의 완료 사항

| 항목 | 내용 | 결과 |
|------|------|------|
| PA-001 | UI 강조 대상 결정 방식 | `oilType` 기반 프론트엔드 예외 처리 확정 |
| PA-003 | 요약 가격 산출 구간 | Day 16~23 실질 타겟 구간 기준으로 확정 |
