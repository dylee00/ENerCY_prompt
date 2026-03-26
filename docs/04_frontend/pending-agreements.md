# 프론트엔드 협의 대기 사항 (Pending Agreements)

## 문서 목적

이 문서는 프론트엔드 명세 작성 과정에서 타 도메인(주로 백엔드)과의 스펙 협의가
완료되지 않아 UI 바인딩을 확정할 수 없는 항목을 누적 관리한다.

각 항목이 협의 완료되면 결과를 기록하고 상태를 `완료`로 변경한다.
협의가 완료된 항목은 관련 명세 문서를 즉시 갱신하고 `[TBD]` 표시를 제거한다.

---

## 협의 안건 목록

(현재 없음)

---

## 완료된 협의 항목

---

### [PA-001] UI 강조 대상 표시용 메타 플래그 추가 요청

- **상태**: 완료 (프론트엔드 예외 처리)
- **관련 문서**: `SKILL.md`, `dashboard-spec.md`, `purchase-recommendation-ui-spec.md`
- **발생 배경**: 오일 종류(Crude Oil, Gasoline, Heating Oil 등)에 따라 최대 유가 또는 최소 유가를 강조해서 보여줘야 한다는 도메인 요구사항이 있다. 초기 명세에서는 프론트엔드가 `oilType` 문자열을 직접 비교하여 강조 대상을 결정하는 방식으로 설계되었으나, 이는 Logic-less UI 원칙 위반이다. 오일 종류가 추가되거나 정책이 변경될 때마다 프론트엔드 코드를 수정해야 하는 구조적 결함이다.

#### 협의 결과

백엔드 플래그 추가 대신, 프론트엔드에서 `oilType`에 따른 강조 로직을 예외적으로 수행하기로 함.

구현 로직:
- `oilType === 'Crude Oil'` → 최소 유가(`predictedMinPrice`) 카드 강조
- `oilType === 'Gasoline'` 또는 `'Heating Oil'` → 최대 유가(`predictedMaxPrice`) 카드 강조

**비고**: `Logic-less UI` 원칙의 유일한 예외 항목으로 관리한다.
`oilType` 값은 강조 분기 판단에만 사용하며, 다른 비즈니스 로직에는 사용하지 않는다.

---

### [PA-002] 예측 시계열 구간 식별자 필드 추가 요청

- **상태**: 완료 (방식 A 채택)
- **관련 문서**: `SKILL.md`, `forecast-ui-spec.md`
- **발생 배경**: 차트에서 Day 1~15(배송 대기 구간)와 Day 16~23(실질 타겟 구간)을 별도 데이터셋으로 분리하여 렌더링해야 한다. 초기 명세에서는 프론트엔드가 인덱스를 직접 잘라 구간을 구분하는 방식을 검토했으나, 이는 배송 리드타임이라는 비즈니스 정책을 프론트엔드에 하드코딩하는 구조적 결함이다.

#### 협의 결과

`GET /api/v1/forecast` 응답의 `forecastPoints` 각 객체에 `periodType` 필드 추가 (방식 A 채택).

확정된 데이터 구조:
```json
"forecastPoints": [
  { "time": "...", "predictedPrice": 79.10, "periodType": "WAITING" },
  { "time": "...", "predictedPrice": 80.50, "periodType": "TARGET" }
]
```

- `"WAITING"`: Day 1~15 배송 대기 구간
- `"TARGET"`: Day 16~23 실질 타겟 구간

---

### [PA-003] 대시보드 요약 가격의 산출 기준 명확화 요청

- **상태**: 완료 (해석 B 확정)
- **관련 문서**: `dashboard-spec.md`, `purchase-recommendation-ui-spec.md`
- **발생 배경**: `predictedMaxPrice` 및 `predictedMinPrice` 값이 어느 구간을 기준으로 산출된 값인지 API 명세에 명시되지 않았다. 전체 23일 기준인지, 실질 타겟 구간(Day 16~23)만 기준인지에 따라 UI의 의미가 달라진다.

#### 협의 결과

dashboard와 recommendation API에서 제공하는 Max/Min 가격은 **실질 타겟 구간(Day 16~23) 기준**으로 산출됨을 확정함.

UI 반영: 카드 부제목 등에 `"16~23일 예측 기준"` 문구 표기.

---

### [PA-004] 모니터링 구간별 성능 지표 응답 구조 확정 요청

- **상태**: 완료 (중첩 구조 사용)
- **관련 문서**: `monitoring-ui-spec.md`
- **발생 배경**: Day 1~15 구간과 Day 16~23 구간의 재학습 threshold가 각각 다르게 설정될 예정이므로, UI에서 두 구간의 성능 지표를 독립적으로 표시해야 한다. `evaluationMetrics` 내부 구조가 확정되지 않았다.

#### 협의 결과

`GET /api/v1/monitoring` 응답의 `latestEvaluation` 필드 내에 구간별 데이터를 중첩(nested) 객체로 제공함.

확정된 데이터 구조:
```json
"latestEvaluation": {
  "waitingPeriod": { "mae": 1.2, "rmse": 1.5 },
  "targetPeriod":  { "mae": 1.8, "rmse": 2.1 }
}
```

**비고**: 기존 `evaluationMetrics` 필드명 대신 `latestEvaluation`으로 확정됨.

---

## 갱신 규칙

- 명세 작성 중 새로운 협의 필요 사항이 발생하면 즉시 이 문서에 추가한다.
- 항목 번호는 `PA-NNN` 형식으로 순차 부여한다.
- 협의 완료 시: 상태를 `완료`로 변경하고, 합의 결과를 기록하고, 관련 명세 문서의 `[TBD]`를 제거한다.
