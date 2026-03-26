# 02_model 문서 인덱스

## 목적

이 디렉토리는 모델 도메인의 공통 계약과 작업 초안을 관리한다.

- 확정 원칙: LSTM 기반, 공통 하이퍼파라미터 정책
- 미정 항목: sequence_length, horizon 길이, 출력 필드 상세, 임계치 수치
- 도메인 경계: recommendation 최종 판단은 백엔드, 모델은 예측 결과 제공

세부 계약 확정 전에는 수치/필드를 임의로 고정하지 않고,
docs/05_shared/init.md 기준으로 합의 후 반영한다.

## 문서 목록

- lstm-modeling-spec.md
- hyperparameter-policy.md
- inference-output-spec.md
- evaluation-metrics.md
- threshold-rule.md
- retraining-trigger-policy.md
- performance-monitoring-spec.md
- mlflow-tracking-policy.md
- model-selection-policy.md

## 현재 상태

- 상태: 작업 중
- 기준 문서: docs/05_shared/init.md
- 실행 환경: 데이터/모델/백엔드 Python 작업은 루트 `ener` 가상환경과 루트 requirements.txt 기준을 따른다.
