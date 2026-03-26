# 예측 시계열 UI 명세

## 문서 목적

이 문서는 ENerCY 프로젝트의 Chart.js 기반 예측 시계열 차트(`ForecastChart.vue`)의
렌더링 명세를 정의한다.

23일 예측 데이터를 두 구간(배송 대기 / 실질 타겟)으로 시각적으로 분리하여 표시하는
규칙을 포함한다.

관련 스킬 문서: `.codex/skills/frontend-development/SKILL.md`
데이터 출처: `GET /api/v1/forecast`

---

## 예측 구간 정의 (합의 사항)

| 구간 이름 | 범위 | 의미 |
|-----------|------|------|
| 배송 대기 구간 | Day 1 ~ Day 15 | 배송 리드타임 기간. 이 구간에 구매해도 아직 배송 중 |
| 실질 타겟 구간 | Day 16 ~ Day 23 | 실제 구매 결정에 반영되는 예측 구간 |

**중요**: 프론트엔드는 어느 포인트가 어느 구간에 속하는지를 **스스로 계산하지 않는다.**
백엔드 API가 각 포인트의 구간 소속을 식별자 필드로 제공한다.

→ 구간 식별자 필드: `periodType` (`"WAITING"` | `"TARGET"`). 각 `forecastPoints` 객체에 포함됨.

---

## 데이터 출처 및 바인딩

### 확정된 API 응답 구조

```json
{
  "forecastHorizon": 23,
  "forecastPoints": [
    { "time": "2026-03-26T00:00:00Z", "predictedPrice": 79.10, "periodType": "WAITING" },
    { "time": "2026-03-27T00:00:00Z", "predictedPrice": 80.50, "periodType": "TARGET" }
  ]
}
```

각 포인트의 `periodType` 필드로 구간을 식별한다.
- `"WAITING"`: Day 1~15 배송 대기 구간
- `"TARGET"`: Day 16~23 실질 타겟 구간

---

## API 필드 → 차트 바인딩

| API 필드 | Chart.js 바인딩 위치 | 처리 방식 |
|----------|---------------------|-----------|
| `forecastPoints[].time` | x축 레이블(`labels`) | ISO 8601 → `MM/DD` 형식 변환 (표시 형식만) |
| `forecastPoints[].predictedPrice` | y축 데이터값(`data`) | 값 그대로 사용 |
| `forecastHorizon` | 차트 총 포인트 수 참조 | 검증용 |
| `forecastPoints[].periodType` | 배송 대기 / 타겟 데이터셋 분리 | `"WAITING"` → 배송 대기, `"TARGET"` → 실질 타겟 |

---

## Chart.js 데이터셋 구성 명세

두 구간을 **별도 데이터셋**으로 구성하여 색상과 범례를 독립 관리한다.
`forecastPoints`를 `periodType` 필드로 필터링하여 각 데이터셋을 구성한다.

### 배송 대기 구간 데이터셋

| 속성 | 값 |
|------|----|
| `label` | `'배송 대기 구간 (Day 1~15)'` |
| `data` | `periodType === 'WAITING'`인 포인트의 `predictedPrice` 배열 |
| `borderColor` | `#78909C` (회색 계열) |
| `borderDash` | `[5, 5]` (점선 — 대기 구간임을 시각적으로 표현) |
| `backgroundColor` | `rgba(120, 144, 156, 0.1)` |
| `tension` | `0.3` |
| `pointRadius` | `3` |

### 실질 타겟 구간 데이터셋

| 속성 | 값 |
|------|----|
| `label` | `'실질 타겟 구간 (Day 16~23)'` |
| `data` | `periodType === 'TARGET'`인 포인트의 `predictedPrice` 배열 |
| `borderColor` | `#1976D2` (Vuetify primary 파란색) |
| `borderDash` | 없음 (실선) |
| `backgroundColor` | `rgba(25, 118, 210, 0.15)` |
| `tension` | `0.3` |
| `pointRadius` | `4` |

### x축 연속성 처리

두 데이터셋이 하나의 x축(`labels`)을 공유하므로,
각 데이터셋은 자신의 구간이 아닌 위치에 `null`을 채워 x축 정렬을 유지한다.
이 `null` 패딩은 값 가공이 아니라 Chart.js의 요구 형식에 맞추기 위한 구조 변환이다.

---

## chartjs-plugin-annotation 구간 분리 설정

`chartjs-plugin-annotation`을 사용하여 두 구간 경계에 시각적 분리 요소를 렌더링한다.

### 구간 경계 수직선

두 구간의 경계는 x축 위 Day 15와 Day 16 사이에 수직선으로 표시한다.
수직선의 x 위치는 `periodType`이 `"WAITING"`에서 `"TARGET"`으로 바뀌는 경계 포인트의 `time` 값을 기준으로 결정한다.

```
annotation 설정:

boundaryLine:
  type: 'line'
  xMin: [periodType WAITING→TARGET 경계 포인트의 time 값]
  xMax: [periodType WAITING→TARGET 경계 포인트의 time 값]
  borderColor: '#E53935'
  borderWidth: 2
  borderDash: [6, 4]
  label:
    content: '배송 리드타임 경계'
    position: 'start'
    color: '#E53935'
    font.size: 11
```

### 배경 영역 음영

각 구간에 배경 음영을 적용하여 구간 범위를 직관적으로 표현한다.

```
waitingZone (배송 대기 구간):
  type: 'box'
  xMin: [periodType 기반 구간 경계 time 값]
  xMax: [periodType 기반 구간 경계 time 값]
  backgroundColor: 'rgba(120, 144, 156, 0.08)'
  borderWidth: 0
  label.content: '배송 대기'

targetZone (실질 타겟 구간):
  type: 'box'
  xMin: [periodType 기반 구간 경계 time 값]
  xMax: [periodType 기반 구간 경계 time 값]
  backgroundColor: 'rgba(25, 118, 210, 0.05)'
  borderWidth: 0
  label.content: '타겟 구간'
```

---

## Chart.js 옵션 명세

| 옵션 | 값 | 비고 |
|------|----|------|
| `responsive` | `true` | |
| `maintainAspectRatio` | `false` | 컨테이너 높이 prop으로 제어 |
| `plugins.legend.position` | `'top'` | |
| `plugins.tooltip.callbacks.label` | `예측 유가: $XX.XX` 형식 | 소수점 2자리 |
| `scales.x.title.text` | `'예측 날짜'` | |
| `scales.y.title.text` | `'유가 (USD/bbl)'` | |

---

## ForecastChart.vue 컴포넌트 Props

| prop | 타입 | 필수 | 설명 |
|------|------|------|------|
| `forecastPoints` | `Array<{ time: string, predictedPrice: number, periodType: 'WAITING' \| 'TARGET' }>` | 필수 | `forecastPoints` 배열 전체 |
| `forecastHorizon` | `number` | 필수 | 총 예측 일수 (23) |
| `height` | `string` | 선택 | 차트 컨테이너 높이 (기본: `'400px'`) |

---

## ForecastView.vue 페이지 구성

`/forecast` 페이지는 아래 요소로 구성한다.

| 영역 | 컴포넌트 | 내용 |
|------|----------|------|
| 상단 정보 바 | `DecisionTimeCard.vue` | 판단 시점 + `oilType` 표시 텍스트 |
| 차트 영역 | `ForecastChart.vue` | 23일 예측 시계열 전체 |
| 하단 범례 설명 | 정적 텍스트 | 배송 대기 / 타겟 구간 의미 설명 |
| 상세 보기 링크 | `v-btn` | 대시보드(`/`)로 돌아가기 |

---

## 협의 완료 사항

| 항목 | 내용 | 결과 |
|------|------|------|
| PA-002 | 구간 식별자 필드 구조 | `periodType: "WAITING" \| "TARGET"` 방식 A 확정 |
