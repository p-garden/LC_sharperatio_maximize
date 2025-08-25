# 📊 Lending Club Credit Risk Analysis Project

## 📌 프로젝트 개요
본 프로젝트는 Lending Club 대출 데이터를 활용하여 부도 여부 예측 및 Sharpe Ratio 극대화 전략을 수립하는 것을 목표로 합니다.  
단순히 대출 상환 여부(0/1)를 예측하는 수준을 넘어, IRR(내부수익률)을 계산하고 무위험 수익률과 비교한 초과수익률(Excess Return)을 타깃값으로 설정하여, 실제 투자자의 관점에서 모델을 평가했습니다.

👉 IRR(Internal Rate of Return, 내부수익률)은 투자로부터 발생하는 미래 현금흐름(cash flow)을 현재 가치로 환산했을 때, 순현재가치(NPV)가 0이 되도록 하는 할인율입니다. 다시 말해, 투자자가 해당 대출에 투자했을 때 원금과 이자 상환을 현재 시점으로 할인하여 수익률을 계산하는 방식입니다. 본 프로젝트에서는 각 대출의 월별 상환금(정상 상환, 조기 상환, 부도 및 회수금 포함)을 기반으로 현금흐름을 구성하고, 이를 이용해 IRR을 산출했습니다【40†source】. 이 과정을 통해 단순 부도 여부가 아닌 실제 투자 성과를 반영할 수 있었습니다.


### 📐 IRR & Sharpe Ratio 계산 로직

- **IRR (Internal Rate of Return)**  
  IRR은 각 대출의 상환 상태에 따라 현금흐름을 다르게 정의하여 계산했습니다.  
  1) 정상 만기 상환: 계약된 기간(term) 동안 매월 동일한 상환금(`installment`)이 유입됩니다.  
  2) 조기 상환: 총 상환금(`total_pymnt`)과 마지막 상환액(`last_pymnt_amnt`)을 활용하여 실제 상환 기간까지만 반영했습니다. 특히 상환 기간이 1개월인 경우, 최초 투자금(-loan_amnt)과 전액 상환금만 고려했습니다.  
  3) 부도 발생: 일부 상환금(`installment`)이 유입된 후, 마지막 달에 실제 납부액(`last_pymnt_amnt`)과 회수금(`recoveries`)을 추가 반영했습니다.  

  이러한 방식으로 각 대출별 현금흐름(cash flow) 리스트를 구성한 뒤, 이를 기반으로 `npf.irr()` 함수를 통해 IRR을 산출했습니다. 즉, IRR은 투자자가 받은 모든 현금흐름을 현재 가치로 할인했을 때 순현재가치(NPV)가 0이 되는 수익률을 의미합니다.

- **Sharpe Ratio**  
  산출된 IRR 값에서 무위험 수익률(rf, 미국 국채 3년/5년물)을 차감하여 초과수익률을 구했습니다. 이후 이 초과수익률의 평균을 표준편차로 나누어 Sharpe Ratio를 계산했습니다.  
  Sharpe Ratio는 투자 성과를 위험 대비 기준으로 평가하는 지표로, 초과수익률이 음수인 경우에는 무위험 자산 투자와 동일하게 처리하기 위해 0으로 대체한 후 계산했습니다.


[![리포트 미리보기](https://github.com/user-attachments/assets/8eca0a75-1903-466f-9568-d1e2d87c8efb)](docs/5조%20최종보고서.pdf)
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