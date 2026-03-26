# performance-monitoring-spec.md

## 목적

이 문서는 배포된 LSTM 모델의 성능을 지속적으로 모니터링하는 방법을 정의한다.

---

## 모니터링 시점

| 시점 | 설명 |
|------|------|
| CSV 업로드 시 | 사용자가 신규 데이터 업로드할 때마다 평가 수행 |
| 배포 후 정기 평가 | 신규 실제값이 확보되면 예측값과 비교하여 오차 계산 |

---

## 모니터링 흐름

```
신규 데이터 유입 (CSV 업로드)
    │
    ▼
현재 Production 모델로 예측 수행
    │
    ▼
예측값 저장 (date, oil_type, predicted_price, horizon_day)
    │
    ▼
실제값 확보 시점에 오차 계산
    │
    ▼
RMSE, MAE, MAPE 계산
    │
    ▼
threshold-rule.md 기준으로 상태 판정
    │
   ┌┴─────────────────┐
HEALTHY/WARNING     CRITICAL
   │                   │
모델 유지           재학습 트리거
   │                   │
상태 API 응답       retraining-trigger-policy.md
```

---

## 모니터링 상태 응답

백엔드 API `/api/v1/monitoring` 응답과 연동된다.

> 필드명은 v0 제안이며, 최종 네이밍은 도메인 합의 후 확정한다.

```json
{
  "modelStatus": "HEALTHY",
  "retrainingRequired": false,
  "latestModelVersion": "lstm-v1.3.2",
  "latestEvaluation": {
    "shortTerm": { "rmse": 2.41, "mae": 1.82, "mape": 2.3 },
    "midTerm":   { "rmse": 4.10, "mae": 3.20, "mape": 3.8 }
  },
  "message": "현재 모델 성능은 안정 상태입니다."
}
```

### message 텍스트 정의

| modelStatus | message |
|-------------|---------|
| HEALTHY | "현재 모델 성능은 안정 상태입니다." |
| WARNING | "모델 성능이 경고 수준입니다. 모니터링을 강화하세요." |
| CRITICAL | "모델 성능이 임계치를 초과했습니다. 재학습이 필요합니다." |

---

## LLM 보고서 연동

- modelStatus가 WARNING 이상이면 LLM 보고서 생성을 권장한다.
- LLM은 아래 정보를 받아 자연어 보고서를 생성한다.
  - 현재 RMSE, MAE 수치
  - 임계치 대비 현재 상태
  - 단기/중기 성능 비교
  - 재학습 권장 여부 및 이유
- 사용자가 보고서 확인 후 재학습 승인/거부를 결정한다.

---

## 관련 문서

- threshold-rule.md
- retraining-trigger-policy.md
- evaluation-metrics.md
- docs/03_backend/init.md
