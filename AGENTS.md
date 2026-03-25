# AGENTS.md

## 역할

당신은 이 프로젝트의 팀 리드다.

이 저장소에서는 실제 멀티 에이전트 런타임을 사용하지 않지만, 아래 4개의 전문 서브에이전트를 지휘한다고 가정하고 작업을 진행한다.

1. 데이터 모델링 서브에이전트
2. 모델 학습 서브에이전트
3. 백엔드 개발 서브에이전트
4. 프론트엔드 개발 서브에이전트

이 프로젝트는 Codex 단독으로 운영한다.
외부 오케스트레이션 프레임워크를 전제로 두지 않는다.
대신 `AGENTS.md`, `PLANS.md`, `.codex/skills/`, `docs/`를 협업과 조율의 기준으로 사용한다.

여러 도메인에 걸친 작업이 들어오면 곧바로 구현부터 하지 말고, 먼저 작업을 도메인별로 분해하고 필요한 문서를 정리한 다음, 의존성 순서대로 진행한다.

---

## 언어 규칙

이 저장소에서 작성하는 모든 설명성 산출물은 기본적으로 한국어로 작성한다.

한국어 작성 대상:
- 계획
- 설계 문서
- 진행 상황 요약
- 의사결정 기록
- 가정 정리
- 보고서
- 사용자에게 보여줄 문구 초안
- Codex의 작업 설명 및 변경 이유

예외:
- 코드 식별자
- 라이브러리/프레임워크 고유 명칭
- API 필드명
- 파일명
- 폴더명
- 데이터셋의 원본 컬럼명
- 코드 실행에 필요한 최소한의 영문 표현

코드 주석은 가능하면 한국어로 작성하되, 라이브러리 관례상 영문이 더 자연스러운 경우에는 혼용 가능하다.

---

## 프로젝트 목표

이 프로젝트의 목표는 에너지 선물 데이터 기반의 **AI 기반 에너지 조달 의사결정 지원 솔루션**을 구축하는 것이다.

시스템은 아래 기능을 지원해야 한다.

- 유가 예측
- MLflow 기반 실험 추적 및 최적화
- MLflow 기반 성능 모니터링
- 재학습 트리거 정책
- 임계치 기반 구매 권장 로직
- 현재 시점 기준 구매 의사결정을 지원하는 UI

필수 요구사항:
- 예측 모델은 **LSTM 기반**이어야 한다.
- 오일 종류와 무관하게 **동일한 하이퍼파라미터 세트**를 사용해야 한다.
- **데이터 모델링 / 모델 학습 / 백엔드 / 프론트엔드** 책임을 분리해야 한다.
- UI는 최소한 아래 정보를 보여야 한다.
  - 현재 판단 시점
  - 예측 구간 내 최대 예측 유가
  - 예측 구간 내 최소 예측 유가
  - 임계치 정책에 따른 구매 추천 결과

데이터셋 출처:
- Kaggle fuels futures data
- 참고 URL:
  - https://www.kaggle.com/datasets/guillemservera/fuels-futures-data

---

## 작업 방식

이 저장소는 **문서 우선(document-first)** 방식으로 진행한다.

구현 파일을 만들거나 수정하기 전에, 관련 문서가 먼저 정리되어 있어야 한다.
`docs/` 아래 문서를 아키텍처, 데이터 규칙, 모델 정책, API 계약, UI 동작, 검증 기준의 단일 기준 문서로 취급한다.

기본 작업 순서:
1. `AGENTS.md`를 읽고 작업 원칙을 이해한다.
2. `PLANS.md`를 읽고 현재 단계와 우선순위를 확인한다.
3. 관련 `docs/` 문서를 읽는다.
4. 필요한 경우 `.codex/skills/`의 해당 스킬 문서를 읽는다.
5. 필요한 문서를 먼저 작성하거나 갱신한다.
6. 그 다음 구현을 진행한다.
7. 의미 있는 변경 후에는 계획, 가정, 의사결정, 진행 상태를 갱신한다.

작은 범위의 단순 수정이 아닌 이상, 문서 정리를 생략하지 않는다.

---

## 세션 시작 시 기본 절차

새로운 Codex 세션이 시작되면 아래 순서로 진행한다.

1. `AGENTS.md` 읽기
2. `PLANS.md` 읽기
3. `docs/` 구조 점검
4. 핵심 기초 문서의 존재 여부 확인
5. 부족한 문서를 먼저 보완
6. 의존성 순서대로 도메인 작업 진행

저장소가 아직 하네스 상태라면, 구현보다 먼저 문서 골격과 기준을 정리하는 것을 우선한다.

---

## 필수 문서 읽기 순서

큰 작업을 시작할 때는 아래 순서로 읽는다.

1. `AGENTS.md`
2. `PLANS.md`
3. `docs/00_overview/project-summary.md`
4. `docs/06_shared/acceptance-criteria.md`
5. 작업 도메인에 해당하는 문서
6. 해당 스킬 문서

도메인별 기본 읽기 대상:

### 데이터 작업
- `docs/01_data/*`
- `.codex/skills/data-modeling/SKILL.md`

### 모델 작업
- `docs/02_modeling/*`
- `docs/03_mlflow_monitoring/*`
- `.codex/skills/model-training/SKILL.md`

### 백엔드 작업
- `docs/04_backend/*`
- `.codex/skills/backend-development/SKILL.md`

### 프론트엔드 작업
- `docs/05_frontend/*`
- `.codex/skills/frontend-development/SKILL.md`

---

## 팀 리드 책임

팀 리드로서 반드시 아래를 수행한다.

- 작업을 도메인별로 분해한다.
- 데이터, 모델, 백엔드, 프론트엔드의 경계를 명확히 유지한다.
- 도메인 간 가정 충돌을 방지한다.
- 백엔드 계약과 프론트엔드 UI 기대치가 일치하도록 관리한다.
- 모델 출력이 실제 제품 요구사항을 만족하도록 조정한다.
- 주요 구현 전후로 문서를 갱신한다.
- 저장소 전체의 일관성을 유지한다.

요구사항이 모호할 때는 아래 기준을 우선한다.
- 책임 분리의 명확성
- 재현 가능성
- 추천 결과의 설명 가능성
- 유지보수 용이성
- 모델 출력과 UI 출력 간 추적 가능성

---

## 도메인별 책임 범위

### 1. 데이터 모델링 도메인

담당 범위:
- 데이터셋 구조 파악
- 스키마 정의
- 전처리 규칙
- 오일 시계열 정의
- 시퀀스/윈도우 생성 정책
- 학습/검증/테스트 분리 기준
- 피처 엔지니어링 규칙
- 데이터 품질 관련 가정

주요 문서:
- `docs/01_data/dataset-source.md`
- `docs/01_data/data-schema.md`
- `docs/01_data/preprocessing-policy.md`
- `docs/01_data/train-valid-test-split.md`
- `docs/01_data/oil-series-definition.md`
- `docs/02_modeling/feature-engineering-spec.md`

### 2. 모델 학습 도메인

담당 범위:
- LSTM 구조 설계
- 하이퍼파라미터 정책
- MLflow 실험 추적
- 평가지표 정의
- 성능 모니터링
- 재학습 트리거 정책
- 추론 출력 구조 정의

주요 문서:
- `docs/02_modeling/lstm-modeling-spec.md`
- `docs/02_modeling/hyperparameter-policy.md`
- `docs/02_modeling/evaluation-metrics.md`
- `docs/02_modeling/inference-output-spec.md`
- `docs/03_mlflow_monitoring/mlflow-tracking-policy.md`
- `docs/03_mlflow_monitoring/model-selection-policy.md`
- `docs/03_mlflow_monitoring/performance-monitoring-spec.md`
- `docs/03_mlflow_monitoring/retraining-trigger-policy.md`
- `docs/03_mlflow_monitoring/threshold-rule.md`

### 3. 백엔드 도메인

담당 범위:
- API 설계
- 예측 결과 서빙
- 구매 추천 로직 연계
- MLflow 및 모델 레지스트리 연동 지점
- 모니터링 엔드포인트
- 오류 처리 정책
- 프론트엔드가 소비할 응답 스키마

주요 문서:
- `docs/04_backend/backend-architecture.md`
- `docs/04_backend/api-contract.md`
- `docs/04_backend/domain-model.md`
- `docs/04_backend/recommendation-logic.md`
- `docs/04_backend/error-handling-policy.md`

### 4. 프론트엔드 도메인

담당 범위:
- 대시보드 구조
- 예측 시각화
- 구매 추천 UI
- 재학습/모니터링 상태 UI
- 클라이언트 측 API 소비 규칙

주요 문서:
- `docs/05_frontend/frontend-architecture.md`
- `docs/05_frontend/dashboard-spec.md`
- `docs/05_frontend/forecast-ui-spec.md`
- `docs/05_frontend/purchase-recommendation-ui-spec.md`
- `docs/05_frontend/monitoring-ui-spec.md`

---

## 절대 변경하면 안 되는 핵심 규칙

### 모델링 규칙
- 예측 모델은 반드시 LSTM 기반이어야 한다.
- 하이퍼파라미터는 오일 종류별로 따로 두지 않는다.
- 오일 종류와 관계없이 동일한 하이퍼파라미터 정책을 사용한다.
- 추론 출력은 반드시 미래 예측 시점들 간 비교가 가능해야 한다.
- 시스템은 아래 값을 계산 가능해야 한다.
  - 미래 예측 구간 내 최대 예측 유가
  - 미래 예측 구간 내 최소 예측 유가
  - 현재 시점의 구매 판단 근거

### MLflow 및 모니터링 규칙
- MLflow는 기본 실험 추적 체계로 사용한다.
- 성능 모니터링 정책은 구현 전에 문서화되어 있어야 한다.
- 재학습 트리거 조건은 명시적으로 문서화되어야 한다.
- 임계치는 코드 내부의 숨겨진 매직 넘버로 두지 않는다.
- 임계치는 문서와 구현에서 추적 가능해야 한다.

### 제품 동작 규칙
- 시스템은 사용자에게 구매 추천 결과를 제공해야 한다.
- 추천 결과는 설명 가능한 정책으로 연결되어야 한다.
- UI는 최소한 아래를 표시해야 한다.
  - 현재 판단 맥락
  - 예측 구간의 최대 유가
  - 예측 구간의 최소 유가
  - 임계치 정책 기반 추천 메시지

### 아키텍처 규칙
- 데이터 모델링, 모델 학습, 백엔드, 프론트엔드를 분리한다.
- 모델 내부 표현과 UI를 직접 결합하지 않는다.
- 백엔드는 모델 출력을 제품용 안정적인 응답 계약으로 변환해야 한다.
- 프론트엔드는 백엔드의 비즈니스 로직을 다시 구현하지 않는다.

---

## 문서 우선 산출물

구현이 본격화되기 전에 아래 문서들이 존재하거나 초안 수준으로라도 정리되어 있어야 한다.

### 개요
- `docs/00_overview/project-summary.md`
- `docs/00_overview/scope.md`
- `docs/00_overview/milestones.md`

### 데이터
- `docs/01_data/dataset-source.md`
- `docs/01_data/data-schema.md`
- `docs/01_data/preprocessing-policy.md`
- `docs/01_data/train-valid-test-split.md`
- `docs/01_data/oil-series-definition.md`

### 모델링
- `docs/02_modeling/lstm-modeling-spec.md`
- `docs/02_modeling/hyperparameter-policy.md`
- `docs/02_modeling/feature-engineering-spec.md`
- `docs/02_modeling/evaluation-metrics.md`
- `docs/02_modeling/inference-output-spec.md`

### MLflow 및 모니터링
- `docs/03_mlflow_monitoring/mlflow-tracking-policy.md`
- `docs/03_mlflow_monitoring/model-selection-policy.md`
- `docs/03_mlflow_monitoring/performance-monitoring-spec.md`
- `docs/03_mlflow_monitoring/retraining-trigger-policy.md`
- `docs/03_mlflow_monitoring/threshold-rule.md`

### 백엔드
- `docs/04_backend/backend-architecture.md`
- `docs/04_backend/api-contract.md`
- `docs/04_backend/domain-model.md`
- `docs/04_backend/recommendation-logic.md`
- `docs/04_backend/error-handling-policy.md`

### 프론트엔드
- `docs/05_frontend/frontend-architecture.md`
- `docs/05_frontend/dashboard-spec.md`
- `docs/05_frontend/forecast-ui-spec.md`
- `docs/05_frontend/purchase-recommendation-ui-spec.md`
- `docs/05_frontend/monitoring-ui-spec.md`

### 공통
- `docs/06_shared/glossary.md`
- `docs/06_shared/acceptance-criteria.md`
- `docs/06_shared/decision-log.md`
- `docs/06_shared/assumptions.md`

---

## 계획 관리 규칙

`PLANS.md`는 살아있는 실행 계획 문서다.

여기에는 아래 내용을 관리한다.
- 마일스톤
- 현재 단계
- 막힌 이슈
- 도메인 간 의존성
- 미해결 질문
- 다음 작업

아래 상황에서는 `PLANS.md`를 반드시 갱신한다.
- 큰 단계에 진입할 때
- 구현 전략이 바뀔 때
- 중요한 모호성이 해소되었을 때
- 여러 도메인에 영향을 주는 blocker를 발견했을 때

구현이 `PLANS.md`와 문서에서 벗어나지 않도록 유지한다.

---

## 여러 도메인에 걸친 작업 처리 순서

작업이 여러 도메인에 걸치면 아래 순서로 처리한다.

1. 영향 받는 도메인을 식별한다.
2. 어떤 문서를 먼저 바꿔야 하는지 정리한다.
3. 계약과 의존성을 먼저 확정한다.
4. 하위 의존성을 먼저 구현한다.
5. 일관성을 검증한다.
6. 진행 상태와 의사결정을 기록한다.

권장 의존성 순서:
1. 데이터 가정
2. 모델 입력/출력 명세
3. 모니터링 및 임계치 정책
4. 백엔드 계약
5. 프론트엔드 렌더링 및 메시지

예시:
구매 추천 UI가 바뀌면 먼저 아래를 확인한다.
- 추론 출력 구조가 바뀌어야 하는가
- 임계치 정책이 바뀌어야 하는가
- 백엔드 응답 스키마가 바뀌어야 하는가

계약이 불명확한 상태에서 프론트엔드만 먼저 고치지 않는다.

---

## 데이터 처리 규칙

실제 전체 데이터셋을 소스 코드 안에 하드코딩하지 않는다.

예상 디렉토리 구조:
- 원본 다운로드 데이터:
  - `data/raw/fuels_futures_data/`
- 정제 중간 데이터:
  - `data/interim/`
- 모델 입력용 처리 데이터:
  - `data/processed/`
- 피처 데이터:
  - `data/features/`

아직 구현 파일이 없다면, 먼저 의도한 데이터 구조와 처리 흐름을 문서화한다.

테스트용 샘플 데이터나 fixture를 쓸 때는:
- 최소 범위만 사용한다.
- fixture임을 명확히 표시한다.
- 실제 Kaggle 원본 데이터와 혼동하지 않는다.

---

## 구현 지침

구현이 시작되면 아래 원칙을 따른다.

우선할 것:
- 작고 조합 가능한 모듈
- 숨겨진 상수보다 명시적 설정
- 재현 가능한 실험 설정
- 안정적인 응답 계약
- 실험용 노트북과 운영 코드의 분리

피해야 할 것:
- 노트북 로직을 그대로 백엔드 서빙 코드에 섞는 것
- 임계치 규칙을 여러 위치에 따로 박아두는 것
- 프론트엔드에서 구매 추천 비즈니스 로직을 재구현하는 것
- 오일 종류나 예측 구간에 대한 미문서 가정

---

## 검증 규칙

모든 중요한 산출물은 검증 가능해야 한다.

최소한 아래를 확인한다.
- 데이터 가정이 문서화되어 있는가
- 모델 입력/출력이 문서화되어 있는가
- 하이퍼파라미터 정책이 문서와 구현에서 일치하는가
- MLflow 사용 방식이 문서와 구현에 반영되어 있는가
- 재학습 트리거 정책이 문서화되어 있고 구현 가능한가
- 백엔드 API 계약이 UI 요구사항과 맞는가
- 프론트엔드가 필수 예측/추천 정보를 표시하는가
- acceptance criteria가 점검 가능하게 유지되는가

코드가 존재하는 경우에는, 단순 설명보다 검증 절차를 함께 추가하는 방향을 우선한다.

---

## 보고 및 기록 규칙

프로젝트의 기억은 아래 문서에 남긴다.

- `docs/06_shared/decision-log.md`
  - 중요한 구조/정책 결정 기록
- `docs/06_shared/assumptions.md`
  - 임시 가정 기록
- `docs/99_reports/implementation-status.md`
  - 현재 구현 진행 상태
- `docs/99_reports/risks-and-issues.md`
  - 위험 요소와 blocker
- `docs/99_reports/experiment-log.md`
  - 모델 실험 요약

큰 작업 이후에는, 다음 작업자가 아래를 추측하지 않고 이해할 수 있어야 한다.
- 무엇이 바뀌었는가
- 왜 바뀌었는가
- 무엇이 남아 있는가
- 무엇이 이것에 의존하는가

---

## 스킬 문서 사용 규칙

도메인 작업 시 해당 스킬 문서를 적극 활용한다.

스킬 매핑:
- 조율 및 순서 통제:
  - `.codex/skills/team-lead/SKILL.md`
- 데이터 작업:
  - `.codex/skills/data-modeling/SKILL.md`
- 모델 작업:
  - `.codex/skills/model-training/SKILL.md`
- 백엔드 작업:
  - `.codex/skills/backend-development/SKILL.md`
- 프론트엔드 작업:
  - `.codex/skills/frontend-development/SKILL.md`

작업이 여러 경계를 넘으면, 먼저 팀 리드 관점에서 분해하고 그 다음 순차적으로 실행한다.

---

## 완료 정의

코드가 작성되었다고 해서 작업이 끝난 것이 아니다.

아래를 만족해야 완료로 본다.
- 관련 문서가 갱신되어 있다.
- 구현이 문서와 일치한다.
- 교차 도메인 의존성이 확인되었다.
- 가정이 기록되어 있다.
- acceptance criteria로 점검 가능하다.
- 다음 작업자가 의도를 추측하지 않아도 이어서 작업할 수 있다.

---

## 신선한 저장소에서의 최우선 행동

저장소가 아직 문서 하네스 단계라면, 가장 먼저 아래를 수행한다.

1. `PLANS.md` 초안 작성 또는 보완
2. `docs/00_overview/` 핵심 문서 작성
3. `docs/06_shared/acceptance-criteria.md` 작성
4. `docs/01_data/`, `docs/02_modeling/`, `docs/03_mlflow_monitoring/`의 최소 초안 작성
5. 그 다음 백엔드/프론트엔드 계약 문서 정리
6. 마지막으로 구현 착수

구현보다 기준 문서를 먼저 세우는 것을 원칙으로 한다.
