# 구매 추천 UI 명세

## 문서 목적

이 문서는 ENerCY 프로젝트의 구매 추천 결과 표시 UI(`RecommendationView.vue`)의
레이아웃, 컴포넌트 구성, API 데이터 바인딩 규칙을 정의한다.

관련 스킬 문서: `.codex/skills/frontend-development/SKILL.md`
데이터 출처: `GET /api/v1/recommendation`

---

## Logic-less 원칙 준수 선언

이 화면에서 프론트엔드가 직접 수행하는 판단은 없다.

| 금지 항목 | 이유 |
|-----------|------|
| `BUY` / `HOLD` 여부를 프론트엔드에서 계산하는 것 | 추천 판단은 백엔드 책임 |
| `predictedMaxPrice > predictedMinPrice` 비교로 추천 결론 도출 | 도메인 로직 침범 |
| `oilType` 문자열을 강조 대상 외 목적으로 비교하는 것 | 강조 분기 이외의 도메인 로직 침범 |
| 인덱스로 예측 구간을 분리하는 것 | [PA-002] 구간 식별자에 위임 |

모든 판단 결과는 백엔드 API 응답 필드를 통해 수신하고, 프론트엔드는 이를 화면에 바인딩만 한다.

---

## 데이터 출처

### API 응답 구조

```json
{
  "recommendation": {
    "action": "BUY",
    "label": "구매 권장",
    "reasonSummary": "16~23일 구간의 예측 최저가가 기준치를 충족했습니다."
  },
  "priceContext": {
    "predictedMaxPrice": 82.15,
    "predictedMinPrice": 76.80
  }
}
```

---

## 레이아웃 구조

```
┌────────────────────────────────────────────────────┐
│  [앱 바] 구매 추천  |  oilType 표시                 │
├────────────────────────────────────────────────────┤
│                                                    │
│  [A] 구매 추천 결과 카드 (배지 + 문구)               │
│                                                    │
├──────────────────────┬─────────────────────────────┤
│                      │                             │
│  [B] 최대 예측 유가   │  [C] 최소 예측 유가          │
│      카드             │       카드                  │
│                      │                             │
├────────────────────────────────────────────────────┤
│                                                    │
│  [D] LLM 분석 리포트 패널                           │
│      (recommendation.reasonSummary)                │
│                                                    │
└────────────────────────────────────────────────────┘
```

Vuetify 구현 기준:
- [A]: `v-row` > `v-col cols="12"`
- [B][C]: `v-row` > `v-col cols="6"` 각각
- [D]: `v-row` > `v-col cols="12"`

---

## 영역별 컴포넌트 및 API 바인딩 상세

### [A] 구매 추천 결과 카드

추천 배지와 설명 문구를 중앙 정렬로 크게 표시한다.
배지 색상은 `recommendation.action` 값에 따라 전환한다. 프론트엔드가 직접 판단하지 않는다.

| UI 요소 | 컴포넌트 | API 필드 | 조건 |
|---------|----------|---------|------|
| 추천 배지 | `RecommendationBadge.vue` | `recommendation.action` | `BUY` → Vuetify `success` 색상, `HOLD` → `warning` 색상 |
| 추천 문구 | 텍스트 (`v-card-title`) | `recommendation.label` | 배지 아래 또는 옆에 표시 |

---

### [B] 최대 예측 유가 카드 / [C] 최소 예측 유가 카드

두 카드는 항상 나란히 표시된다.
어느 카드를 강조할지는 **백엔드가 제공하는 `highlightTarget` 플래그**에 의존한다.
프론트엔드는 이 플래그 값만 보고 강조 스타일을 전환한다.

#### 공통 바인딩

| UI 요소 | API 필드 |
|---------|---------|
| 최대 예측 유가 수치 | `priceContext.predictedMaxPrice` |
| 최소 예측 유가 수치 | `priceContext.predictedMinPrice` |
| 강조 스타일 전환 기준 | `oilType` | Crude Oil → min 강조, Gasoline·Heating Oil → max 강조 (PA-001 예외 허용) |

#### 강조 스타일 전환 규칙

| `oilType` 값 | 강조 대상 카드 | 강조 스타일 |
|--------------|--------------|------------|
| `"Crude Oil"` | 최소 예측 유가 카드 [C] | 진한 테두리 (`v-card` `variant="outlined"` + `color="primary"`) |
| `"Gasoline"` 또는 `"Heating Oil"` | 최대 예측 유가 카드 [B] | 진한 테두리 (`v-card` `variant="outlined"` + `color="primary"`) |
| 알 수 없는 값 | 강조 없음 | 두 카드 동일 스타일 유지 |

**설계 원칙**: `PriceSummaryCard.vue` 컴포넌트는 `oilType` prop을 받아 강조 스타일을 전환한다.
이는 Logic-less UI 원칙의 **유일한 예외**로, 강조 분기 이외의 판단에는 `oilType`을 사용하지 않는다.

#### 카드 수치 표시 형식

| 항목 | 표시 형식 |
|------|-----------|
| 가격 수치 | `$XX.XX` |
| 단위 표시 | `/bbl` (카드 하단 보조 텍스트) |
| 산출 구간 표시 | `"16~23일 예측 기준"` (카드 하단 보조 텍스트) |

---

### [D] LLM 분석 리포트 패널

추천 근거 요약을 표시하며, 향후 LLM 전문 리포트로 교체 가능한 구조를 유지한다.

| UI 요소 | 컴포넌트 | API 필드 | 비고 |
|---------|----------|---------|------|
| 패널 타이틀 | 정적 텍스트 | — | `"추천 근거"` 고정 문구 |
| 리포트 본문 | `LlmReportPanel.vue` | `recommendation.reasonSummary` | 텍스트 스크롤 영역 |
| 확장 슬롯 | `v-card` named slot | — | 향후 LLM 전문 리포트 삽입 예정 |

---

## `PriceComparisonPanel.vue` 컴포넌트 설계

`[B]`와 `[C]` 카드를 묶어 관리하는 컨테이너 컴포넌트다.

### Props

| prop | 타입 | 필수 | 설명 |
|------|------|------|------|
| `predictedMaxPrice` | `number` | 필수 | 최대 예측 유가 (Day 16~23 기준) |
| `predictedMinPrice` | `number` | 필수 | 최소 예측 유가 (Day 16~23 기준) |
| `oilType` | `string` | 필수 | 강조 대상 카드 결정에 사용 (PA-001 예외 허용) |

### 컴포넌트 책임 경계

- `oilType` 값을 받아 해당 카드에 강조 스타일만 적용한다.
- 강조 분기 외의 비즈니스 로직 판단은 이 컴포넌트 책임이 아니다.

---

## API 데이터 매핑 전체 요약

```
GET /api/v1/recommendation 응답
│
├── recommendation
│   ├── action            → [A] 배지 색상 분기
│   ├── label             → [A] 배지 텍스트
│   └── reasonSummary     → [D] LLM 리포트 패널 본문
└── priceContext
    ├── predictedMaxPrice → [B] 최대 예측 유가 수치
    ├── predictedMinPrice → [C] 최소 예측 유가 수치
    └── (oilType 기반 강조 분기) → [B][C] 강조 스타일 전환 기준 (PA-001 예외 허용)
```

---

## 협의 완료 사항

| 항목 | 내용 | 결과 |
|------|------|------|
| PA-001 | 가격 카드 강조 대상 결정 방식 | `oilType` 기반 프론트엔드 예외 처리 확정 |
| PA-003 | 요약 가격 산출 구간 | Day 16~23 실질 타겟 구간 기준으로 확정 |
