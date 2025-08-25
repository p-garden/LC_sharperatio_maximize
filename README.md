# 📊 Lending Club Credit Risk Analysis Project

## 📌 프로젝트 개요
본 프로젝트는 Lending Club 대출 데이터를 활용하여 **부도 여부 예측 및 Sharpe Ratio 극대화 전략**을 수립하는 것을 목표로 합니다.  
단순히 대출 상환 여부(0/1)를 예측하는 수준을 넘어, **IRR(내부수익률)을 계산하고 무위험 수익률과 비교한 초과수익률(Excess Return)**을 타깃값으로 설정하여, 실제 투자자의 관점에서 모델을 평가했습니다.

[![리포트 미리보기](docs/report_preview.png)](docs/5조%20최종보고서.pdf)
---

## 🗂 데이터
- **출처**: Lending Club 공개 데이터
- **기간**: 2007 ~ 2018
- **주요 변수**
  - `loan_amnt`: 대출 금액
  - `int_rate`: 대출 이자율
  - `annual_inc`: 연소득
  - `loan_status`: 상환 여부
  - `total_pymnt`, `total_rec_prncp`, `recoveries` 등 상환 관련 변수
  - 차주 신용도 및 금융 거래 기록 관련 변수

---

## ⚙️ 분석 과정
1. **데이터 전처리**
   - 결측치 처리 및 이상치 제거
   - 파생변수 생성 (ROI, DTI, IRR 등)
   - 로그 변환 및 정규화

2. **모델링**
   - **분류 모델 (Default 여부 예측)**  
     - Logistic Regression / Random Forest / LightGBM / CatBoost / SVM  
     - 앙상블 및 스태킹 모델 적용  
     - Hyperparameter Tuning → Random Search 활용  

   - **회귀 모델 (투자 수익률 기반)**  
     - 각 대출별 **IRR(Internal Rate of Return)** 계산  
     - **무위험 수익률(rf)** 과 비교하여 `Excess Return = IRR - rf` 를 타깃값으로 설정  
     - 즉, 단순 상환 여부 예측이 아닌 **투자자의 초과수익률 극대화**를 목표로 학습  
     - IRR 기반 예측값을 Threshold 기준으로 변환해 **이진 분류 대체 가능**

3. **평가 지표**
   - 일반 지표: Accuracy, ROC AUC, F1-score
   - 금융 지표: **Sharpe Ratio, IRR, Investor Expected Cost**
   - Threshold 최적화: `λ = FN Loss / TP Profit` 기반 커스텀 함수  

---

## 📈 주요 결과
- Sharpe Ratio 기준 평균 **1.39 ~ 2.0** 확보
- 무위험 수익률보다 낮은 경우 → 초과수익률을 0으로 대체하여 Sharpe Ratio 계산
- 회귀 모델(IRR 기반)을 적용했을 때 투자 성과가 더욱 개선됨  
- Threshold를 높이면 Sharpe Ratio는 올라가지만 승인률이 낮아지는 trade-off 발생

---

## 📊 시각화
- **대출 상태별 ROI 분포**
- **Sharpe Ratio 분포 (50회 반복 실험)**
- **Threshold 변화에 따른 Precision-Recall 곡선**

---
## 💡 결론 및 한계
- **결론**: IRR 기반 초과수익률을 타깃값으로 설정한 모델이 단순 분류 모델보다 Sharpe Ratio 기준에서 더 높은 투자 성과를 보여줌  
- **한계**: 실제 환경에서는 금리 변동, 경기 사이클, 조기 상환 등의 외부 요인을 반영할 필요 있음  
- **향후 계획**: 
  - 외부 거시경제 변수(금리, CCSI 등) 결합  
  - Explainable AI 기법 적용  
  - 포트폴리오 최적화 전략 확장  

---