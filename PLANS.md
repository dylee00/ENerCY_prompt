# PLANS.md

## 문서 목적

이 문서는 AI 기반 에너지 조달 의사결정 지원 솔루션 프로젝트의 살아있는 실행 계획서다.

이 문서는 아래를 관리한다.

- 현재 프로젝트 단계
- 단계별 목표와 산출물
- 도메인 간 의존성
- 우선순위
- 미해결 질문
- 리스크와 대응 방향
- 다음 작업

이 문서는 구현보다 먼저 정리되어야 하며, 주요 방향이 바뀔 때마다 갱신되어야 한다.

---

## 프로젝트 개요

### 프로젝트명
AI 기반 에너지 조달 의사결정 지원 솔루션

### 목표
에너지 선물 시계열 데이터를 기반으로 유가를 예측하고, 예측 결과와 임계치 정책을 바탕으로 현재 시점의 구매 의사결정을 지원하는 시스템을 구축한다.

### 핵심 요구사항
- 유가 예측 모델링
- LSTM 기반 예측 모델 사용
- MLflow 기반 실험 추적 및 최적화
- MLflow 기반 성능 모니터링 및 재학습 트리거
- 오일 종류와 무관한 동일 하이퍼파라미터 정책
- 현재 시점 기준 미래 예측 구간의 최대/최소 유가 제공
- threshold 기반 구매 추천
- 데이터 / 모델 / 백엔드 / 프론트엔드 분리

### 데이터셋
- Kaggle fuels futures data
- URL:
  - https://www.kaggle.com/datasets/guillemservera/fuels-futures-data

---

## 프로젝트 운영 원칙

### 1. 문서 우선
구현 전에 관련 문서를 먼저 작성하거나 갱신한다.

### 2. 도메인 분리
아래 4개 도메인을 분리해서 다룬다.
- 데이터 모델링
- 모델 학습
- 백엔드
- 프론트엔드

### 3. 계약 우선
여러 도메인에 걸친 작업은 아래 순서로 정리한다.
1. 데이터 가정
2. 모델 입력/출력 정의
3. 모니터링 및 threshold 정책
4. 백엔드 API 계약
5. 프론트엔드 표시 방식

### 4. 추적 가능성
추천 결과는 반드시 문서화된 기준과 연결 가능해야 한다.

### 5. 재현 가능성
실험 설정, 하이퍼파라미터, 평가지표, 선택 기준은 재현 가능해야 한다.

---

## 현재 프로젝트 상태

### 현재 단계
초기 하네스 및 문서 골격 수립 단계

### 현재 목표
- 루트 운영 문서 정리
- 도메인별 기본 설계 문서 작성
- 핵심 정책 확정
- 구현 착수 전 기준선 수립

### 현재 완료 상태
- [x] 루트 `AGENTS.md` 초안 작성
- [x] 루트 `PLANS.md` 초안 작성
- [ ] 문서 디렉토리 구조 확정
- [ ] 도메인별 핵심 문서 초안 작성
- [ ] acceptance criteria 정의
- [ ] 데이터 스키마 및 전처리 정책 정의
- [ ] LSTM 입력/출력 규격 정의
- [ ] threshold 정책 정의
- [ ] 백엔드 API 계약 정의
- [ ] 프론트엔드 UI 요구사항 정의
- [ ] 구현 착수

---

## 단계별 실행 계획

## Phase 0. 프로젝트 하네스 정리

### 목표
Codex가 안정적으로 작업할 수 있는 문서 기반 구조를 만든다.

### 주요 작업
- 루트 `AGENTS.md` 작성
- 루트 `PLANS.md` 작성
- `docs/` 구조 확정
- `.codex/skills/` 구조 확정

### 산출물
- `AGENTS.md`
- `PLANS.md`
- 디렉토리 구조 초안
- 스킬 문서 위치 정의

### 완료 기준
- 루트 문서만 읽어도 프로젝트 목표와 작업 순서를 이해할 수 있어야 한다.
- 다음 작업자가 무엇부터 해야 하는지 바로 판단할 수 있어야 한다.

### 상태
진행 중

---

## Phase 1. 프로젝트 개요 및 공통 기준 문서화

### 목표
프로젝트의 범위, 용어, 공통 기준, 완료 조건을 먼저 확정한다.

### 주요 작업
- 프로젝트 요약 문서 작성
- 범위 문서 작성
- 마일스톤 문서 작성
- 용어집 작성
- acceptance criteria 작성
- 가정 문서 작성
- decision log 시작

### 대상 문서
- `docs/00_overview/project-summary.md`
- `docs/00_overview/scope.md`
- `docs/00_overview/milestones.md`
- `docs/06_shared/glossary.md`
- `docs/06_shared/acceptance-criteria.md`
- `docs/06_shared/assumptions.md`
- `docs/06_shared/decision-log.md`

### 산출물
- 프로젝트 범위 정의
- 공통 용어 정의
- 완료 조건 정의
- 초기 가정 목록

### 완료 기준
- 도메인별 문서를 쓰기 전에 공통 기준이 확정되어 있어야 한다.
- acceptance criteria가 체크리스트 형태로 점검 가능해야 한다.

### 상태
시작 전

---

## Phase 2. 데이터 이해 및 데이터 모델링 문서화

### 목표
Kaggle 데이터셋 구조를 이해하고, 모델 입력으로 이어질 데이터 처리 기준을 정의한다.

### 주요 작업
- 데이터셋 구성 파악
- 컬럼/스키마 정의
- 오일 종류 정의
- 결측치/이상치/정렬 기준 정의
- 시계열 단위 및 인덱스 기준 정의
- train/validation/test 분리 정책 정의
- 윈도우 생성 규칙 정의
- feature engineering 기준 정의

### 대상 문서
- `docs/01_data/dataset-source.md`
- `docs/01_data/data-schema.md`
- `docs/01_data/preprocessing-policy.md`
- `docs/01_data/train-valid-test-split.md`
- `docs/01_data/oil-series-definition.md`
- `docs/02_modeling/feature-engineering-spec.md`

### 산출물
- 데이터 스키마 문서
- 전처리 규칙 문서
- 오일 시계열 정의 문서
- 데이터 분리 정책 문서

### 완료 기준
- 어떤 데이터를 입력으로 쓸지 명확해야 한다.
- 어떤 컬럼을 어떻게 처리할지 문서로 설명 가능해야 한다.
- 동일한 데이터 처리 파이프라인을 재현할 수 있어야 한다.

### 상태
시작 전

### 선행 조건
- Phase 1 완료

---

## Phase 3. 모델링 및 실험 정책 문서화

### 목표
LSTM 모델 구조, 입력/출력 형태, 공통 하이퍼파라미터 정책, 평가지표를 확정한다.

### 주요 작업
- LSTM 구조 정의
- 입력 텐서 구조 정의
- 출력 정의
- 예측 horizon 정의
- 오일 종류 공통 하이퍼파라미터 정책 정의
- 평가 방식 정의
- 모델 선택 기준 정의

### 대상 문서
- `docs/02_modeling/lstm-modeling-spec.md`
- `docs/02_modeling/hyperparameter-policy.md`
- `docs/02_modeling/evaluation-metrics.md`
- `docs/02_modeling/inference-output-spec.md`

### 산출물
- LSTM 모델링 명세
- 하이퍼파라미터 정책 문서
- 평가지표 문서
- 추론 출력 문서

### 완료 기준
- 모델 입력/출력 구조가 문서로 명확해야 한다.
- 오일 종류별로 하이퍼파라미터를 다르게 쓰지 않는다는 정책이 고정되어 있어야 한다.
- 예측 결과로 최대/최소 미래 유가를 계산할 수 있어야 한다.

### 상태
시작 전

### 선행 조건
- Phase 2 완료

---

## Phase 4. MLflow 및 성능 모니터링 정책 문서화

### 목표
MLflow 사용 방식, 성능 추적 방식, 재학습 트리거 기준, threshold 정책을 문서화한다.

### 주요 작업
- MLflow 추적 정책 정의
- 실험 로깅 기준 정의
- 모델 선택 기준 정의
- 모니터링 지표 정의
- 성능 저하 판단 기준 정의
- 재학습 trigger 조건 정의
- 구매 추천 threshold 규칙 정의

### 대상 문서
- `docs/03_mlflow_monitoring/mlflow-tracking-policy.md`
- `docs/03_mlflow_monitoring/model-selection-policy.md`
- `docs/03_mlflow_monitoring/performance-monitoring-spec.md`
- `docs/03_mlflow_monitoring/retraining-trigger-policy.md`
- `docs/03_mlflow_monitoring/threshold-rule.md`

### 산출물
- MLflow 정책 문서
- 모니터링 정책 문서
- 재학습 기준 문서
- threshold 정책 문서

### 완료 기준
- 실험 로그에 무엇을 남길지 명확해야 한다.
- 재학습 조건이 문서와 코드에서 일치 가능해야 한다.
- 구매 추천 규칙이 매직 넘버 없이 설명 가능해야 한다.

### 상태
시작 전

### 선행 조건
- Phase 3 완료

---

## Phase 5. 백엔드 계약 및 추천 로직 설계

### 목표
프론트엔드가 사용할 안정적인 API 계약과 추천 로직 연계 방식을 정의한다.

### 주요 작업
- API 엔드포인트 정의
- 요청/응답 스키마 정의
- 예측 결과 응답 구조 정의
- 최대/최소 미래 유가 응답 방식 정의
- threshold 기반 추천 응답 정의
- 오류 처리 정책 정의

### 대상 문서
- `docs/04_backend/backend-architecture.md`
- `docs/04_backend/api-contract.md`
- `docs/04_backend/domain-model.md`
- `docs/04_backend/recommendation-logic.md`
- `docs/04_backend/error-handling-policy.md`

### 산출물
- 백엔드 아키텍처 문서
- API 계약 문서
- 추천 로직 문서
- 에러 처리 정책 문서

### 완료 기준
- 프론트엔드가 어떤 데이터를 어떤 형식으로 받을지 명확해야 한다.
- 추천 결과와 설명 메시지가 API 수준에서 정의되어 있어야 한다.
- ML 출력과 제품 응답 사이의 변환 책임이 백엔드에 위치해야 한다.

### 상태
시작 전

### 선행 조건
- Phase 4 완료

---

## Phase 6. 프론트엔드 UI 설계

### 목표
사용자가 구매 판단을 쉽게 내릴 수 있도록 화면 구조와 표시 규칙을 정의한다.

### 주요 작업
- 대시보드 구조 정의
- 예측 차트 표시 규칙 정의
- 최대/최소 유가 카드 정의
- 구매 추천 카드 정의
- threshold 상태 표시 정의
- 모니터링 상태 패널 정의

### 대상 문서
- `docs/05_frontend/frontend-architecture.md`
- `docs/05_frontend/dashboard-spec.md`
- `docs/05_frontend/forecast-ui-spec.md`
- `docs/05_frontend/purchase-recommendation-ui-spec.md`
- `docs/05_frontend/monitoring-ui-spec.md`

### 산출물
- 화면 구조 문서
- 컴포넌트 역할 문서
- 사용자 메시지 규칙 문서

### 완료 기준
- 사용자가 현재 판단 시점, 최대/최소 유가, 추천 결과를 한 화면 흐름에서 이해할 수 있어야 한다.
- 프론트엔드는 백엔드 계약에만 의존하도록 설계되어야 한다.

### 상태
시작 전

### 선행 조건
- Phase 5 완료

---

## Phase 7. 구현 착수

### 목표
문서 기준에 따라 데이터, 모델, 백엔드, 프론트엔드를 순차적으로 구현한다.

### 권장 구현 순서
1. 데이터 로딩 및 전처리
2. 시퀀스/윈도우 생성
3. LSTM 학습 파이프라인
4. MLflow 추적 및 모델 선택
5. 모니터링 및 재학습 트리거 구현
6. 백엔드 API 구현
7. 프론트엔드 UI 구현
8. 통합 검증

### 산출물
- 데이터 파이프라인 코드
- 모델 학습 코드
- MLflow 연동 코드
- 백엔드 API 코드
- 프론트엔드 UI 코드

### 완료 기준
- 구현이 문서와 일치해야 한다.
- 최소 기능 흐름이 end-to-end로 연결되어야 한다.
- acceptance criteria로 점검 가능해야 한다.

### 상태
시작 전

### 선행 조건
- Phase 6 완료

---

## 도메인별 현재 할 일

## 데이터 모델링 도메인
### 해야 할 일
- [ ] 데이터셋 파일 구성 파악
- [ ] 주요 컬럼 정리
- [ ] 날짜/시계열 인덱스 기준 정의
- [ ] 오일 종류 분류 기준 정리
- [ ] 결측치/이상치 처리 원칙 정리
- [ ] window 생성 기준 초안 작성

### 산출 문서
- `docs/01_data/dataset-source.md`
- `docs/01_data/data-schema.md`
- `docs/01_data/preprocessing-policy.md`
- `docs/01_data/oil-series-definition.md`

---

## 모델 학습 도메인
### 해야 할 일
- [ ] LSTM 기본 구조 정의
- [ ] 입력 시퀀스 길이 후보 정의
- [ ] 예측 horizon 정의
- [ ] 공통 하이퍼파라미터 정책 정의
- [ ] MLflow 기록 항목 정의
- [ ] 평가지표 정의

### 산출 문서
- `docs/02_modeling/lstm-modeling-spec.md`
- `docs/02_modeling/hyperparameter-policy.md`
- `docs/02_modeling/evaluation-metrics.md`
- `docs/02_modeling/inference-output-spec.md`

---

## 백엔드 도메인
### 해야 할 일
- [ ] forecast API 초안 작성
- [ ] recommendation API 초안 작성
- [ ] monitoring API 초안 작성
- [ ] 응답 스키마 정의
- [ ] 오류 응답 구조 정의

### 산출 문서
- `docs/04_backend/api-contract.md`
- `docs/04_backend/recommendation-logic.md`
- `docs/04_backend/error-handling-policy.md`

---

## 프론트엔드 도메인
### 해야 할 일
- [ ] 대시보드 정보 구조 정의
- [ ] 예측 차트 요구사항 정의
- [ ] 구매 추천 카드 UI 요구사항 정의
- [ ] 최대/최소 가격 카드 정의
- [ ] 모니터링 UI 정의

### 산출 문서
- `docs/05_frontend/dashboard-spec.md`
- `docs/05_frontend/forecast-ui-spec.md`
- `docs/05_frontend/purchase-recommendation-ui-spec.md`
- `docs/05_frontend/monitoring-ui-spec.md`

---

## 핵심 의존성 맵

### 데이터 -> 모델
- 데이터 스키마가 확정되어야 모델 입력 구조를 정의할 수 있다.
- 시계열 분리 정책이 확정되어야 평가 전략을 정의할 수 있다.

### 모델 -> 모니터링
- 평가지표와 추론 출력이 확정되어야 성능 모니터링과 threshold 정책을 설계할 수 있다.

### 모델/모니터링 -> 백엔드
- 예측 결과 형식과 추천 기준이 확정되어야 백엔드 응답 계약을 설계할 수 있다.

### 백엔드 -> 프론트엔드
- API 응답 구조가 확정되어야 화면 컴포넌트와 사용자 메시지를 안정적으로 설계할 수 있다.

---

## 현재 미해결 질문

- 데이터셋 내 오일 종류 구분은 어떤 컬럼과 어떤 값 체계를 기준으로 할 것인가?
- 예측 horizon은 며칠 또는 몇 시점으로 정의할 것인가?
- threshold는 절대값 기준인가, 상대 비교 기준인가, 둘의 조합인가?
- 추천 결과는 단순 구매/보류 2단계인가, 다단계인가?
- 재학습 trigger는 성능 저하만 기준으로 할 것인가, 데이터 drift도 고려할 것인가?
- UI에서 사용자에게 보여줄 설명 문구의 수준은 얼마나 상세해야 하는가?

이 질문들은 `docs/06_shared/assumptions.md`와 `docs/06_shared/decision-log.md`에 함께 관리한다.

---

## 초기 가정

아래 가정은 임시이며, 문서화 후 변경 가능하다.

- [가정] 데이터는 시계열 정렬 가능한 날짜 컬럼을 포함한다.
- [가정] 여러 오일 종류를 하나의 공통 파이프라인으로 처리할 수 있다.
- [가정] 오일 종류별로 모델 구조는 바꾸지 않는다.
- [가정] 오일 종류별로 하이퍼파라미터도 바꾸지 않는다.
- [가정] 프론트엔드는 백엔드가 제공한 추천 결과를 표시하는 역할에 집중한다.
- [가정] 추천 판단의 핵심 로직은 백엔드에 위치한다.

---

## 주요 리스크

### 1. 데이터 구조 불확실성
설명:
- Kaggle 데이터셋의 실제 컬럼 구조와 기대 구조가 다를 수 있다.

대응:
- Phase 2에서 데이터 스키마 문서를 먼저 확정한다.
- 필요한 경우 assumptions에 임시 기록 후 수정한다.

### 2. threshold 정책의 모호성
설명:
- 구매 추천 기준이 너무 단순하거나 너무 복잡하면 제품 설명 가능성이 떨어질 수 있다.

대응:
- threshold-rule 문서에서 기준을 명확히 한다.
- 추천 메시지와 정책 간 연결을 유지한다.

### 3. 모델 출력과 UI 요구사항 불일치
설명:
- 모델은 예측값만 내놓고, UI는 최대/최소/추천 맥락까지 필요로 한다.

대응:
- inference-output-spec과 backend api-contract를 먼저 맞춘다.

### 4. 도메인 간 경계 붕괴
설명:
- 프론트엔드나 백엔드에 모델 정책이 중복되거나, 임계치가 여러 군데 퍼질 수 있다.

대응:
- recommendation-logic과 threshold-rule을 단일 기준으로 유지한다.

### 5. MLflow 사용 범위 불명확
설명:
- 실험 추적만 할지, 등록/선택/모니터링까지 연결할지 초기 설계가 모호할 수 있다.

대응:
- MLflow 관련 문서를 Phase 4에서 명확히 분리해 정리한다.

---

## 다음 행동

### 최우선
- [ ] `docs/00_overview/project-summary.md` 작성
- [ ] `docs/00_overview/scope.md` 작성
- [ ] `docs/00_overview/milestones.md` 작성
- [ ] `docs/06_shared/acceptance-criteria.md` 작성
- [ ] `docs/06_shared/glossary.md` 작성

### 그 다음
- [ ] `docs/01_data/` 핵심 문서 초안 작성
- [ ] `docs/02_modeling/` 핵심 문서 초안 작성
- [ ] `docs/03_mlflow_monitoring/threshold-rule.md` 초안 작성

### 이후
- [ ] 백엔드 API 계약 초안 작성
- [ ] 프론트엔드 UI 사양 초안 작성
- [ ] 구현 착수 순서 확정

---

## 갱신 규칙

아래 상황이 발생하면 이 문서를 반드시 갱신한다.

- 새로운 phase로 넘어갈 때
- 주요 요구사항 해석이 바뀔 때
- 도메인 간 의존성이 바뀔 때
- blocker가 생겼을 때
- 구현 시작 또는 종료 시점
- acceptance criteria가 변경될 때

---

## 변경 이력

### 2026-03-25
- 초안 생성
- 문서 우선 프로젝트 계획 구조 정의
- 단계별 실행 계획 및 의존성 정리
