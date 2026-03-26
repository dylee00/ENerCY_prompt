# Backend FastAPI Skill

## name
backend-fastapi

## description
기존 Python 모델 코드를 FastAPI backend에 안전하게 연결하고, threshold 기반 성능 모니터링 및 재학습 트리거, 프론트엔드 명세에 맞는 JSON API를 구현할 때 사용하는 Skill이다.

이 Skill은 아래 상황에서 사용한다.

- 기존 모델 코드를 backend에서 import해서 사용해야 할 때
- import 시점 부작용(자동 학습, 자동 plot 생성, 자동 model load)을 제어해야 할 때
- FastAPI route를 프론트엔드 명세에 맞게 구현해야 할 때
- threshold 기준으로 모델 상태를 판단하고 재학습을 트리거해야 할 때
- recommendation 판단을 backend에 두고 frontend는 표시만 하도록 설계해야 할 때

---

## 보안 및 환경 변수 규칙

실제 API key, secret, token은 이 문서에 절대 평문으로 넣지 않는다.

필요한 값은 아래처럼 **환경 변수 이름만** 사용한다.

- `OPENAI_API_KEY`
- `MODEL_PROVIDER_API_KEY`
- `RETRAIN_RMSE_THRESHOLD`
- `WARNING_RMSE_THRESHOLD`
- `DEFAULT_FORECAST_HORIZON`

실제 값은 아래 중 하나로 관리한다.

1. 루트 `.env`
2. 셸 환경 변수
3. 배포 환경 secret manager

원칙:
- `SKILL.md`에 실제 비밀값을 적지 않는다.
- git에 secret이 올라가지 않게 한다.
- backend는 `os.getenv()` 또는 settings 객체로 환경 변수를 읽는다.

---

## 현재 base code에서 먼저 인지해야 할 문제

### 1. `main.py`의 import 구조 문제

현재 `main.py`는 아래를 상단 import로 불러온다.

- `.model`
- `.weight_used_model`

하지만 이 두 모듈은 import 시점에 무거운 작업이 발생할 수 있다. FastAPI 서버 시작 시 import만으로 학습, 모델 로딩, plot 생성이 일어나면 안 된다.

또한 현재 `main.py`는 상대 import와 절대 import가 섞여 있다.

예:
- `from . import model`
- `from . import weight_used_model`
- `from config import UPLOAD_DIR, IMAGE_DIR, MODEL_IMG_DIR`

backend 구현 시 import 경로를 일관되게 정리해야 한다.

---

### 2. `model.py`의 부작용 문제

현재 `model.py`는 import 시점에 아래를 수행한다.

- CSV 데이터 로드
- 데이터 전처리
- LSTM 학습
- 모델 저장
- 모델 구조 이미지 저장

즉, module import만 해도 학습이 수행된다. backend에서는 이 구조를 그대로 사용하면 안 된다.

원칙:
- 학습은 함수 호출 시점에만 수행되게 만든다.
- import 시에는 함수, 클래스, 설정만 로드되게 만든다.

---

### 3. `weight_used_model.py`의 부작용 문제

현재 `weight_used_model.py`는 import 시점에 아래를 수행한다.

- 저장 모델 로드
- 데이터 로드
- 모델 구조 이미지 저장

이 역시 backend import 시점 부작용이다. 가능하면 model load와 plot 생성은 함수 호출 시점으로 미룬다.

---

### 4. metric 반환 형식 문제

현재 `model.py`, `weight_used_model.py`의 `process()` 함수는 RMSE를 숫자가 아니라 문자열로 반환한다.

예:
- `The root mean squared error is 12.34.`

backend는 threshold 비교를 위해 이를 숫자로 변환해야 한다.

---

### 5. API 구조 문제

현재 `main.py`는 `/upload`, `/download`, `/download_shapes` 등 실습용 route 중심이다.

이번 프로젝트에서는 반드시 아래 API를 구현해야 한다.

- `GET /api/v1/dashboard`
- `GET /api/v1/forecast`
- `GET /api/v1/recommendation`
- `GET /api/v1/monitoring`

---

## 확정된 백엔드 합의 반영 규칙

이 Skill은 아래 합의안을 기준으로 구현한다.

### 1. summary 계산 구간

아래 값은 **실질 타겟 구간(16~23일)** 기준으로 계산한다.

- `forecastSummary.predictedMaxPrice`
- `forecastSummary.predictedMinPrice`
- `priceContext.predictedMaxPrice`
- `priceContext.predictedMinPrice`
- `expectedSpread`

즉:
- dashboard summary는 16~23일 기준
- recommendation priceContext도 16~23일 기준

---

### 2. monitoring metric 구조

`latestEvaluation`은 **구간별 중첩 구조**를 사용한다.

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

---

### 3. 재학습 판단 방식

재학습 필요 여부는 **백엔드가 MLflow에서 최신 지표를 읽어 판단**한다.

즉:
- 모델이 backend endpoint를 직접 호출하지 않는다.
- backend가 최신 metric을 읽고 상태를 계산한다.

---

### 4. horizon 기준

- 최대 horizon: `23`
- `/forecast` 기본 horizon: `23`
- dashboard / recommendation의 summary 계산 기준: `16~23일`

---

### 5. threshold 1차 기준 metric

재학습 필요 여부와 상태 판단의 1차 기준 metric은 **`midTerm.rmse`** 다.

권장 해석:
- `HEALTHY`: `midTerm.rmse <= WARNING_RMSE_THRESHOLD`
- `WARNING`: `WARNING_RMSE_THRESHOLD < midTerm.rmse <= RETRAIN_RMSE_THRESHOLD`
- `CRITICAL`: `midTerm.rmse > RETRAIN_RMSE_THRESHOLD`

`shortTerm` metric은 monitoring 참고용으로 함께 제공하되, 1차 재학습 판단 기준으로는 사용하지 않는다.

---

### 6. frontend pending agreement 반영

#### PA-001. highlightTarget
현재 단계에서는 backend 응답에 `highlightTarget`을 추가하지 않는다.

원칙:
- recommendation, threshold, 재학습 판단은 backend 책임
- 카드 강조, 배지 색상, 시각 효과는 frontend 표현 책임

#### PA-002. 예측 구간 식별자
`/api/v1/forecast`의 각 `forecastPoints` 항목에는 `periodType`을 포함한다.

허용 값:
- `WAITING`
- `TARGET`

#### PA-003. dashboard summary 산출 기준
dashboard의 `predictedMaxPrice`, `predictedMinPrice`는 **16~23일 target period 기준**이다.

#### PA-004. monitoring 지표 응답 구조
`latestEvaluation`은 **중첩 구조**를 사용하며, UI는 이 구조를 그대로 사용한다.

---

## 이 Skill을 사용할 때의 구현 원칙

### 원칙 1. Route는 얇게 유지한다

route는 아래만 담당한다.

- query parameter 검증
- service 호출
- response 반환
- 예외 변환

비즈니스 로직은 route 안에 쓰지 않는다.

---

### 원칙 2. 서비스 계층을 둔다

최소한 아래 역할은 service 또는 adapter 계층으로 분리한다.

- model import 및 호출
- metric 추출 및 정규화
- MLflow 최신 metric 조회
- threshold 비교
- monitoring 상태 계산
- retraining trigger
- recommendation 계산
- dashboard 응답 조립

---

### 원칙 3. recommendation 로직은 backend에 둔다

recommendation 판단은 frontend가 아니라 backend가 수행해야 한다.

backend는 최소한 아래를 계산해야 한다.

- 현재 가격
- 미래 예측값 목록
- 최대 예측 유가
- 최소 예측 유가
- expectedSpread
- BUY / HOLD
- 추천 이유 요약
- thresholdStatus

frontend는 계산하지 않고 표시만 해야 한다.

---

### 원칙 4. threshold는 하드코딩하지 않는다

threshold 값은 route 내부 매직 넘버로 넣지 않는다.

권장 방식:
- `config.py`
- 환경 변수
- settings 객체

중 하나로 관리한다.

---

### 원칙 5. toy project에 맞는 현실적인 재학습 전략을 쓴다

현재 프로젝트는 토이 프로젝트이므로, threshold 기준 미달 시 동기 재학습을 허용할 수 있다.

단, 재학습 로직은 route 안에 직접 쓰지 말고 service 함수로 분리한다.

---

## 권장 파일 구조

현재 구조를 완전히 뒤엎지 말고, backend 구현을 위해 필요한 최소 파일만 추가한다.

예시:
- `main.py`
- `config.py`
- `schemas.py`
- `services/model_adapter.py`
- `services/monitoring_service.py`
- `services/retraining_service.py`
- `services/recommendation_service.py`

필요 시:
- `routers/dashboard.py`
- `routers/forecast.py`
- `routers/recommendation.py`
- `routers/monitoring.py`

토이 프로젝트라면 router 분리 없이 `main.py + schemas.py + services/*` 정도까지만 해도 충분하다.

---

## model adapter 구현 기준

adapter는 기존 `model.py`, `weight_used_model.py`를 backend 친화적으로 감싸는 계층이다.

최소 기능:
- 예측 실행
- metric 추출
- RMSE 문자열을 숫자로 변환
- MLflow metric 조회에 필요한 키 정리
- retrain 진입점 제공
- 예측 결과를 backend schema에 맞게 변환

최소 함수 예시:
- `predict(oil_type: str, horizon: int) -> dict`
- `evaluate(oil_type: str) -> dict`
- `retrain(oil_type: str) -> dict`
- `get_latest_metrics(oil_type: str) -> dict`

주의:
- 기존 `process()`가 이미지 경로와 RMSE 문자열만 반환하는 구조라면, adapter에서 숫자 metric과 예측 결과 구조를 재조립해야 한다.
- 불가능하면 model 쪽 최소 수정은 허용한다.

---

## monitoring 상태 계산 기준

최소 상태:
- `HEALTHY`
- `WARNING`
- `CRITICAL`

최소 응답 항목:
- `modelStatus`
- `retrainingRequired`
- `latestModelVersion`
- `latestEvaluation`
- `message`

권장 추가 항목:
- `evaluationThresholds`

권장 흐름:
1. MLflow 또는 최신 평가 결과에서 metric 수집
2. `midTerm.rmse` 추출
3. threshold 비교
4. 상태 계산
5. `retrainingRequired` 계산
6. 기준 미달 시 retrain 실행 여부 결정
7. monitoring 응답 반환

---

## recommendation 계산 기준

최소 입력:
- 현재 가격
- forecast points
- threshold 정책

최소 출력:
- `action`
- `label`
- `reasonSummary`
- `thresholdStatus`

권장 액션:
- `BUY`
- `HOLD`

추가 원칙:
- recommendation 계산에 사용하는 max/min/spread는 **16~23일 타겟 구간 기준**으로 계산한다.

---

## 필수 API 구현 기준

### `GET /api/v1/dashboard`

반드시 아래를 한 번에 반환해야 한다.

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

위 두 값은 **16~23일 기준**이다.

---

### `GET /api/v1/forecast`

반드시 아래를 반환해야 한다.

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
- 요청값이 없으면 전체 예측 구간을 반환한다.
- 각 `forecastPoints` 항목은 `periodType`을 포함해야 한다.
- `periodType`은 `WAITING` 또는 `TARGET` 값을 가진다.

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

---

### `GET /api/v1/recommendation`

반드시 아래를 반환해야 한다.

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

위 값은 **16~23일 기준**이다.

---

### `GET /api/v1/monitoring`

반드시 아래를 반환해야 한다.

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
- 상태 및 재학습 판단의 기준은 `midTerm.rmse`다.

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
  },
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

---

## 공통 에러 응답 형식

반드시 아래 형식을 사용한다.

```json
{
  "code": "BAD_REQUEST",
  "message": "요청 파라미터가 올바르지 않습니다.",
  "details": {}
}
```

최소 처리 대상:
- 잘못된 `oilType`
- 잘못된 `horizon`
- 모델 파일 없음
- 예측 실패
- 재학습 실패
- 내부 서버 오류

---

## 실행 환경 규칙

- 데이터 / 모델 / 백엔드는 동일한 `ener` 가상환경을 사용한다.
- Python 3.11 기준으로 작성한다.
- FastAPI 기반으로 구현한다.
- 공용 의존성은 루트 `requirements.txt` 기준으로 맞춘다.

---

## 구현 우선순위

### 1순위
- import 부작용 제거 또는 우회
- metric 문자열 파싱 및 정규화
- MLflow 최신 metric 조회 흐름 구현
- response schema 정의
- `/api/v1` route 골격 구현

### 2순위
- monitoring / retraining service 분리
- recommendation service 분리
- dashboard 응답 조립

### 3순위
- 기존 `/upload`, `/download` 계열 route와 공존 여부 정리
- config 확장
- 검증 코드 정리

---

## 완료 기준

이 Skill을 사용한 backend 작업은 아래를 만족해야 완료다.

- `/api/v1/dashboard` 구현
- `/api/v1/forecast` 구현
- `/api/v1/recommendation` 구현
- `/api/v1/monitoring` 구현
- 공통 에러 응답 형식 구현
- threshold 비교 가능
- 기준 미달 시 재학습 트리거 가능
- recommendation 판단이 backend에 위치
- `forecastPoints[].periodType`가 반영됨
- monitoring 응답에 중첩형 `latestEvaluation` 구조가 반영됨
- `midTerm.rmse` 기준 상태 판단이 가능함
- import 부작용에 대한 방어 또는 리팩터링 반영
- `ener` 가상환경 기준으로 실행 가능
