# lstm-modeling-spec.md

## 목적

이 문서는 에너지 선물 가격 예측에 사용하는 LSTM 모델의 구조와 설계 원칙을 정의한다.

---

## 모델 개요

- 모델 종류: LSTM (Long Short-Term Memory)
- 적용 대상 유종: Crude Oil (CL=F), RBOB Gasoline (RB=F), Heating Oil (HO=F)
- 하이퍼파라미터: 유종 구분 없이 동일한 세트 적용 (hyperparameter-policy.md 참조)

---

## 예측 목표 정의

유종별 예측 대상 가격이 다르다.

| 유종 | 예측 목표 | 비즈니스 목적 |
|------|-----------|---------------|
| Crude Oil (CL=F) | low (저가) | 싸게 구매하기 위한 최적 구매 시점 탐색 |
| RBOB Gasoline (RB=F) | high (고가) | 가격 상방 리스크 파악을 위한 보조 시계열 |
| Heating Oil (HO=F) | high (고가) | 가격 상방 리스크 파악을 위한 보조 시계열 |

---

## 예측 Horizon 설계

### 비즈니스 배경
- 현재 구매 결정 시점 기준으로 실제 배송까지 약 15일 소요
- 따라서 16일~23일 후 가격 예측이 실질적인 의사결정에 의미 있음

### Horizon 분리 정책
시계열 예측은 예측 시점이 멀어질수록 정확도가 급격히 낮아지는 특성이 있다.
이를 고려하여 horizon을 두 구간으로 분리하고, 각각 별도 재학습 임계치를 적용한다.

| 구간 | 예측 시점 | 용도 | 재학습 임계치 |
|------|-----------|------|---------------|
| 단기 (short-term) | 1일 ~ 15일 후 | 모델 성능 모니터링 기준 | threshold-rule.md 참조 |
| 중기 (mid-term) | 16일 ~ 23일 후 | 실제 구매 의사결정 보조 기준 | threshold-rule.md 참조 (단기보다 완화된 기준 적용) |

- 중기 horizon은 단기보다 예측 오차가 크게 나타나는 것이 자연스럽다.
- 따라서 중기 재학습 임계치는 단기보다 높게 설정한다.

### 의사결정 출력
- 16~23일 예측 구간 내 **최저가 날짜** → Crude Oil 구매 추천 기준
- 16~23일 예측 구간 내 **최고가 날짜** → 시세 판단 보조 지표
- 최대/최소 가격 산출 기준: 실질 타겟 구간(16~23일)만 사용 (D-014 참조)

### Horizon 제안
- 모델은 전체 23개 시점(1~23일)을 한 번에 출력한다.
- 단기(1~15일): periodType = "WAITING"
- 중기(16~23일): periodType = "TARGET"
- horizon 기본 제안값: 23 (합의 전 초안)

> 주의: horizon 길이와 구간 분리는 docs/05_shared/init.md의 미정 항목으로 관리한다.
> 구현 시점에 임의 확정하지 않고 팀 합의 후 고정한다.

---

## LSTM 구조

```
입력 레이어
  └─ shape: (sequence_length, n_features)

LSTM 레이어 1
  └─ units: hyperparameter-policy.md 참조
  └─ return_sequences: True

Dropout 레이어 1

LSTM 레이어 2
  └─ units: hyperparameter-policy.md 참조
  └─ return_sequences: False

Dropout 레이어 2

Dense 출력 레이어
  └─ units: forecast_horizon (예측 시점 수)
  └─ activation: linear
```

- 출력은 예측 시점 수(forecast_horizon)만큼의 가격 시퀀스
- 단기/중기 모델을 각각 별도 학습하거나, 전체 horizon을 한 번에 출력하는 multi-step 방식 선택 가능
- 최종 방식은 실험 결과 기반으로 결정하며 decision-log.md에 기록
- recommendation 최종 판단은 백엔드가 수행하며, 모델은 예측값과 요약 근거만 제공한다.

---

## 입력 데이터 구조

- 데이터 에이전트(docs/01_data/)가 전처리한 결과를 입력으로 받음
- 입력 피처 목록과 컬럼 정의는 docs/01_data/init.md에서 합의된 기준을 따른다.
- sequence_length(입력 윈도우 크기)는 hyperparameter-policy.md 참조

---

## 학습/검증/테스트 분리

- train/valid/test 분리 기준은 docs/01_data/init.md의 합의안을 따른다.
- 시계열 특성상 시간 순서를 반드시 유지한다 (shuffle 금지)

---

## 관련 문서

- hyperparameter-policy.md
- evaluation-metrics.md
- inference-output-spec.md
- threshold-rule.md
- docs/01_data/init.md
- docs/05_shared/init.md
- docs/05_shared/decision-log.md
