# docs/03_backend/init.md

## 역할

당신은 이 프로젝트의 **백엔드 서브에이전트**다.

당신의 목표는 현재 제공된 base code 구조를 최대한 활용하면서도, 프론트엔드 명세에 맞는 FastAPI 기반 backend를 구현하는 것이다.

이번 backend의 핵심 역할은 아래 두 가지다.

1. 모델을 import하고, 현재 성능이 threshold 기준보다 나쁘면 재학습을 트리거하는 로직 구현
2. 프론트엔드 명세에 맞는 `/api/v1` 기반 JSON API 구현

---

## 현재 작업 범위

현재 이 디렉토리에는 아래 파일이 있다.

- `main.py`
- `config.py`
- `model.py`
- `weight_used_model.py`
- `init.md`

backend 구현은 위 base code를 바탕으로 진행한다.

원칙:
- 기존 코드를 무시하고 새 프로젝트처럼 갈아엎지 않는다.
- 필요한 최소 수준의 리팩터링과 파일 분리는 허용한다.
- route는 얇게 유지하고, 비즈니스 로직은 service 또는 adapter 계층으로 분리한다.

---

## 현재 base code Context

### 1. `main.py`

현재 `main.py`는 실습용 FastAPI 서버 구조를 가지고 있다.

현재 특징:
- `/upload`, `/download`, `/download_shapes` 등 실습용 route가 있다.
- 정적 파일 mount, CORS, health check, root route가 있다.
- 상단에서 `.model`, `.weight_used_model`을 import하고 있다.
- 동시에 `importlib.import_module()`로도 지연 import를 일부 사용하고 있다.
- import 구조가 일관되지 않다.
- 프론트엔드가 요구하는 `/api/v1/dashboard`, `/forecast`, `/recommendation`, `/monitoring` API는 아직 없다.

현재 문제:
- backend 실무용 API 구조와 맞지 않는다.
- import 시점에 모델 관련 부작용이 발생할 가능성이 있다.
- route와 모델 호출 구조가 분리되어 있지 않다.

---

### 2. `config.py`

현재 `config.py`는 경로 관련 설정 중심이다.

현재 특징:
- `BASE_DIR` 기본값이 로컬 절대경로 기반이다.
- 업로드 / 모델 / 이미지 경로가 정의되어 있다.
- threshold, model status, API 관련 설정은 없다.

현재 문제:
- 현재 프로젝트 요구사항에 필요한 threshold 관련 설정이 없다.
- 환경 변수 기반 설정 구조가 부족하다.
- toy project 기준으로는 확장 가능하지만, backend 정책 값 추가가 필요하다.

---

### 3. `model.py`

현재 `model.py`는 import 시 아래를 자동 실행한다.

- CSV 로드
- 전처리
- LSTM 학습
- 모델 저장
- model plot 저장

현재 문제:
- import 시 학습이 자동 수행된다.
- backend가 import만 해도 학습이 돌아가면 안 된다.
- `process(dataset)`는 결과 이미지 경로와 RMSE 문자열을 반환한다.
- RMSE가 문자열이라 threshold 비교에 바로 쓰기 어렵다.
- 일부 컬럼 및 기간 기준이 하드코딩되어 있을 수 있다.

backend 관점 핵심 판단:
- import 부작용을 줄이거나 우회해야 한다.
- 가능하면 학습은 명시적 함수 호출 시점에만 수행되게 해야 한다.

---

### 4. `weight_used_model.py`

현재 `weight_used_model.py`는 import 시 아래를 자동 실행한다.

- 저장 모델 로드
- 데이터 로드
- model plot 저장

현재 문제:
- import만 해도 model load / plot 생성이 발생한다.
- `process(dataset)` 역시 이미지 경로와 RMSE 문자열을 반환한다.
- backend는 이 결과를 프론트 응답용 구조로 변환해야 한다.

backend 관점 핵심 판단:
- 예측 실행, metric 추출, 응답 변환을 adapter 계층에서 감싸야 한다.

---

## backend가 반드시 지켜야 하는 원칙

1. recommendation 판단은 frontend가 아니라 backend가 수행한다.
2. threshold 비교 로직은 frontend로 넘기지 않는다.
3. route 함수는 얇게 유지하고 비즈니스 로직은 service 계층으로 분리한다.
4. import 시점 부작용을 줄이거나 우회해야 한다.
5. threshold는 매직 넘버가 아니라 설정값으로 관리한다.
6. data / model / backend는 동일한 `ener` 가상환경을 사용한다.
7. Python 3.11 기준으로 구현한다.
8. toy project 수준이므로 동기 재학습은 허용하되 구조는 분리한다.
9. 실제 API key나 secret은 문서에 쓰지 않고 환경 변수로만 사용한다.
10. 프론트엔드는 recommendation 결과를 표시만 하고 계산은 하지 않는다.
11. 단, **UI 표현용 강조(카드 하이라이트 등)는 프론트엔드 표현 로직으로 허용**한다.

---

## 필요한 환경 변수

실제 값은 문서에 적지 않고 `.env` 또는 실행 환경에서 주입한다.

필요한 환경 변수 예시:
- `OPENAI_API_KEY`
- `MODEL_PROVIDER_API_KEY`
- `RETRAIN_RMSE_THRESHOLD`
- `WARNING_RMSE_THRESHOLD`
- `DEFAULT_FORECAST_HORIZON`

원칙:
- 실제 API key를 이 문서에 적지 않는다.
- backend는 `os.getenv()` 또는 settings 객체로 환경 변수를 읽는다.
- threshold와 horizon은 설정값으로 읽도록 구현한다.

---

## 프론트엔드 협의 반영 사항

이 섹션은 프론트엔드 pending agreement(PA-001 ~ PA-004)에 대한 백엔드 기준 확정안을 정의한다.

### PA-001. UI 강조 대상 표시용 메타 플래그

현재 단계에서는 `highlightTarget` 필드를 backend 응답에 추가하지 않는다.

즉:
- backend는 recommendation, forecast summary, monitoring 데이터를 제공한다.
- 프론트엔드는 `oilType`과 도메인 정책에 따라 카드 강조 표시를 수행한다.

선택 이유:
- 강조 표시 자체는 recommendation 판단 로직이 아니라 **표현 로직**에 가깝다.
- toy project 범위에서는 backend 응답을 불필요하게 확장하지 않고도 구현 가능하다.
- 다만 추후 오일 종류 정책이 복잡해지면 `highlightTarget` 필드를 도입할 수 있다.

현재 원칙:
- recommendation / threshold / 재학습 판단은 backend 책임
- 카드 강조, 배지 색상, 시각 효과는 frontend 표현 책임

---

### PA-002. 예측 시계열 구간 식별자 필드

`/api/v1/forecast`의 `forecastPoints` 각 항목에 `periodType` 필드를 추가한다.

예시:

```json
{
  "forecastPoints": [
    {
      "time": "2026-03-26T00:00:00Z",
      "predictedPrice": 79.10,
      "periodType": "WAITING"
    },
    {
      "time": "2026-04-10T00:00:00Z",
      "predictedPrice": 80.50,
      "periodType": "TARGET"
    }
  ]
}
```

허용 값:
- `WAITING`
- `TARGET`

선택 이유:
- frontend가 인덱스로 `0~14`, `15~22`를 직접 자르는 로직을 두지 않게 한다.
- 리드타임 정책 변경 시 frontend 수정 범위를 줄일 수 있다.

---

### PA-003. dashboard summary 가격 산출 기준

`/api/v1/dashboard`의 `forecastSummary.predictedMaxPrice` 및 `forecastSummary.predictedMinPrice`는 **실질 타겟 구간(16~23일)** 기준으로 계산한다.

즉:
- dashboard summary의 최대/최소 가격은 16~23일 기준
- recommendation priceContext의 최대/최소 가격도 16~23일 기준

선택 이유:
- 구매 의사결정과 직접 연결되는 구간이 16~23일이기 때문이다.
- 전체 1~23일 기준 summary는 사용자에게 구매 판단 맥락을 흐릴 수 있다.

---

### PA-004. monitoring 성능 지표 응답 구조

`/api/v1/monitoring`의 `latestEvaluation`은 **구간별 중첩 구조**를 사용한다.

예시:

```json
{
  "latestEvaluation": {
    "shortTerm": {
      "rmse": 2.41,
      "mae": 1.82,
      "mape": 2.3
    },
    "midTerm": {
      "rmse": 4.10,
      "mae": 3.20,
      "mape": 3.8
    }
  }
}
```

선택 이유:
- `shortTerm` / `midTerm` 구분이 명확하다.
- metric 확장이 쉽다.
- threshold 적용 대상을 설명하기 쉽다.
- UI에서 구간별 카드 렌더링이 쉽다.

---

### horizon 추가 확인 사항

`horizon`은 아래 기준으로 사용한다.

- 최대 horizon: `23`
- `/api/v1/forecast` 기본 horizon: `23`
- `/api/v1/dashboard` summary 계산 기준: `16~23일`
- `/api/v1/recommendation` summary 계산 기준: `16~23일`

운영 원칙:
- forecast API는 전체 예측 구간 조회용으로 사용한다.
- dashboard / recommendation API는 구매 판단용 요약 응답으로 사용한다.

---

## threshold 및 monitoring 기준

백엔드는 threshold 및 상태 판단을 아래 기준으로 수행한다.

### 1. 1차 판단 metric

재학습 필요 여부와 모델 상태 판단의 1차 기준 metric은 **`midTerm.rmse`** 로 한다.

선택 이유:
- 실제 구매 의사결정과 직접 연결되는 구간이 16~23일이기 때문이다.

### 2. shortTerm metric의 역할

`shortTerm` 지표는 monitoring 참고용으로 함께 제공한다.
단, `retrainingRequired`의 1차 판단 기준으로는 사용하지 않는다.

### 3. 상태 판단 기준

기본 상태는 아래와 같이 둔다.

- `HEALTHY`
- `WARNING`
- `CRITICAL`

권장 해석:
- `HEALTHY`: `midTerm.rmse <= WARNING_RMSE_THRESHOLD`
- `WARNING`: `WARNING_RMSE_THRESHOLD < midTerm.rmse <= RETRAIN_RMSE_THRESHOLD`
- `CRITICAL`: `midTerm.rmse > RETRAIN_RMSE_THRESHOLD`

### 4. 재학습 필요 여부

- `midTerm.rmse > RETRAIN_RMSE_THRESHOLD` 이면 `retrainingRequired = true`
- 그 외에는 `retrainingRequired = false`

### 5. threshold 시각화 지원

frontend의 monitoring UI를 위해, backend는 `latestEvaluation` 외에 구간별 threshold 정보도 함께 제공하는 것을 권장한다.

예시:

```json
{
  "evaluationThresholds": {
    "shortTerm": {
      "warningRmse": 2.5,
      "retrainRmse": 4.0
    },
    "midTerm": {
      "warningRmse": 3.5,
      "retrainRmse": 5.0
    }
  }
}
```

주의:
- 전체 `modelStatus`와 `retrainingRequired`의 최종 판단 기준은 여전히 `midTerm.rmse`다.
- `shortTerm` threshold는 구간별 모니터링 시각화용 보조 정보다.

### 6. 설정값 관리

threshold 값은 route 내부 매직 넘버로 두지 않고 설정값으로 관리한다.

예시 환경 변수:
- `WARNING_RMSE_THRESHOLD`
- `RETRAIN_RMSE_THRESHOLD`
- `DEFAULT_FORECAST_HORIZON`

필요 시 구간별 threshold 확장을 허용한다.
예:
- `WARNING_RMSE_THRESHOLD_SHORT`
- `RETRAIN_RMSE_THRESHOLD_SHORT`
- `WARNING_RMSE_THRESHOLD_MID`
- `RETRAIN_RMSE_THRESHOLD_MID`

---

## 프론트엔드 명세

### Base URL
- `/api/v1`

### 공통 응답 원칙
- 모든 응답은 JSON
- 시간은 ISO 8601 문자열 사용
- 숫자 가격은 number 사용
- 추천 판단 로직은 백엔드가 수행
- 프론트엔드는 계산하지 않고 표시만 수행

### 공통 에러 응답 형식

```json
{
  "code": "BAD_REQUEST",
  "message": "요청 파라미터가 올바르지 않습니다.",
  "details": {}
}
```

---

## 반드시 구현해야 하는 API

### 1. `GET /api/v1/dashboard`

메인 대시보드 핵심 정보를 한 번에 반환한다.

반드시 포함:
- `generatedAt`
- `oilType`
- `currentDecisionTime`
- `currentPrice`
- `forecastSummary`
- `recommendation`
- `monitoring`

추가 원칙:
- `forecastSummary.predictedMaxPrice`
- `forecastSummary.predictedMinPrice`

위 두 값은 **16~23일 타겟 구간 기준**으로 계산한다.

---

### 2. `GET /api/v1/forecast`

차트 표시용 미래 예측 시계열을 반환한다.

반드시 포함:
- `generatedAt`
- `oilType`
- `currentDecisionTime`
- `currentPrice`
- `forecastHorizon`
- `forecastPoints`
- `summary`

추가 원칙:
- 기본 `horizon`은 `23`
- 최대 `horizon`은 `23`
- 요청값이 없으면 전체 구간을 반환한다.
- 각 `forecastPoints` 항목은 `periodType`을 포함해야 한다.
- `periodType`은 `WAITING` 또는 `TARGET` 값을 가진다.

---

### 3. `GET /api/v1/recommendation`

현재 시점 기준 구매 추천 결과를 반환한다.

반드시 포함:
- `generatedAt`
- `oilType`
- `currentDecisionTime`
- `currentPrice`
- `recommendation`
- `priceContext`

추가 원칙:
- `priceContext.predictedMaxPrice`
- `priceContext.predictedMinPrice`
- `expectedSpread`

위 값은 **16~23일 타겟 구간 기준**으로 계산한다.

---

### 4. `GET /api/v1/monitoring`

모델 상태와 재학습 필요 여부를 반환한다.

반드시 포함:
- `generatedAt`
- `modelStatus`
- `retrainingRequired`
- `latestModelVersion`
- `latestEvaluation`
- `message`

권장 포함:
- `evaluationThresholds`

추가 원칙:
- `latestEvaluation`은 **중첩 구조**를 사용한다.
- `retrainingRequired` 판단 기준은 **`midTerm.rmse`** 다.
- backend가 MLflow의 최신 지표를 읽어 상태를 계산한다.

---

## backend가 구현해야 하는 핵심 로직

### 1. model import 및 adapter 계층

기존 `model.py`, `weight_used_model.py`를 route에서 직접 다루지 말고 adapter 계층을 둔다.

adapter가 해야 하는 일:
- 기존 model module import
- 예측 실행
- metric 추출
- RMSE 문자열을 숫자로 변환
- retrain 진입점 제공
- MLflow 최신 metric 조회 보조

최소 adapter 함수 예시:
- `predict(oil_type: str, horizon: int) -> dict`
- `evaluate(oil_type: str) -> dict`
- `retrain(oil_type: str) -> dict`
- `get_latest_metrics(oil_type: str) -> dict`

중요:
- 현재 `process()`가 이미지 경로와 RMSE 문자열을 반환한다면, backend adapter에서 숫자 metric과 응답용 구조를 재조립해야 한다.
- 불가능하면 model 파일에 최소 수준의 수정은 허용한다.

---

### 2. threshold 기반 monitoring

현재 성능 metric을 threshold와 비교하여 상태를 계산한다.

최소 상태:
- `HEALTHY`
- `WARNING`
- `CRITICAL`

최소 판단:
- 성능이 기준 이상이면 `retrainingRequired = false`
- 성능이 기준 미달이면 `retrainingRequired = true`

현재 base code 기준으로 현실적인 1차 metric은 RMSE다.

중요:
- RMSE는 문자열이 아니라 숫자로 비교해야 한다.
- 1차 판단 기준 metric은 `midTerm.rmse`다.
- threshold는 config 또는 환경 변수 기반 설정값으로 관리한다.

---

### 3. 재학습 트리거

threshold 기준 미달 시 재학습을 트리거할 수 있어야 한다.

toy project 기준 허용:
- monitoring 요청 시 동기 재학습
- recommendation 요청 시 상태 평가 후 재학습
- 별도 service 함수에서 retrain 수행

권장:
- retrain 로직은 `retraining_service.py` 같은 서비스 함수로 분리한다.
- route 안에서 직접 학습 코드를 호출하지 않는다.
- backend가 MLflow에서 최신 metric을 읽어 판단한다.

---

### 4. recommendation 계산

recommendation 판단은 backend가 수행한다.

최소 계산 항목:
- 현재 가격
- 미래 예측값 목록
- 최대 예측 유가
- 최소 예측 유가
- expectedSpread
- BUY / HOLD
- thresholdStatus
- reasonSummary

최소 action:
- `BUY`
- `HOLD`

원칙:
- recommendation 결과는 프론트에서 다시 계산하지 않도록 backend에서 완성해서 반환한다.
- recommendation 관련 max/min 및 spread 계산은 **16~23일 타겟 구간 기준**이다.

---

## 권장 구현 구조

현재 구조를 크게 무너뜨리지 않되, 아래 수준의 최소 분리를 권장한다.

예시:
- `main.py`
- `config.py`
- `schemas.py`
- `services/model_adapter.py`
- `services/monitoring_service.py`
- `services/retraining_service.py`
- `services/recommendation_service.py`

토이 프로젝트이므로 router 분리까지는 필수가 아니다.
다만 route에 비즈니스 로직을 몰아넣지 않는다.

---

## backend가 실제로 먼저 해야 할 일

### 1단계
- `main.py`의 상단 import 구조 점검
- `.model`, `.weight_used_model`의 top-level import 제거 또는 우회
- `/api/v1` route 골격 추가
- response schema 정의

### 2단계
- model adapter 추가
- metric 문자열 파싱 추가
- MLflow metric 조회 흐름 추가
- monitoring 상태 계산 추가
- retraining trigger 추가

### 3단계
- recommendation 계산 service 추가
- dashboard 응답 조립
- forecast periodType 부여 로직 추가
- 공통 에러 응답 처리 추가

### 4단계
- 기존 `/upload`, `/download` 계열 route 유지 여부 판단
- 필요 시 deprecated 또는 분리 표시
- 검증 및 정리

---

## 구현 시 명시적 제약

- FastAPI를 사용한다.
- 기존 코드를 무시하고 새 프로젝트처럼 갈아엎지 않는다.
- 기존 `model.py`, `weight_used_model.py`는 최대한 재사용하되, import 부작용을 방치하지 않는다.
- threshold는 설정값으로 관리한다.
- recommendation 로직은 backend에 둔다.
- frontend는 계산하지 않고 표시만 하게 한다.
- `ener` 가상환경 기준으로 동작해야 한다.

---

## backend가 다른 도메인과 맞춰야 하는 것

### 모델 도메인과 맞춰야 하는 것
- 예측값 배열 구조
- forecast horizon
- 성능 metric
- retrain 진입점
- shortTerm / midTerm metric 기록 구조

### 프론트엔드 도메인과 맞춰야 하는 것
- dashboard 응답 필드
- forecast 차트용 필드
- recommendation 카드용 필드
- monitoring 패널용 필드
- `periodType` 사용 방식
- `latestEvaluation` 구조 사용 방식

### 공통 문서와 맞춰야 하는 것
- `docs/05_shared/init.md`의 공통 계약
- `PLANS.md`의 병렬 구현 범위
- `AGENTS.md`의 environment / 분리 책임 규칙

---

## 작업 후 반드시 반영할 문서

backend 작업이 끝나면 아래를 반영한다.

### 1. `PLANS.md`
- backend 진행 상태
- 병합 전/후 상태
- 남은 blocker
- 다음 작업

### 2. `docs/05_shared/init.md`
- recommendation 응답 구조
- monitoring 응답 구조
- threshold 관련 공통 계약
- periodType 관련 공통 계약
- 새로 생긴 미정 사항

### 3. `docs/03_backend/init.md`
- 실제 구현 방향
- 실제 수정한 구조
- 최종 API 응답 구조
- retraining 동작 방식

---

## 완료 기준

아래를 만족하면 backend 서브에이전트 작업을 완료로 본다.

- `/api/v1/dashboard` 구현
- `/api/v1/forecast` 구현
- `/api/v1/recommendation` 구현
- `/api/v1/monitoring` 구현
- 공통 에러 응답 형식 구현
- threshold 비교 가능
- threshold 기준 미달 시 재학습 트리거 가능
- recommendation 판단이 backend에 위치
- 프론트 명세와 JSON 필드가 맞음
- `forecastPoints[].periodType`가 반영됨
- monitoring 응답에 중첩형 `latestEvaluation` 구조가 반영됨
- `midTerm.rmse` 기준 상태 판단이 가능함
- import 부작용에 대한 방어 또는 리팩터링이 반영됨
- `ener` 가상환경 기준으로 실행 가능
