# Assumptions

## 문서 목적

이 문서는 프로젝트 진행 중 아직 확정되지 않았거나, 임시로 채택한 가정을 기록한다.

이 문서의 목적은 다음과 같다.

- 문서와 구현 사이에 숨어 있는 전제를 드러낸다.
- 병렬 작업 시 서로 다른 가정을 쓰는 문제를 줄인다.
- 나중에 검증, 수정, 폐기해야 할 항목을 명시한다.
- 의사결정이 완료되기 전까지 임시 기준을 공유한다.

가정은 확정 사항이 아니다.
가정이 검증되거나 정책으로 승격되면 관련 문서와 decision log에 반영한다.

---

## 가정 관리 규칙

각 가정은 아래 상태 중 하나를 가진다.

- 초안: 임시로 두는 가정
- 검토 중: 검증 또는 논의가 진행 중인 가정
- 확정: 관련 문서와 정책에 반영된 가정
- 폐기: 더 이상 사용하지 않는 가정

각 가정은 가능하면 아래 항목을 함께 가진다.

- ID
- 내용
- 이유
- 영향 범위
- 검증 필요 여부
- 현재 상태

---

## 현재 가정 목록

### A-001. 데이터셋에는 시계열 정렬 가능한 날짜 기준 컬럼이 존재한다.

- 상태: 초안
- 이유: 시계열 모델링과 train/validation/test 분리에는 시간 순서 기준이 필요하다.
- 영향 범위: 데이터 모델링, 모델 학습
- 검증 필요: 예
- 검증 방법: 데이터셋 컬럼 구조 확인
- 후속 문서 반영 대상:
  - `docs/01_data/data-schema.md`
  - `docs/01_data/train-valid-test-split.md`

---

### A-002. 여러 오일 종류를 하나의 공통 파이프라인으로 처리할 수 있다.

- 상태: 초안
- 이유: 요구사항에서 오일 종류와 무관한 동일 하이퍼파라미터 정책을 요구한다.
- 영향 범위: 데이터 모델링, 모델 학습
- 검증 필요: 예
- 검증 방법: 오일 종류별 데이터 구조와 분포 확인
- 후속 문서 반영 대상:
  - `docs/01_data/oil-series-definition.md`
  - `docs/02_modeling/hyperparameter-policy.md`

---

### A-003. 오일 종류별로 하이퍼파라미터를 따로 두지 않는다.

- 상태: 확정
- 이유: 요구사항에 명시된 핵심 규칙이다.
- 영향 범위: 모델 학습, MLflow 정책
- 검증 필요: 아니오
- 후속 문서 반영 대상:
  - `docs/02_modeling/hyperparameter-policy.md`
  - `docs/03_mlflow_monitoring/mlflow-tracking-policy.md`

---

### A-004. 예측 결과는 미래 시점별 값의 리스트 또는 이에 준하는 구조로 제공한다.

- 상태: 초안
- 이유: 최대/최소 미래 예측 유가를 계산하려면 horizon 내 개별 시점 값이 필요하다.
- 영향 범위: 모델 학습, 백엔드, 프론트엔드
- 검증 필요: 예
- 검증 방법: 추론 출력 설계 시 명확화
- 후속 문서 반영 대상:
  - `docs/02_modeling/inference-output-spec.md`
  - `docs/04_backend/api-contract.md`

---

### A-005. 구매 추천의 핵심 판단 로직은 백엔드에 위치한다.

- 상태: 확정
- 이유: 프론트엔드에서 비즈니스 로직을 재구현하면 정책 일관성이 깨질 수 있다.
- 영향 범위: 백엔드, 프론트엔드
- 검증 필요: 아니오
- 후속 문서 반영 대상:
  - `docs/04_backend/recommendation-logic.md`
  - `docs/05_frontend/purchase-recommendation-ui-spec.md`

---

### A-006. 프론트엔드는 백엔드가 계산한 추천 결과와 설명 문구를 표시하는 역할에 집중한다.

- 상태: 확정
- 이유: 추천 로직의 단일 책임 원칙을 유지하기 위함이다.
- 영향 범위: 프론트엔드
- 검증 필요: 아니오
- 후속 문서 반영 대상:
  - `docs/05_frontend/purchase-recommendation-ui-spec.md`

---

### A-007. MLflow는 실험 추적의 기본 도구로 사용한다.

- 상태: 확정
- 이유: 요구사항에 명시되어 있다.
- 영향 범위: 모델 학습, 모니터링, 백엔드 연계
- 검증 필요: 아니오
- 후속 문서 반영 대상:
  - `docs/03_mlflow_monitoring/mlflow-tracking-policy.md`
  - `docs/03_mlflow_monitoring/model-selection-policy.md`

---

### A-008. 재학습 트리거는 최소한 성능 기준 이탈을 포함한다.

- 상태: 초안
- 이유: 요구사항에 성능 모니터링 및 재학습 triggering이 포함되어 있다.
- 영향 범위: 모델 학습, 모니터링, 백엔드
- 검증 필요: 예
- 검증 방법: 성능 지표와 임계 기준 정의
- 후속 문서 반영 대상:
  - `docs/03_mlflow_monitoring/performance-monitoring-spec.md`
  - `docs/03_mlflow_monitoring/retraining-trigger-policy.md`

---

### A-009. threshold는 문서화된 설정값으로 관리한다.

- 상태: 확정
- 이유: 매직 넘버를 피하고 추천 근거를 추적 가능하게 만들기 위함이다.
- 영향 범위: 모델 학습, 백엔드, 프론트엔드 설명
- 검증 필요: 아니오
- 후속 문서 반영 대상:
  - `docs/03_mlflow_monitoring/threshold-rule.md`
  - `docs/04_backend/recommendation-logic.md`

---

### A-010. 추천 결과는 최소한 "구매 권장" 또는 "보류"를 구분할 수 있어야 한다.

- 상태: 초안
- 이유: 현재 요구사항에는 threshold를 넘으면 구매 권장이라고 되어 있으므로 최소 2단계 정책은 필요하다.
- 영향 범위: 백엔드, 프론트엔드
- 검증 필요: 예
- 검증 방법: 추천 정책 문서화 시 확정
- 후속 문서 반영 대상:
  - `docs/03_mlflow_monitoring/threshold-rule.md`
  - `docs/04_backend/api-contract.md`

---

### A-011. 최대 예측 유가와 최소 예측 유가는 현재 판단 시점에서 동일한 forecast horizon 안에서 계산한다.

- 상태: 초안
- 이유: UI와 추천 로직이 같은 기준의 예측 범위를 바라봐야 한다.
- 영향 범위: 모델 학습, 백엔드, 프론트엔드
- 검증 필요: 예
- 검증 방법: horizon 정의 및 API 계약 확정
- 후속 문서 반영 대상:
  - `docs/02_modeling/inference-output-spec.md`
  - `docs/04_backend/api-contract.md`
  - `docs/05_frontend/forecast-ui-spec.md`

---

### A-012. 초기 단계에서는 문서 우선으로 진행하고, 구현은 문서 합의 후 시작한다.

- 상태: 확정
- 이유: 현재 저장소가 하네스 및 기준 문서 수립 단계이기 때문이다.
- 영향 범위: 전체 프로젝트
- 검증 필요: 아니오

---

## 검증이 필요한 핵심 가정

현재 우선 검증 대상은 아래와 같다.

1. 데이터셋의 실제 컬럼 구조와 날짜 기준
2. 오일 종류 구분 기준
3. 예측 horizon 정의
4. threshold 정책의 형태
5. 재학습 trigger 조건 범위
6. 추천 결과 단계 수

이 항목들은 우선적으로 아래 문서에 반영해야 한다.

- `docs/01_data/data-schema.md`
- `docs/01_data/oil-series-definition.md`
- `docs/02_modeling/inference-output-spec.md`
- `docs/03_mlflow_monitoring/retraining-trigger-policy.md`
- `docs/03_mlflow_monitoring/threshold-rule.md`

---

## 가정 변경 규칙

아래 상황이 생기면 이 문서를 갱신한다.

- 데이터셋 분석 결과 기존 가정이 틀린 것으로 확인된 경우
- 요구사항 해석이 명확해진 경우
- decision log에 기록할 수준의 정책 결정이 완료된 경우
- API 계약 또는 UI 명세에 영향을 주는 변경이 생긴 경우

가정이 확정되면:
- 해당 상태를 `확정`으로 변경한다.
- 관련 정책 문서에 반영한다.
- `decision-log.md`에 필요한 경우 기록한다.

가정이 폐기되면:
- 폐기 사유를 남긴다.
- 영향을 받은 문서와 구현을 함께 점검한다.

---

## 문서 상태

- 상태: 초안
- 작성일: 2026-03-25
- 마지막 갱신일: 2026-03-25
