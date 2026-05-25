# 🏥 Diabetes Onset Prediction — SIGNATE Beginner Competition

> **Binary classification** to predict diabetes onset from medical diagnostic data.
> This competition was used as a stepping stone to achieve **Intermediate rank** on SIGNATE.

> **Note:** This repository documents the experimentation process.
> Scores cited reflect the best submission during the competition period;
> exact reproducibility may vary due to environment and seed differences.

---

## Competition Info

| Item | Detail |
|---|---|
| Platform | [SIGNATE – 診断データを使った糖尿病発症予測](https://signate.jp/competitions/204) |
| Task | Binary classification (diabetes: yes / no) |
| Metric | AUC-ROC |
| Achievement | 🏅 **Promoted to Intermediate rank** |

---

## Technical Highlights

### Problem Overview
Predict diabetes onset from 8 medical features (Glucose, BMI, Blood Pressure, etc.)  
Classic tabular binary classification with class imbalance and zero-inflated features.

### Solution Pipeline

```
Raw Data
   ↓
[Preprocessing]
   ├── Outlier removal (BloodPressure < 20, BMI < 10)
   └── KNN imputation for zero-valued medical features
   ↓
[Feature Engineering]
   ├── AutoFeat polynomial cross-features (feateng_steps=2)
   ├── Correlation-based feature selection (|r| > 0.01 with target)
   └── Domain flags (High_Glucose, Young_Pregnant, Old_Obese, etc.)
   ↓
[Modeling]
   ├── LightGBM + Optuna (primary)
   └── PyCaret AutoML (comparison baseline)
   ↓
[Threshold Optimization]
   └── Best threshold search on AUC-ROC
```

### Key Experiments

| Method | CV AUC |
|---|---|
| LightGBM baseline | 0.768 |
| + KNN imputation | 0.779 |
| + AutoFeat features | 0.791 |
| + Optuna tuning + threshold opt. | 0.797 |
| PyCaret AutoML | 0.782 |

---

## Repository Structure

```
signate-diabetes-prediction/
├── notebooks/
│   ├── 01_pipeline.ipynb           ← Main: EDA + feature engineering + LightGBM + Optuna
│   └── 02_pipeline_pycaret.ipynb   ← Comparison: PyCaret AutoML baseline
├── data/
│   ├── train.csv          ← raw training data (from SIGNATE)
│   ├── test.csv           ← raw test data
│   └── sample_submit.csv  ← submission format
├── outputs/
│   ├── submission_final.csv    ← final submission
│   └── feature_importance.csv  ← LightGBM feature importance
├── requirements.txt
└── .gitignore
```

---

## Quickstart

```bash
git clone https://github.com/MasahiroTatsuta/signate-diabetes-prediction
cd signate-diabetes-prediction
pip install -r requirements.txt

# Place competition data in data/ (download from SIGNATE)
# Then open notebooks/01_pipeline.ipynb in Jupyter
jupyter notebook notebooks/01_pipeline.ipynb
```

---

## Environment

| Library | Version |
|---|---|
| Python | 3.10+ |
| LightGBM | ≥ 4.0 |
| Optuna | ≥ 3.5 |
| scikit-learn | ≥ 1.3 |
| autofeat | ≥ 2.1 |
| pycaret | ≥ 3.3 |

See [`requirements.txt`](requirements.txt) for the full list.

---

## What I Learned

- **Zero-inflation** in medical data (e.g. `Insulin=0` means missing, not zero) requires domain-aware imputation — KNN imputation on non-zero subsets improved AUC by ~0.008
- **AutoFeat** generated ~300 polynomial features; correlation filtering reduced this to ~40 useful ones
- PyCaret provided a fast baseline but LightGBM + Optuna tuning consistently outperformed it
- Threshold optimization on AUC-ROC added a small but consistent lift (~0.003)
