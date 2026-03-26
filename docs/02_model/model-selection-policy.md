# model-selection-policy.md

## 목적

이 문서는 재학습 후 신규 모델과 기존 모델 중 어떤 모델을 배포할지 결정하는 기준을 정의한다.

현재 기준값과 비교 규칙은 이번 라운드 초안이며,
docs/05_shared/init.md의 미정 항목 합의 후 확정한다.

---

## 모델 선택 기준

### 주요 기준: 단기 RMSE

- 신규 모델의 단기(1~15일) RMSE < 기존 Production 모델의 단기 RMSE → 신규 모델 배포
- 신규 모델의 단기 RMSE ≥ 기존 모델 → 기존 모델 유지

### 보조 기준: 중기 RMSE

- 단기 RMSE가 동일 수준이면 중기(16~23일) RMSE를 비교하여 결정
- 중기 RMSE도 동일 수준이면 기존 모델 유지 (안정성 우선)

---

## 배포 결정 흐름

```
재학습 완료
    │
    ▼
신규 모델 test set 평가
    │
    ▼
기존 Production 모델 성능과 비교
    │
   ┌┴────────────────────┐
신규 모델 성능 우수     기존 모델 성능 동일/우수
   │                       │
Model Registry Staging     기존 모델 유지
   │                       │
LLM 보고서 생성           모니터링 계속
   │
사용자 승인
   │
Production 승격 및 배포
```

---

## MLflow Model Registry 상태 관리

| 상태 | 설명 |
|------|------|
| None | 등록만 된 상태, 서빙 미사용 |
| Staging | 성능 검증 통과, 사용자 승인 대기 |
| Production | 실제 서빙에 사용 중인 모델 |
| Archived | 이전 버전, 롤백 용도로 보관 |

- Production 모델은 유종별로 항상 1개만 유지한다.
- 신규 모델이 Production으로 승격되면 기존 모델은 Archived로 이동한다.

---

## 롤백 정책

- 신규 모델 배포 후 성능이 오히려 저하되면 Archived 모델로 롤백 가능하다.
- 롤백 트리거 조건: 배포 후 첫 평가에서 RMSE가 배포 전보다 20% 이상 증가

---

## 관련 문서

- mlflow-tracking-policy.md
- retraining-trigger-policy.md
- threshold-rule.md
- evaluation-metrics.md
