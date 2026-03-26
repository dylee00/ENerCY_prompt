# hyperparameter-policy.md

## 목적

이 문서는 LSTM 모델의 하이퍼파라미터 정책을 정의한다.
유종(오일 종류)에 상관없이 동일한 하이퍼파라미터 세트를 사용한다. (AGENTS.md 절대 규칙)

단, 이번 라운드에서는 일부 항목이 아직 미정이므로 아래 값은 "초기 제안값"으로 취급한다.

---

## 핵심 원칙

- 하이퍼파라미터는 유종별로 따로 두지 않는다.
- 모든 유종(CL=F, RB=F, HO=F)에 동일한 값을 적용한다.
- 임계치 및 설정값은 코드 내 매직 넘버로 두지 않는다.
- 설정 파일(config) 또는 MLflow params로 관리하여 추적 가능하게 유지한다.
- 미정 항목(sequence_length, horizon)은 팀 합의 전까지 고정값으로 확정하지 않는다.

---

## 하이퍼파라미터 정의

| 파라미터 | 초기 제안값(미확정) | 설명 |
|----------|---------------------|------|
| sequence_length | 60 | 입력 시퀀스 길이 제안값 (합의 전) |
| lstm_units_1 | 64 | 첫 번째 LSTM 레이어 유닛 수 제안값 |
| lstm_units_2 | 32 | 두 번째 LSTM 레이어 유닛 수 제안값 |
| dropout_rate | 0.2 | Dropout 비율 제안값 |
| forecast_horizon_short | 15 | 단기 예측 시점 수 제안값 |
| forecast_horizon_mid | 8 | 중기 예측 시점 수 제안값 |
| batch_size | 32 | 배치 크기 제안값 |
| epochs | 50 | 최대 학습 에폭 수 제안값 |
| learning_rate | 0.001 | Adam optimizer 학습률 제안값 |
| early_stopping_patience | 5 | Early stopping 기준 제안값 |

> 위 값은 초기 제안값이며 아직 확정값이 아니다.
> 확정 시 docs/05_shared/init.md와 docs/05_shared/decision-log.md에 반영한다.

---

## 설정 파일 관리 방식

하이퍼파라미터는 아래 방식으로 관리한다.

```yaml
# config/model_config.yaml 예시
model:
  sequence_length: 60
  lstm_units_1: 64
  lstm_units_2: 32
  dropout_rate: 0.2
  forecast_horizon_short: 15
  forecast_horizon_mid: 8
  batch_size: 32
  epochs: 50
  learning_rate: 0.001
  early_stopping_patience: 5
```

- 코드에서 직접 숫자를 사용하지 않고 config를 로드해서 사용한다.
- MLflow 실험 시 params로 자동 기록된다.

---

## MLflow 연동

- 학습 시작 전 모든 하이퍼파라미터를 `mlflow.log_params()`로 기록한다.
- 실험별 하이퍼파라미터 변경 이력이 MLflow UI에서 추적 가능해야 한다.

---

## 관련 문서

- lstm-modeling-spec.md
- mlflow-tracking-policy.md
- docs/05_shared/decision-log.md
