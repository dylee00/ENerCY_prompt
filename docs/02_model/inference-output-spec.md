# inference-output-spec.md

## 목적

이 문서는 LSTM 모델의 추론 출력 구조를 정의한다.
백엔드가 이 출력을 받아 API 응답으로 변환하므로, 백엔드 계약과 반드시 일치해야 한다.

현재 버전은 v0 초안이며, 필드명은 docs/05_shared/init.md의 미정 항목 합의 후 확정한다.

---

## 출력 구조 원칙

- 모델 출력은 미래 예측 시점들 간 비교가 가능해야 한다. (AGENTS.md 절대 규칙)
- 시스템은 아래 값을 계산 가능해야 한다.
  - 미래 예측 구간 내 최대 예측 유가
  - 미래 예측 구간 내 최소 예측 유가
  - 현재 시점의 구매 판단 근거

---

## 추론 입력

```python
{
  "oil_type": "CL=F",           # 유종 티커
  "sequence": [...],            # 최근 sequence_length(60)일치 피처 배열
  "inference_date": "2026-03-25"  # 추론 기준일
}
```

---

## 추론 출력

모델은 아래 구조의 딕셔너리를 반환한다.

> 아래 스키마는 제안안(v0)이다. 구현 시 임의 확정하지 않는다.

```python
{
  "oil_type": "CL=F",
  "inference_date": "2026-03-25",
  "forecast_points": [
    # 단기 구간 (1~15일): WAITING
    {"date": "2026-03-26", "predicted_price": 78.50, "horizon_day": 1,  "period_type": "WAITING"},
    # ... (2~14일 생략)
    {"date": "2026-04-09", "predicted_price": 77.80, "horizon_day": 15, "period_type": "WAITING"},
    # 중기 구간 (16~23일): TARGET
    {"date": "2026-04-10", "predicted_price": 76.80, "horizon_day": 16, "period_type": "TARGET"},
    {"date": "2026-04-11", "predicted_price": 77.20, "horizon_day": 17, "period_type": "TARGET"},
    {"date": "2026-04-12", "predicted_price": 78.50, "horizon_day": 18, "period_type": "TARGET"},
    {"date": "2026-04-13", "predicted_price": 79.10, "horizon_day": 19, "period_type": "TARGET"},
    {"date": "2026-04-14", "predicted_price": 80.30, "horizon_day": 20, "period_type": "TARGET"},
    {"date": "2026-04-15", "predicted_price": 81.00, "horizon_day": 21, "period_type": "TARGET"},
    {"date": "2026-04-16", "predicted_price": 79.50, "horizon_day": 22, "period_type": "TARGET"},
    {"date": "2026-04-17", "predicted_price": 78.20, "horizon_day": 23, "period_type": "TARGET"}
  ],
  "summary": {
    # 최대/최소 계산 구간은 TARGET(16~23일) 제안안 기준
    "predicted_max_price": 81.00,
    "predicted_max_date": "2026-04-15",
    "predicted_min_price": 76.80,
    "predicted_min_date": "2026-04-10"
  },
  "target_type": "low"   # 유종별 예측 목표: "low" | "high" | "close"
}
```

---

## target_type 정의

target_type은 모델 학습 타깃을 나타내는 내부 속성이다.
최종 recommendation 결과 단계(BUY/HOLD 등)는 백엔드 정책에서 별도로 결정한다.

| 유종 | target_type | 설명 |
|------|-------------|------|
| CL=F (Crude Oil) | "low" | 저가 예측 → 구매 추천에 활용 |
| RB=F (RBOB Gasoline) | "high" | 고가 예측 → 시세 판단 보조 지표 |
| HO=F (Heating Oil) | "high" | 고가 예측 → 시세 판단 보조 지표 |
| NG=F (Natural Gas, 선택) | "close" | 흐름 모니터링 목적의 종가 예측 |

---

## 백엔드 변환 책임

백엔드는 이 출력을 받아 아래 API 응답 필드로 변환한다.

| 모델 출력 필드 | API 응답 필드 |
|----------------|---------------|
| forecast_points[].predicted_price | forecastPoints[].predictedPrice |
| forecast_points[].date | forecastPoints[].time |
| summary.predicted_max_price | forecastSummary.predictedMaxPrice |
| summary.predicted_min_price | forecastSummary.predictedMinPrice |

- 모델은 원시 예측값을 반환한다.
- recommendation 최종 판단은 백엔드에서 수행한다.
- 모델은 추천 판단을 하지 않는다.

---

## 스케일 복원 주의사항

- 모델 학습 시 가격은 정규화(MinMaxScaler 등)되어 있다.
- 추론 출력은 반드시 원래 가격 스케일로 역변환(inverse_transform)하여 반환한다.
- 백엔드에 전달되는 predicted_price는 실제 가격 단위여야 한다.

---

## 관련 문서

- lstm-modeling-spec.md
- docs/03_backend/init.md
- docs/05_shared/init.md
