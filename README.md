# E-Commerce Fraud Detection System

End-to-end data science pipeline for real-time fraud risk scoring, built in a security intelligence firm (SIS) context. Covers exploratory analysis, feature engineering, XGBoost modeling, threshold optimization via cost-benefit analysis, and an operational Tableau dashboard.

---

## Project Overview

This project simulates the full lifecycle of a production fraud detection system — from raw transaction data to a business-ready monitoring dashboard. The workflow mirrors what an analytics team at a financial crimes or security intelligence firm would deliver: interpretable signals, model transparency via SHAP, and threshold decisions grounded in dollar-value trade-offs rather than pure accuracy metrics.

---

## Dataset

| Attribute | Value |
|---|---|
| Transactions | 299,695 |
| Features | 17 columns |
| Fraud rate | 2.21% (class imbalance handled with `scale_pos_weight=44`) |
| Source | [E-Commerce Fraud Detection Dataset](https://www.kaggle.com/) — Kaggle, CC0 Public Domain |

---

## Methodology

### 1. EDA
- Fraud rate segmented by country, hour-of-day, merchant category, and security flag combinations
- Correlation heatmap across all numeric features
- Distribution analysis: fraudulent vs. legitimate transaction amounts

### 2. Feature Engineering
Six rule-based features derived from domain knowledge:

| Feature | Description |
|---|---|
| `security_fails` | Composite score (0–3): CVV fail + AVS fail + 3D Secure fail |
| `amount_vs_avg` | Transaction amount divided by user's historical average spend |
| `country_mismatch` | Flag: card country ≠ transaction country |
| `all_security_failed` | Boolean: all three security checks failed simultaneously |
| `mismatch_and_security_fail` | Boolean: country mismatch AND ≥1 security fail |
| `amount_spike` | Boolean: transaction ≥ 3× user's average spend |

### 3. Modeling
- **Algorithm:** XGBoost (`n_estimators=500`, `max_depth=8`, `learning_rate=0.03`)
- **Class imbalance:** `scale_pos_weight=44` (ratio of negatives to positives)
- **Interpretability:** SHAP values for global and local feature attribution

### 4. Threshold Optimization
Threshold selected via cost-benefit analysis rather than F1 maximization:
- Analyst rate: $35/hr
- Review time: 15 min → $8.75 per alert
- Average fraud value: $590
- Optimal threshold: **0.12** (maximizes net business value per period)

---

## Key EDA Findings

- CVV fail = **20× fraud multiplier** vs. baseline
- AVS fail = **18× fraud multiplier** vs. baseline
- Card/transaction country mismatch + 2 security fails → **41.4% fraud rate** (18.8× baseline)
- High-risk group (all 5 conditions met) → **44.71% fraud rate**
- Fraudulent transactions average **$590** vs. $168 for legitimate (3.5× higher)
- Shipping distance was the **#1 SHAP feature** by importance

---

## Model Results

| Metric | Value |
|---|---|
| Algorithm | XGBoost |
| ROC-AUC | **0.9755** |
| Recall | **94%** (catches 1,239 of 1,322 fraud cases in test set) |
| Precision | 20% |
| Threshold | 0.12 (cost-optimized) |
| Engineered features in top 8 SHAP | 3 of 8 |

---

## Business Impact

| Metric | Value |
|---|---|
| Net value per 60k transactions | **$677,049** |
| Annualized estimate | **$3,385,244** |
| Analyst cost at threshold 0.12 | $53,961 per period |
| False positives flagged | 4,928 |
| Fraud cases missed | 83 of 1,322 (6.3%) |

> At threshold 0.12, each false alarm costs $8.75 in analyst time. Each caught fraud saves $590. The math strongly favors a lower threshold despite increased alert volume.

---

## Stack

| Category | Tools |
|---|---|
| Language | Python |
| Data | pandas, numpy |
| Modeling | XGBoost, scikit-learn |
| Explainability | SHAP |
| Visualization | seaborn, matplotlib, Tableau |
| Environment | Jupyter, SQL |

---

## Dashboard

4-panel Tableau control monitoring report built on `outputs/scored_transactions.csv`:

1. **Daily fraud rate trend** — time-series view of flagged transaction rates
2. **Risk tier breakdown** — Low / Medium / High / Critical segment volumes
3. **Fraud probability distribution** — histogram of model score output
4. **High-risk transactions table** — drilldown on flagged cases with key features

---

## How to Run

```bash
# 1. Clone the repo
git clone <repo-url>
cd fraud-detection-project

# 2. Install dependencies
pip install -r requirements.txt

# 3. Open the notebook
jupyter notebook notebooks/01_eda.ipynb

# 4. Run all cells in order
```

---

## Repository Structure

```
fraud-detection-project/
├── notebooks/
│   └── 01_eda.ipynb          # EDA, feature engineering, modeling, threshold analysis
├── outputs/
│   └── scored_transactions.csv  # Model-scored transactions (gitignored)
├── data/                        # Raw data (gitignored)
├── requirements.txt
└── README.md
```
