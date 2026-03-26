# mlflow-tracking-policy.md

## 목적

이 문서는 MLflow를 활용한 실험 추적 및 모델 관리 정책을 정의한다.

현재 일부 태그/지표 필드는 초안이며, docs/05_shared/init.md 기준 합의 후 확정한다.

---

## MLflow 역할

- 모든 학습 실험의 파라미터, 지표, 아티팩트를 추적한다.
- 모델 버전을 관리하고 배포 상태를 기록한다.
- 백엔드는 MLflow에서 최신 모델 정보와 성능 지표를 조회한다.

---

## 실험(Experiment) 구조

유종별로 실험을 분리한다.

| 실험 이름 | 대상 유종 |
|-----------|-----------|
| energy-forecast-CL | Crude Oil (CL=F) |
| energy-forecast-RB | RBOB Gasoline (RB=F) |
| energy-forecast-HO | Heating Oil (HO=F) |

---

## Run 기록 항목

### Parameters (학습 시작 전 기록)

```python
mlflow.log_params({
    "oil_type": "CL=F",
    "sequence_length": 60,
    "lstm_units_1": 64,
    "lstm_units_2": 32,
    "dropout_rate": 0.2,
    "forecast_horizon_short": 15,
    "forecast_horizon_mid": 8,
    "batch_size": 32,
    "epochs": 50,
    "learning_rate": 0.001,
    "trigger_reason": "manual" # 또는 "rmse_threshold_exceeded"
})
```

### Metrics (학습/평가 후 기록)

```python
mlflow.log_metrics({
    "rmse_short": ...,
    "mae_short": ...,
    "mape_short": ...,
    "rmse_mid": ...,
    "mae_mid": ...,
    "mape_mid": ...,
    "val_loss": ...,
    "train_loss": ...
})
```

### Tags

```python
mlflow.set_tags({
    "model_type": "LSTM",
    "horizon_type": "multi-step",
    "target_type": "low",  # 또는 "high", "close" (유종 정책에 따름)
    "retraining_triggered": "false",  # 또는 "true"
    "deployed": "false"
})
```

### Artifacts

- 학습된 모델 파일 (`.h5` 또는 `SavedModel`)
- 스케일러 파일 (`scaler.pkl`)
- 학습 곡선 이미지
- 예측 vs 실제값 비교 플롯

---

## 모델 레지스트리

- 학습 완료 후 성능이 기존 모델보다 나으면 Model Registry에 등록한다.
- 등록 단계: None → Staging → Production
- Production 상태의 모델만 실제 서빙에 사용한다.

```
신규 학습 완료
    │
    ▼
test RMSE 비교 (신규 vs 기존 Production)
    │
신규 모델 성능이 더 좋으면
    │
    ▼
Model Registry → Staging 등록
    │
사용자 승인 (LLM 보고서 확인 후)
    │
    ▼
Production 승격
```

---

## 백엔드 연동 방식 (MA-003 제안: 폴링 방식)

백엔드는 MLflow에서 직접 지표를 폴링하여 조회한다.
모델은 MLflow에 기록만 하고, 백엔드가 필요할 때 읽어가는 구조다.
모델이 백엔드를 직접 호출하지 않는다. (도메인 분리 원칙 준수)

```python
# Production 모델 로드
model = mlflow.pyfunc.load_model("models:/energy-forecast-CL/Production")

# 최신 run 지표 조회
client = mlflow.tracking.MlflowClient()
latest_run = client.search_runs(experiment_ids=[...], order_by=["start_time DESC"])[0]
rmse_short = latest_run.data.metrics["rmse_short"]
rmse_mid = latest_run.data.metrics["rmse_mid"]
```

---

## 관련 문서

- hyperparameter-policy.md
- evaluation-metrics.md
- model-selection-policy.md
- retraining-trigger-policy.md
