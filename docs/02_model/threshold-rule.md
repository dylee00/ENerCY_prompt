# threshold-rule.md

## 목적

이 문서는 모델 성능 모니터링 및 재학습 트리거에 사용하는 임계치를 정의한다.
임계치는 코드 내 매직 넘버로 두지 않으며, 이 문서와 config 파일에서 추적 가능하게 관리한다.

현재 수치와 정책 형태는 초안이며, docs/05_shared/init.md의 미정 항목 합의 후 확정한다.

---

## 핵심 원칙

- 임계치는 단기(1~15일)와 중기(16~23일) 구간을 분리하여 설정한다.
- 중기 임계치는 단기보다 완화된 값을 적용한다. (예측 horizon이 멀수록 오차가 자연스럽게 커짐)
- 임계치 변경 시 반드시 decision-log.md에 이유를 기록한다.

---

## RMSE 기반 임계치 (주요 기준)

아래 수치는 "임시 제안값"이다.

### 단기 예측 (1~15일 후)

| 상태 | RMSE 기준 | modelStatus | retrainingRequired |
|------|-----------|-------------|---------------------|
| HEALTHY | RMSE ≤ 2.0 | "HEALTHY" | false |
| WARNING | 2.0 < RMSE ≤ 3.5 | "WARNING" | false |
| CRITICAL | RMSE > 3.5 | "CRITICAL" | true |

### 중기 예측 (16~23일 후)

| 상태 | RMSE 기준 | modelStatus | retrainingRequired |
|------|-----------|-------------|---------------------|
| HEALTHY | RMSE ≤ 3.5 | "HEALTHY" | false |
| WARNING | 3.5 < RMSE ≤ 5.5 | "WARNING" | false |
| CRITICAL | RMSE > 5.5 | "CRITICAL" | true |

> 위 수치는 초기 제안값이며 확정값이 아니다.
> 실험/합의 후 확정 시 docs/05_shared/decision-log.md에 기록한다.

---

## modelStatus 판정 로직

단기와 중기 중 더 심각한 상태를 최종 modelStatus로 사용한다.

```
단기 → WARNING, 중기 → HEALTHY  → 최종: WARNING
단기 → HEALTHY, 중기 → CRITICAL → 최종: CRITICAL
단기 → CRITICAL, 중기 → WARNING → 최종: CRITICAL
```

retrainingRequired는 단기 또는 중기 중 하나라도 CRITICAL이면 true가 된다.

---

## 유종별 임계치 적용

- 모든 유종에 동일한 임계치를 적용한다. (AGENTS.md 절대 규칙)
- 단, 유종별 가격 스케일 차이를 고려하여 MAPE 지표를 병행 모니터링한다.

---

## recommendation 임계치 (초안)

recommendation 최종 판단은 백엔드에서 수행한다.
다만 기준 방식은 아래 셋 중 어느 형태로 확정할지 아직 미정이다.

- 절대값 기준
- 상대값 기준
- 절대/상대 혼합 기준

아래 표는 구매 추천 중심의 상대값 방식 예시 제안안이다.

| 유종 유형 | 제안 기준(미확정) | 설명 |
|-----------|------------------|------|
| 구매형 (Crude, Brent) | 예측 최저가 ≤ 현재가 × 0.97 | 예측 최저가가 현재가보다 3% 이상 낮으면 BUY, 아니면 HOLD 제안 |

판매형/다단계 recommendation은 이번 라운드 미정 범위로 두고, docs/05_shared/init.md 합의 후 확정한다.

> 3% 기준은 예시 제안값이다. 합의 전까지 구현 고정값으로 사용하지 않는다.

---

## config 파일 예시

```yaml
# config/threshold_config.yaml
threshold:
  short_term:
    rmse_healthy: 2.0
    rmse_warning: 3.5
  mid_term:
    rmse_healthy: 3.5
    rmse_warning: 5.5
  recommendation:
    buy_discount_rate: 0.03
    hold_threshold_buffer: 0.00
```

---

## 관련 문서

- evaluation-metrics.md
- retraining-trigger-policy.md
- docs/03_backend/init.md
- docs/05_shared/decision-log.md
