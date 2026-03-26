# retraining-trigger-policy.md

## 목적

이 문서는 LSTM 모델의 재학습이 언제, 어떤 조건에서 트리거되는지를 정의한다.
재학습 트리거 조건은 반드시 명시적으로 문서화되어야 한다. (AGENTS.md 절대 규칙)

현재 임계치 수치와 트리거 범위는 초안이며, docs/05_shared/init.md 합의 후 확정한다.

---

## 재학습 트리거 조건

아래 조건 중 하나라도 충족되면 재학습을 트리거한다.

### 조건 1. 성능 기반 트리거 (주요, 제안안)

- 단기(1~15일) RMSE > 3.5 (threshold-rule.md 참조)
- 중기(16~23일) RMSE > 5.5 (threshold-rule.md 참조)
- 위 조건 충족 시 → retrainingRequired: true, modelStatus: "CRITICAL"

> 위 수치는 임시 제안값이다. 구현 고정 전 합의가 필요하다.

### 조건 2. 데이터 기반 트리거

- 신규 CSV 업로드 시 기존 모델로 예측 수행 후 오차 계산
- 오차가 임계치 초과이면 자동 재학습 트리거

### 조건 3. 수동 트리거 (LLM 보고서 승인)

- LLM이 성능 지표 기반 보고서를 생성
- 사용자가 UI에서 재학습 승인 시 트리거
- 이 경우 RMSE 임계치와 무관하게 재학습 실행

---

## 재학습 실행 흐름

```
CSV 업로드
    │
    ▼
기존 모델로 예측 수행
    │
    ▼
실제값 vs 예측값 오차 계산 (RMSE, MAE)
    │
    ▼
임계치 판단 (threshold-rule.md 기준)
    │
   ┌┴──────────────────┐
   │                   │
RMSE ≤ 임계치       RMSE > 임계치
   │                   │
기존 모델 유지      재학습 트리거
   │                   │
신규 예측 수행      신규 모델 학습
   │                   │
의사결정 추천       MLflow 등록
   │                   │
LLM 보고서 생성     성능 비교 후 배포 여부 결정
                       │
                   LLM 보고서 생성
                       │
                   사용자 승인
                       │
                   신규 모델 배포
                       │
                   신규 예측 수행
```

---

## 재학습 실행 정책

- 재학습은 전체 학습 데이터(train+valid)를 사용한다.
- 재학습 시 동일한 하이퍼파라미터를 사용한다. (hyperparameter-policy.md 참조)
- 재학습된 모델은 test set 기준 성능이 기존 모델보다 나을 때만 배포한다.
- 성능 비교 기준: 단기 RMSE를 주 기준으로 사용한다.
- 재학습 결과는 MLflow에 별도 run으로 기록한다.

---

## 기존 모델 유지 조건

아래 경우에는 기존 모델을 유지한다.

- RMSE가 임계치 이하인 경우
- 재학습 후 신규 모델 성능이 기존 모델보다 나쁜 경우
- 사용자가 LLM 보고서 확인 후 재학습을 거부한 경우

---

## MLflow 기록

재학습 트리거 시 아래를 MLflow에 기록한다.

```python
mlflow.log_param("trigger_reason", "rmse_threshold_exceeded")
mlflow.log_param("trigger_horizon", "mid_term")  # 또는 "short_term"
mlflow.log_metric("trigger_rmse", rmse_value)
mlflow.set_tag("retraining_triggered", "true")
```

---

## 관련 문서

- threshold-rule.md
- evaluation-metrics.md
- mlflow-tracking-policy.md
- model-selection-policy.md
- docs/03_backend/init.md
