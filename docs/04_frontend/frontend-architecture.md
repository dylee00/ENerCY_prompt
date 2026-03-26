# 프론트엔드 아키텍처 명세

## 문서 목적

이 문서는 ENerCY 프로젝트 프론트엔드의 Vue 프로젝트 폴더 구조,
전역 상태 관리 전략, 라우팅 구조를 정의한다.

관련 스킬 문서: `.codex/skills/frontend-development/SKILL.md`

---

## 기술 스택 요약

| 항목 | 선택 |
|------|------|
| Framework | Vue 3 (`<script setup>`, Composition API) |
| UI Library | Vuetify 3 |
| 상태 관리 | Pinia |
| HTTP 클라이언트 | Axios |
| 라우터 | Vue Router 4 |
| 차트 | Chart.js + vue-chartjs + chartjs-plugin-annotation |
| 빌드 도구 | Vite |

---

## 프로젝트 폴더 구조

```
src/
├── api/                        # Axios 기반 API 호출 모듈
│   ├── dashboardApi.ts         # GET /api/v1/dashboard
│   ├── forecastApi.ts          # GET /api/v1/forecast
│   ├── recommendationApi.ts    # GET /api/v1/recommendation
│   └── monitoringApi.ts        # GET /api/v1/monitoring
│
├── assets/                     # 정적 리소스 (이미지, 폰트 등)
│
├── components/                 # 재사용 가능한 단위 컴포넌트
│   ├── common/
│   │   ├── StatusBadge.vue         # 모델 상태 배지 (HEALTHY / DEGRADED 등)
│   │   ├── RecommendationBadge.vue # 추천 배지 (BUY / HOLD)
│   │   └── LoadingOverlay.vue      # 로딩 오버레이
│   │
│   ├── dashboard/
│   │   ├── DecisionTimeCard.vue    # 현재 판단 시점 표시
│   │   ├── CurrentPriceCard.vue    # 현재 유가 카드
│   │   ├── PriceSummaryCard.vue    # 최대/최소 예측 유가 카드 (oilType 강조 포함)
│   │   ├── RecommendationCard.vue  # 구매 추천 결과 카드
│   │   └── LlmReportPanel.vue      # LLM 분석 리포트 전용 패널
│   │
│   ├── forecast/
│   │   └── ForecastChart.vue       # Chart.js 기반 예측 시계열 차트
│   │
│   ├── recommendation/
│   │   └── PriceComparisonPanel.vue # 최대/최소 유가 비교 패널
│   │
│   └── monitoring/
│       ├── ModelStatusPanel.vue    # 모델 상태 + 재학습 경고 패널
│       └── MetricsPanel.vue        # 구간별 성능 지표 패널
│
├── composables/                # Vue 3 composable (비즈니스 로직 없음, 데이터 페칭/포맷만)
│   ├── useDashboard.ts
│   ├── useForecast.ts
│   ├── useRecommendation.ts
│   └── useMonitoring.ts
│
├── router/
│   └── index.ts                # Vue Router 라우트 정의
│
├── stores/                     # Pinia 전역 상태 스토어
│   ├── useDashboardStore.ts
│   ├── useForecastStore.ts
│   ├── useRecommendationStore.ts
│   └── useMonitoringStore.ts
│
├── views/                      # 라우트 단위 페이지 컴포넌트
│   ├── DashboardView.vue       # 메인 대시보드 페이지
│   ├── ForecastView.vue        # 예측 시계열 페이지
│   ├── RecommendationView.vue  # 구매 추천 상세 페이지
│   └── MonitoringView.vue      # 시스템 모니터링 페이지
│
├── App.vue                     # 루트 컴포넌트
└── main.ts                     # 앱 진입점 (Vuetify, Pinia, Router 등록)
```

---

## 전역 상태 관리 전략 (Pinia)

### 원칙

- 스토어는 API 응답 데이터를 **원본 그대로** 보관한다.
- 스토어에서 임계치 계산, 추천 판단 등 비즈니스 로직을 수행하지 않는다.
- 컴포넌트는 스토어에서 데이터를 읽어 화면에 바인딩한다.

### 스토어 구조 패턴

각 스토어는 아래 구조를 따른다.

```ts
// 예시: useDashboardStore.ts
export const useDashboardStore = defineStore('dashboard', () => {
  const data = ref<DashboardResponse | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function fetch() {
    loading.value = true
    error.value = null
    try {
      data.value = await dashboardApi.get()
    } catch (e) {
      error.value = '데이터를 불러오지 못했습니다.'
    } finally {
      loading.value = false
    }
  }

  return { data, loading, error, fetch }
})
```

### 스토어별 보관 데이터

| 스토어 | 보관 데이터 | 출처 API |
|--------|-------------|----------|
| `useDashboardStore` | `DashboardResponse` 전체 | `GET /api/v1/dashboard` |
| `useForecastStore` | `forecastHorizon`, `forecastPoints[]` | `GET /api/v1/forecast` |
| `useRecommendationStore` | `recommendation`, `priceContext` | `GET /api/v1/recommendation` |
| `useMonitoringStore` | `modelStatus`, `retrainingRequired`, `latestModelVersion`, `latestEvaluation` | `GET /api/v1/monitoring` |

---

## 라우팅 구조

### 라우트 정의

| 경로 | 컴포넌트 | 설명 |
|------|----------|------|
| `/` | `DashboardView.vue` | 메인 대시보드 (기본 진입점) |
| `/forecast` | `ForecastView.vue` | 예측 시계열 상세 |
| `/recommendation` | `RecommendationView.vue` | 구매 추천 상세 |
| `/monitoring` | `MonitoringView.vue` | 시스템 모니터링 상태 |

### 네비게이션 구조

- 상단 앱 바(`v-app-bar`) 또는 좌측 네비게이션 드로어(`v-navigation-drawer`)를 통해 페이지 전환
- 기본 진입 경로는 `/` (대시보드)

---

## API 호출 레이어 설계

### 원칙

- 모든 HTTP 요청은 `src/api/` 하위 모듈을 통해서만 수행한다.
- 컴포넌트 또는 composable에서 직접 `axios.get()`을 호출하지 않는다.
- API 모듈은 응답 데이터를 그대로 반환한다. 가공하지 않는다.

### Axios 인스턴스 설정

```ts
// src/api/axiosInstance.ts
import axios from 'axios'

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
})

export default apiClient
```

### 환경 변수

| 변수명 | 설명 |
|--------|------|
| `VITE_API_BASE_URL` | 백엔드 API 서버 주소 |

---

## 컴포넌트 설계 원칙

1. **단방향 데이터 흐름**: 스토어 → composable → 컴포넌트 props → template
2. **Logic-less 컴포넌트**: 컴포넌트는 전달받은 props를 렌더링하는 것만 담당
3. **oilType props 전달**: `oilType`은 부모 컴포넌트에서 자식 컴포넌트로 prop으로 전달하여 강조 UI 분기에 사용
4. **로딩/에러 처리**: 각 뷰 컴포넌트에서 `loading`, `error` 상태를 처리하고 하위 컴포넌트에는 데이터만 전달

---

## 미확정 협의 사항

자세한 내용은 `docs/04_frontend/pending-agreements.md` 참조
