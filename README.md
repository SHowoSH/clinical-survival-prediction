# clinical-survival-prediction
Survival analysis and risk stratification using CoxPH and Random Survival Forest on TCGA clinical data.
# 임상 데이터를 활용한 생존 분석 및 위험도 예측 모델

[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3.0-orange?logo=scikit-learn)](https://scikit-learn.org/)
[![Lifelines](https://img.shields.io/badge/Lifelines-0.27.0-blue)](https://lifelines.readthedocs.io/)
[![scikit-survival](https://img.shields.io/badge/scikit--survival-0.21.0-green)](https://scikit-survival.readthedocs.io/)

> **TCGA의 임상 데이터만을 사용하여, (1) 생존 예측, (2) 위험도 계층화, (3) 치료 효과 예측이라는 세 가지 핵심 과업을 수행하는 다각적 예후 예측 모델을 개발한 프로젝트입니다.**

---

## 1. 프로젝트 개요

-   **목표**: 오믹스 데이터 없이, 접근성 높은 **임상 정보(Clinical Data)만으로** 환자의 예후를 다각적으로 예측하는 모델의 가능성을 검증합니다.
-   **데이터**: TCGA 4대 암종 임상 데이터 (LIHC, KIRC, OV, STAD)
-   **수행 과업**:
    1.  **생존 예측 (Survival Prediction)**: 환자의 생존 기간과 사망 이벤트를 예측합니다.
    2.  **위험도 계층화 (Risk Stratification)**: 환자를 예후에 따라 저/중/고위험군으로 분류합니다.
    3.  **치료 효과 예측 (Treatment Effect Prediction)**: 치료 여부에 따른 예후 개선 효과를 예측합니다.

---

## 2. 분석 방법론

### 2.1. 데이터 전처리
-   **결측치 처리**: 분석의 안정성을 위해 결측률이 80% 이상인 변수는 제거하고, 수치형 변수는 **중앙값(Median)**, 범주형 변수는 **최빈값(Mode)**으로 대체했습니다.
-   **특성 선택**: **VIF (분산팽창지수)**를 이용해 다중공선성이 높은 변수들을 제거하여 모델의 안정성을 확보했습니다. (최종 16~19개 특성 사용)
-   **인코딩**: `One-Hot Encoding` 및 순서형 변수에 대한 `Label Encoding`을 적용했습니다.

### 2.2. 모델링 전략
본 프로젝트는 각 과업의 목적에 맞춰 다양한 생존 분석 및 머신러닝 모델을 구축하고 성능을 비교했습니다.

-   **Task 1: 생존 예측 (Survival Prediction)**
    -   **사용 모델**: `Cox Proportional Hazards (CoxPH)`, `Random Survival Forest (RSF)`, `Gradient Boosting Survival Analysis (GBSA)`
    -   **평가 지표**: `C-index (Concordance Index)`
-   **Task 2: 위험도 계층화 (Risk Stratification)**
    -   **방법**: Cox 모델로 위험 점수를 계산해 환자를 저/중/고위험군으로 정의한 뒤, 이 그룹을 예측하는 분류 모델을 학습했습니다.
    -   **사용 모델**: `Random Forest`, `Logistic Regression`, `XGBoost`
    -   **평가 지표**: `Accuracy`, `AUC`
-   **Task 3: 치료 효과 예측 (Treatment Effect Prediction)**
    -   **방법**: 치료 여부에 따른 예후가 양호할지를 예측하는 이진 분류 모델을 학습했습니다.
    -   **사용 모델**: `Random Forest`, `XGBoost`, `LightGBM`
    -   **평가 지표**: `Accuracy`, `AUC`

### 2.3. 모델 해석 (XAI)
-   `SHAP`과 `LIME`을 분류/회귀 모델에 적용하여, 어떤 임상 변수가 모델의 예측에 중요한 영향을 미치는지 분석하고 결과를 시각화했습니다.

---

## 3. 주요 결과

### 3.1. 모델별 성능 요약
각 과업에 대한 테스트 세트 기준 최종 성능은 다음과 같습니다.

| 분석 과업 | 모델 | 성능 지표 | 테스트 점수 |
| :--- | :--- | :--- | :--- |
| **생존 예측** | **Random Survival Forest** | **C-index** | **0.717** |
| | CoxPH (scikit-survival) | C-index | 0.638 |
| | GBSA | C-index | 0.707 |
| | CoxPH (Lifelines) | C-index | 0.667 |
| **위험도 계층화** | **XGBoost Classifier** | **AUC** | **0.978** |
| | Random Forest Classifier | Accuracy | 0.933 |
| | Logistic Regression | Accuracy | 0.933 |
| **치료 효과 예측**| **XGBoost Classifier** | **AUC** | **0.681** |
| | Random Forest Classifier | Accuracy | 0.627 |
| | LightGBM Classifier | Accuracy | 0.640 |

-   **결론**: 생존 예측에서는 **Random Survival Forest**가, 위험도 및 치료 효과 분류에서는 **XGBoost**가 가장 우수한 성능을 보였습니다.

### 3.2. 시각화 결과
모델이 예측한 위험군에 따른 생존율 차이를 **Kaplan-Meier 생존 곡선**으로 시각화하여, 고위험군과 저위험군이 명확하게 분리됨을 확인했습니다.

<p align="center">
  <img src="./assets/kaplan_meier_curve.png" alt="Kaplan-Meier Curve" width="600"/>
</p>

---

## 4. 사용 방법

1.  **Repository 클론 및 `requirements.txt` 설치**
2.  **노트북 파일 실행**:
    -   **`survival_prediction.ipynb`**: 생존 예측 모델(Cox, RSF, GBSA)을 훈련하고 평가합니다.
    -   **`risk_stratification_retry.ipynb`**: 위험도 계층화 모델(RF, Logistic, XGB)을 훈련하고 Kaplan-Meier 곡선을 생성합니다.
    -   **`treatment_effect_prediction.ipynb`**: 치료 효과 예측 모델(RF, XGB, LGBM)을 훈련하고 XAI 분석을 수행합니다.