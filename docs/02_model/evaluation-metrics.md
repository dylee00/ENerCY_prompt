# evaluation-metrics.md

## 목적

이 문서는 LSTM 모델 성능 평가에 사용하는 지표와 기준을 정의한다.
재학습 트리거 판단의 기준이 되므로, threshold-rule.md와 함께 관리한다.

---

## 기본 평가 지표

| 지표 | 설명 | 사용 목적 |
|------|------|-----------|
| RMSE | Root Mean Squared Error | 주요 재학습 트리거 기준 |
| MAE | Mean Absolute Error | 보조 모니터링 지표 |
| MAPE | Mean Absolute Percentage Error | 가격 수준 무관 상대 오차 파악 |

### 지표 계산 방식

```
RMSE = sqrt(mean((y_pred - y_true)^2))
MAE  = mean(|y_pred - y_true|)
MAPE = mean(|y_pred - y_true| / |y_true|) * 100
```

---

## Horizon별 평가 분리

단기와 중기 예측은 성격이 다르므로 각각 별도로 평가한다.

| 구간 | 예측 시점 | 평가 지표 | 비고 |
|------|-----------|-----------|------|
| 단기 | 1~15일 후 | RMSE, MAE, MAPE | 상대적으로 엄격한 기준 |
| 중기 | 16~23일 후 | RMSE, MAE, MAPE | 단기보다 완화된 기준 적용 |

- 중기 오차가 단기보다 크게 나타나는 것은 자연스러운 현상이다.
- 중기 임계치를 단기와 같이 설정하면 불필요한 재학습이 반복될 수 있다.

---

## 유종별 평가

- 동일한 지표를 모든 유종에 적용한다.
- 단, 유종별 가격 스케일이 다르므로 MAPE를 병행 확인한다.
  - Crude Oil: 달러/배럴 단위
  - Gasoline, Heating Oil: 달러/갤런 단위 (상대적으로 낮은 절대값)

---

## 평가 기준 임계치

임계치 상세 값은 threshold-rule.md에서 관리한다.
아래는 개념적 분류만 정의한다.

| 상태 | 의미 | API modelStatus 값 |
|------|------|--------------------|
| HEALTHY | RMSE가 임계치 이하 | "HEALTHY" |
| WARNING | RMSE가 경고 임계치 초과 | "WARNING" |
| CRITICAL | RMSE가 재학습 임계치 초과 | "CRITICAL" |

---

## MLflow 기록 대상

학습/평가 시 아래 지표를 MLflow에 기록한다.

```python
mlflow.log_metric("rmse_short", rmse_short)
mlflow.log_metric("mae_short", mae_short)
mlflow.log_metric("mape_short", mape_short)
mlflow.log_metric("rmse_mid", rmse_mid)
mlflow.log_metric("mae_mid", mae_mid)
mlflow.log_metric("mape_mid", mape_mid)
```

- 유종별로 실험 run을 분리하거나 태그로 구분한다.
- 평가는 반드시 test set 기준으로 수행한다.

---

## 백엔드 연동 (MA-002 제안: 방식 A 중첩 구조)

API `/api/v1/monitoring` 응답에 아래 구조로 제공된다.

```json
"latestEvaluation": {
  "shortTerm": { "rmse": 2.41, "mae": 1.82, "mape": 2.3 },
  "midTerm":   { "rmse": 4.10, "mae": 3.20, "mape": 3.8 }
}
```

- 모델은 MLflow에 단기/중기 지표를 각각 기록한다.
- 백엔드는 MLflow에서 폴링하여 위 구조로 변환해 응답한다.
- 응답 필드명은 현재 제안안이며, 최종 네이밍은 도메인 합의 후 확정한다.

---

## 관련 문서

- threshold-rule.md
- retraining-trigger-policy.md
- mlflow-tracking-policy.md
- docs/03_backend/init.md
