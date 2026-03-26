# oil-series-definition.md

## 문서 목적

이 문서는 프로젝트에서 사용하는 오일 종류와 각 오일의 비즈니스 역할, feature, 예측 target을 정의한다.

---

## 사용 오일 종류

이 프로젝트에서는 전체 5개 오일 중 아래 3개를 사용한다.

| ticker | commodity | 비즈니스 역할 |
|--------|-----------|-------------|
| CL=F | Crude Oil | 구매 원가 예측 |
| HO=F | Heating Oil | 판매가 예측 |
| RB=F | RBOB Gasoline | 판매가 예측 |

---

## 미사용 오일 종류

| ticker | commodity | 미사용 이유 |
|--------|-----------|------------|
| BZ=F | Brent Crude Oil | 데이터 수 부족 (4,196개로 타 oil 대비 적음) |
| NG=F | Natural Gas | SK에너지 사업 영역과 거리가 있음 |

---

## 오일별 비즈니스 로직

### Crude Oil (CL=F)
- 역할: **구매 원가**
- 목적: 가장 저렴한 구매 시점 파악
- 예측 target: **Low (저가)**
- 이유: 원유를 최저가에 구매하는 시점을 찾기 위함

### Heating Oil (HO=F)
- 역할: **판매 제품**
- 목적: 가장 비싼 판매 시점 파악
- 예측 target: **High (고가)**
- 이유: 정제 제품을 최고가에 판매하는 시점을 찾기 위함

### RBOB Gasoline (RB=F)
- 역할: **판매 제품**
- 목적: 가장 비싼 판매 시점 파악
- 예측 target: **High (고가)**
- 이유: 정제 제품을 최고가에 판매하는 시점을 찾기 위함

---

## 오일별 입력 Feature 및 예측 Target 정의

| commodity | 입력 Feature | 예측 Target |
|-----------|-------------|------------|
| Crude Oil | Low, Close, Volume | Low |
| Heating Oil | High, Close, Volume | High |
| RBOB Gasoline | High, Close, Volume | High |

---

## 모델 독립성

- 3개 오일 각각에 대해 **독립적인 LSTM 모델**을 학습한다.
- 모델 구조와 하이퍼파라미터는 동일하게 유지한다.
- 입력 feature와 예측 target만 오일별로 다르다.

---

## 예측 구간 정의

- 배송 리드타임: 15일
- 예측 구간: **Day 16 ~ Day 23 (8일)**
- 이유: 오늘 구매 결정 시 실제 도착 이후 약 1주일간의 가격을 예측하여 최적 구매/판매 시점 파악

---

## 문서 상태

- 상태: 초안
- 작성일: 2026-03-25
- 마지막 갱신일: 2026-03-25
