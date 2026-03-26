# 모니터링 UI 명세

## 문서 목적

이 문서는 ENerCY 프로젝트의 시스템 모니터링 페이지(`MonitoringView.vue`)의
레이아웃, 컴포넌트 구성, API 데이터 바인딩 규칙을 정의한다.

관련 스킬 문서: `.codex/skills/frontend-development/SKILL.md`
데이터 출처: `GET /api/v1/monitoring`

---

## Logic-less 원칙 준수 선언

이 화면에서 프론트엔드가 직접 수행하는 판단은 없다.

| 금지 항목 | 이유 |
|-----------|------|
| MAE, RMSE 수치를 보고 재학습 필요 여부를 프론트엔드에서 판단하는 것 | 재학습 정책은 백엔드 책임 |
| 성능 지표 값을 임계치와 비교하여 경고를 생성하는 것 | threshold 정책 침범 |
| `modelStatus` 이외의 기준으로 모델 상태를 판단하는 것 | 도메인 로직 침범 |

모든 상태 판단 결과는 백엔드 API 응답 필드를 통해 수신하고,
프론트엔드는 이를 화면에 바인딩하여 표시만 한다.

---

## 데이터 출처

### 확정된 API 응답 구조

```json
{
  "modelStatus": "HEALTHY",
  "retrainingRequired": false,
  "latestModelVersion": "lstm-v1.3.2",
  "latestEvaluation": {
    "waitingPeriod": { "mae": 1.2, "rmse": 1.5 },
    "targetPeriod":  { "mae": 1.8, "rmse": 2.1 }
  }
}
```

### `latestEvaluation` 내부 구조

두 예측 구간의 성능 지표를 중첩(nested) 객체로 제공한다.

| 필드 | 내용 |
|------|------|
| `latestEvaluation.waitingPeriod.mae` | Day 1~15 구간 MAE |
| `latestEvaluation.waitingPeriod.rmse` | Day 1~15 구간 RMSE |
| `latestEvaluation.targetPeriod.mae` | Day 16~23 구간 MAE |
| `latestEvaluation.targetPeriod.rmse` | Day 16~23 구간 RMSE |

---

## 레이아웃 구조

```
┌────────────────────────────────────────────────────────────┐
│  [앱 바] 시스템 모니터링  |  모델 버전 표시                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  [A] 재학습 경고 배너 (retrainingRequired = true 시 표시)   │
│                                                            │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  [B] 모델 전체 상태 카드 (modelStatus 배지)                 │
│                                                            │
├────────────────────┬───────────────────────────────────────┤
│                    │                                       │
│  [C] Day 1~15      │  [D] Day 16~23                        │
│  성능 지표 패널      │  성능 지표 패널                        │
│  (MetricsPanel)    │  (MetricsPanel)                       │
│                    │                                       │
└────────────────────┴───────────────────────────────────────┘
```

Vuetify 구현 기준:
- [A]: `v-banner` (전체 너비, 조건부 표시)
- [B]: `v-row` > `v-col cols="12"`
- [C][D]: `v-row` > `v-col cols="6"` 각각

---

## 영역별 컴포넌트 및 API 바인딩 상세

### [앱 바] 상단 표시

| UI 요소 | 컴포넌트 | API 필드 | 표시 형식 |
|---------|----------|---------|-----------|
| 모델 버전 | 보조 텍스트 | `latestModelVersion` | `모델 버전: lstm-v1.3.2` |

---

### [A] 재학습 경고 배너

`retrainingRequired` 값은 백엔드가 재학습 필요 여부를 판단하여 내려주는 결과값이다.
프론트엔드는 이 값이 `true`일 때 배너를 표시하고, `false`일 때 숨긴다.
재학습이 필요한지 여부를 프론트엔드가 직접 판단하지 않는다.

| UI 요소 | 컴포넌트 | API 필드 | 렌더링 조건 |
|---------|----------|---------|------------|
| 경고 배너 | `v-banner` | `retrainingRequired` | `true`일 때만 표시 (`v-if`) |
| 배너 아이콘 | `v-icon` | — | `mdi-alert-circle` (경고 아이콘) |
| 배너 색상 | — | — | Vuetify `color="warning"` |
| 배너 문구 | 정적 텍스트 | — | `"모델 재학습이 필요합니다. 백엔드 운영팀에 문의하세요."` |

---

### [B] 모델 전체 상태 카드

`modelStatus` 값은 백엔드가 내려주는 상태 열거값이다.
프론트엔드는 이 값을 읽어 배지 색상과 텍스트를 전환한다.
모델 상태가 양호한지 여부를 프론트엔드가 판단하지 않는다.

| `modelStatus` 값 | 배지 색상 | 표시 텍스트 |
|-----------------|-----------|------------|
| `HEALTHY` | Vuetify `success` (녹색) | `정상` |
| `DEGRADED` | Vuetify `warning` (주황) | `성능 저하` |
| `RETRAINING_NEEDED` | Vuetify `error` (붉은색) | `재학습 필요` |
| 그 외 알 수 없는 값 | Vuetify `default` (회색) | 수신된 값 그대로 표시 |

**설계 원칙**: 위 열거값 목록은 현재 알려진 값 기준이다.
백엔드가 새로운 `modelStatus` 값을 추가할 경우,
프론트엔드는 알 수 없는 값을 회색 배지로 폴백 처리하여 UI가 깨지지 않도록 한다.

| UI 요소 | 컴포넌트 | API 필드 |
|---------|----------|---------|
| 상태 배지 | `StatusBadge.vue` | `modelStatus` |
| 마지막 갱신 텍스트 | 보조 텍스트 | — (데이터 페칭 시각 기준, 별도 API 필드 없음) |

---

### [C] Day 1~15 성능 지표 패널 / [D] Day 16~23 성능 지표 패널

두 구간의 성능 지표는 **독립된 `MetricsPanel.vue` 인스턴스** 두 개로 렌더링한다.
각 패널은 동일한 컴포넌트를 사용하되, 서로 다른 구간 데이터를 props로 받는다.

두 패널의 레이아웃과 컴포넌트 구조는 대칭으로 설계하여,
한 구간의 지표가 이상할 때 다른 구간 지표와 나란히 비교할 수 있도록 한다.

#### `MetricsPanel.vue` Props

| prop | 타입 | 필수 | 설명 |
|------|------|------|------|
| `periodLabel` | `string` | 필수 | 패널 타이틀 표시용 구간 이름 |
| `metrics` | `object \| null` | 필수 | 해당 구간의 성능 지표 객체. 구조는 [TBD: PA-004] |

#### 바인딩 (PA-004 완료 후 갱신)

| 패널 | `periodLabel` prop | `metrics` prop 바인딩 |
|------|-------------------|-----------------------|
| [C] Day 1~15 패널 | `'Day 1 ~ 15 (배송 대기 구간)'` | `latestEvaluation.waitingPeriod` |
| [D] Day 16~23 패널 | `'Day 16 ~ 23 (실질 타겟 구간)'` | `latestEvaluation.targetPeriod` |

#### `MetricsPanel.vue` 내부 표시 항목

| UI 요소 | 항목명 | 바인딩 필드 |
|---------|--------|------------|
| 패널 타이틀 | 구간 이름 | `periodLabel` prop |
| MAE 수치 | 평균 절대 오차 | `metrics.mae` |
| RMSE 수치 | 평균 제곱근 오차 | `metrics.rmse` |
| 수치 표시 형식 | — | 소수점 2자리 (`X.XX`) |

---

## `ModelStatusPanel.vue` 컴포넌트 설계

`[B]` 상태 카드를 담당하는 컴포넌트다.

### Props

| prop | 타입 | 필수 | 설명 |
|------|------|------|------|
| `modelStatus` | `string` | 필수 | 모델 상태 열거값 |
| `latestModelVersion` | `string` | 필수 | 모델 버전 문자열 |

### 컴포넌트 책임 경계

- `modelStatus` 값에 따른 배지 색상·텍스트 전환만 수행한다.
- 모델이 정상인지 비정상인지 판단하는 로직을 내부에 두지 않는다.
- 알 수 없는 `modelStatus` 값이 수신되면 회색 폴백 처리한다.

---

## API 데이터 매핑 전체 요약

```
GET /api/v1/monitoring 응답
│
├── modelStatus          → [B] 상태 배지 색상 및 텍스트
├── retrainingRequired   → [A] 경고 배너 표시 여부 (true → 표시, false → 숨김)
├── latestModelVersion   → [앱 바] 모델 버전 텍스트
└── latestEvaluation
    ├── waitingPeriod    → [C] Day 1~15 MetricsPanel.metrics prop
    └── targetPeriod     → [D] Day 16~23 MetricsPanel.metrics prop
```

---

## 협의 완료 사항

| 항목 | 내용 | 결과 |
|------|------|------|
| PA-004 | 성능 지표 응답 구조 | `latestEvaluation.waitingPeriod` / `targetPeriod` 중첩 구조 확정 |
