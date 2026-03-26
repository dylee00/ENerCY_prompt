# preprocessing-policy.md

## 문서 목적

이 문서는 Fuels Futures 데이터셋의 전처리 규칙과 정규화 정책을 정의한다.

---

## 전처리 순서

1. 사용 오일 종류 필터링
2. 날짜 타입 변환
3. 날짜 기준 오름차순 정렬
4. 주말 제거
5. 공휴일(주중 비거래일) 보간
6. feature 선택
7. 정규화 (스케일링)

---

## 사용 오일 필터링

- `commodity` 컬럼 기준으로 아래 3개만 필터링한다.
  - Crude Oil
  - Heating Oil
  - RBOB Gasoline
- 각 오일별로 **독립적인 데이터프레임**으로 분리하여 처리한다.

---

## 날짜 처리

- `date` 컬럼을 `datetime` 타입으로 변환한다.
- 변환 후 오름차순 정렬한다.

```python
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)
```

---

## 주말 처리

- 토요일, 일요일은 선물거래소 휴장일이므로 **데이터에서 제거한다.**
- 주말은 보간 없이 그냥 없는 날로 처리한다.
- 이유: 주말 휴장은 모든 주에 동일하게 적용되어 시계열 패턴에 영향을 주지 않음

```python
df = df[df['date'].dt.dayofweek < 5].reset_index(drop=True)
```

---

## 공휴일(주중 비거래일) 보간 정책

주말을 제외한 주중 공휴일은 패턴의 연속성을 유지하기 위해 **보간으로 채운다.**

### 보간이 필요한 이유

- 월화수목금 패턴이 있다가 갑자기 월화목금만 있으면 LSTM이 잘못된 패턴을 학습할 수 있다.
- 예: 수요일 공휴일 → 화→목 사이의 하루 공백이 생겨 시계열 연속성이 깨짐
- 보간으로 채우면 LSTM이 자연스러운 흐름으로 학습 가능

### 케이스 1: 단일 공휴일 (1일)

```
월 화 [수 공휴일] 목 금
```

- 수요일 값 = (화요일 종가 + 목요일 시가) / 2
- open = high = low = close = 위 평균값
- volume = 0 (거래 없음)

```
보간값 = (전날_close + 다음날_open) / 2
```

### 케이스 2: 연속 공휴일 (2일 이상)

```
월 화 [수 목 연휴] 금
```

- 연휴 전날 종가(close)와 연휴 다음날 시가(open)를 기준으로 **선형 내분**한다.
- n = 연휴 일수, i = 연휴 내 순서 (1부터 시작)

```
i번째 날 보간값 = 전날_close + (다음날_open - 전날_close) × i / (n + 1)
```

예시 (수, 목 이틀 연휴):
```
수 = 화종가 + (금시가 - 화종가) × 1/3
목 = 화종가 + (금시가 - 화종가) × 2/3
```

- open = high = low = close = 보간값
- volume = 0 (거래 없음)

### 보간 적용 범위

- 주중(월~금) 기준으로 날짜 연속성을 확인한다.
- 데이터에 없는 주중 날짜를 감지하면 위 규칙으로 보간한다.
- 주말은 보간 대상에서 제외한다.

---

## Feature 선택

각 오일별로 아래 컬럼만 사용한다.

| commodity | 사용 컬럼 |
|-----------|----------|
| Crude Oil | date, low, close, volume |
| Heating Oil | date, high, close, volume |
| RBOB Gasoline | date, high, close, volume |

- `open` 컬럼은 `close`와 상관관계가 높아 제외한다.
- 오일별 예측 목적에 따라 `high` 또는 `low` 를 선택적으로 사용한다.

---

## 정규화 (스케일링)

### 사용 방법: RobustScaler

```
X_scaled = (X - median) / IQR
```

### RobustScaler를 선택한 이유

- 유가 데이터는 급등/급락(이상치)이 자주 발생한다.
  - 예: 2020년 코로나 당시 Crude Oil 선물가격이 음수까지 하락
- 24년치 데이터로 시간에 따라 가격 범위가 크게 변한다.
- RobustScaler는 중앙값(median)과 IQR 기준으로 변환하여 이상치에 가장 강건하다.

### 스케일링 적용 기준

- Scaler는 **Train 데이터로만 fit** 한다.
- Validation, Test, Hold-out은 Train 기준 Scaler로 **transform만** 한다.
- 이유: Data Leakage 방지 (미래 데이터의 통계 정보가 학습에 개입되면 안됨)

```python
from sklearn.preprocessing import RobustScaler

scaler = RobustScaler()
X_train_scaled = scaler.fit_transform(X_train)   # fit + transform
X_val_scaled   = scaler.transform(X_val)          # transform만
X_test_scaled  = scaler.transform(X_test)         # transform만
X_hold_scaled  = scaler.transform(X_hold)         # transform만
```

### 역정규화

- 모델 예측값은 스케일링된 값이므로 최종 출력 시 **역정규화(inverse_transform)** 를 적용한다.
- 역정규화를 통해 LSTM 출력값을 다시 실제 달러 단위로 변환한다.

```python
# 예측값 → 실제 가격으로 변환
predicted_price = scaler.inverse_transform(predicted_scaled)
```

---

## 문서 상태

- 상태: 업데이트
- 작성일: 2026-03-25
- 마지막 갱신일: 2026-03-26
