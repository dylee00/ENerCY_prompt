# dataset-source.md

## 문서 목적

이 문서는 AI 기반 에너지 조달 의사결정 지원 솔루션에서 사용하는 데이터셋의 출처와 구성을 정의한다.

---

## 데이터셋 정보

- 이름: Fuels Futures Data
- 출처: Kaggle
- URL: https://www.kaggle.com/datasets/guillemservera/fuels-futures-data
- 라이선스: Kaggle 데이터셋 이용 약관 준수

---

## 데이터 기간

- 시작일: 2000-08-23
- 종료일: 2024-06-24
- 총 기간: 약 24년

---

## 파일 구성

| 파일명 | 설명 |
|--------|------|
| all_fuels_data.csv | 전체 오일 종류 통합 파일 (메인 사용 파일) |
| individual_data/Crude_Oil_data.csv | Crude Oil 개별 파일 |
| individual_data/Heating_Oil_data.csv | Heating Oil 개별 파일 |
| individual_data/RBOB_Gasoline_data.csv | RBOB Gasoline 개별 파일 |
| individual_data/Brent_Crude_Oil_data.csv | Brent Crude Oil 개별 파일 (미사용) |
| individual_data/Natural_Gas_data.csv | Natural Gas 개별 파일 (미사용) |

---

## 사용 파일

이 프로젝트에서는 `all_fuels_data.csv` 를 메인 파일로 사용한다.

---

## 데이터 저장 경로

- 원본 데이터: `data/raw/fuels_futures_data/`
- 전처리 중간 데이터: `data/interim/`
- 모델 입력용 데이터: `data/processed/`

---

## 문서 상태

- 상태: 초안
- 작성일: 2026-03-25
- 마지막 갱신일: 2026-03-25
