# Credit Card Default Risk Model

## Overview
This project builds an **end-to-end credit risk scoring system** using the UCI Credit Card Default dataset.  
The objective is not to chase metrics, but to design a **realistic, business-usable risk model** that generalises well and translates predictions into actionable decisions.

---

## Dataset
- **Source:** UCI Credit Card Default Dataset  
- **Samples:** 30,000 customers  
- **Target:** Default in the next month (binary)  
- **Default rate:** ~22%

---

## Feature Engineering

Raw monthly variables were transformed into **behavioural risk signals**, reflecting how credit risk is assessed in practice.

### Behavioural Risk (Most Important)
- `recent_delay` – recency of delinquency  
- `num_delayed_months` – frequency of late payments  
- `max_delay` – severity of delinquency  

### Payment Discipline
- `avg_payment`  
- `payment_ratio`  
- `recent_payment_ratio`  

### Capacity & Exposure
- `credit_limit`  
- `avg_bill`  
- `utilization_ratio`  

Extreme ratios were **clipped** to reduce sensitivity to outliers(as linear models have difficulty for outliers).

---

## Feature Selection Decisions

### Dropped Demographic Features
- `gender`, `education`, `marital_status`

**Why:**
- Low incremental predictive power  
- Risk of proxy bias  
- Credit risk should be driven by repayment behavior, not personal attributes  

### Dropped Engineered Features
- `mean_delay`
- `any_severe_delay`

**Reason:**
- Redundant with core delay features  
- Signal already captured by `recent_delay`, `max_delay`, and `num_delayed_months`

---

## Train / Validation / Test Strategy
- Stratified splits to preserve default rate  
- Strict separation to avoid leakage  
- Final evaluation performed on a held-out test set  

---

## Models Evaluated

### Logistic Regression
- Baseline model for interpretability and feature validation  
- Confirmed dominance of behavioural risk features  

### Random Forest (Constrained)
- Shallow trees and large leaf sizes to ensure stability  
- Balanced non-linearity with generalisation  

### Gradient Boosting
- Training ROC improved with tuning  
- Validation ROC stagnated or declined  
- Indicated overfitting and signal ceiling reached  

---

## Model Comparison

| Model               | Validation ROC-AUC | Test ROC-AUC | Risk Bucket Quality | Verdict |
|---------------------|-------------------:|-------------:|---------------------|---------|
| Logistic Regression | ~0.74              | ~0.74        | Good                | Baseline |
| Random Forest       | **~0.77**          | **~0.77**    | **Excellent**       | **Final Model** |
| Gradient Boosting   | ~0.77              | ~0.77        | Unstable            | Rejected (Overfit) |

---

## Model Selection
**Final Model:** Random Forest

Selection was based on:
- Validation performance  
- Generalization behavior  
- **Quality of percentile-based risk buckets**, not ROC alone  

---

## Risk Buckets (Business Evaluation)

Predicted probabilities were converted into **percentile-based risk buckets**.

### Test Set Default Rates

| Risk Bucket | Default Rate |
|------------|--------------|
| Very Low   | ~5.6%        |
| Low        | ~10.1%       |
| Medium     | ~14.2%       |
| High       | ~23.7%       |
| Very High  | ~57.1%       |

- Strictly monotonic risk increase  
- ~**10× separation** between safest and riskiest groups  
- Stable behavior on unseen data  

This indicates **production-grade ranking performance**.

---

## Key Learnings
- Credit risk models rank **relative risk**, not certainty  
- Behavioural features dominate demographic attributes  
- More complex models do not guarantee better business outcomes  
- Knowing **when to stop tuning** is critical  

---

## Limitations & Next Steps

**Limitations**
- No temporal validation  
- No cost-sensitive optimisation  
- No drift monitoring  

**Next Steps**
- Expected loss–based decision thresholds  
- Monitoring and retraining strategy  
- Deployment as a scoring service  

---

## Conclusion
This project demonstrates a **clean, realistic credit risk modelling pipeline**, from feature design to model selection. 
The emphasis is on **decision quality and generalisation**, not metric chasing.
