# Small-sample modeling and mechanism-guided design of polyamide nanofiltration membranes via Bayesian optimization

This repository contains the code accompanying the paper:

> **Small-sample modeling and mechanism-guided design of polyamide nanofiltration membranes via Bayesian optimization**
> Zihang Zhao *et al.*, *AIChE Journal* (under review).

The code implements a small-sample machine-learning workflow for polyamide nanofiltration membranes:

1. **XGBoost regression models** for membrane structure descriptors and separation performance, with hyperparameters tuned by **Bayesian optimization**.
2. **SHAP** and **Partial Dependence Plot (PDP)** analyses for mechanism-guided interpretation.
3. A **Genetic Algorithm (GA)** that uses the trained models to inversely design membrane fabrication recipes (PIP / TMC concentrations) that fall into user-specified target intervals for both the structural property (RP) and the separation performance (SJ).

---

## 1. Repository structure

```
.
├── train_models.py                 # Train RP and SJ XGBoost models, save to models/
├── run_ga_target1.py               # GA optimization for target interval 1
├── run_ga_target2.py               # GA optimization for target interval 2
├── structure_H2O_MgCl2.py          # Structure model for H2O / MgCl2 system
├── structure_H2O_LiCl.py           # Structure model for H2O / LiCl system
├── preparation_rp.py               # Preparation–RP regression model
├── preparation_D.py                # Preparation–D regression model
├── data/
│   ├── structure - H2O MgCl2.xlsx  # See "Data availability" below
│   ├── structure - H2O LiCl.xlsx
│   ├── preparation - rp.xlsx
│   └── preparation - D.xlsx
├── models/                         # Saved XGBoost models (created by train_models.py)
├── results/                        # All output files (created at runtime)
├── requirements.txt
└── README.md
```

The four modeling scripts (`structure_H2O_MgCl2.py`, `structure_H2O_LiCl.py`, `preparation_rp.py`, `preparation_D.py`) are independent and each one trains an XGBoost model, exports predictions, and produces SHAP / PDP analyses for one dataset.

The three GA scripts (`train_models.py` + two `run_ga_target*.py`) together implement the inverse design workflow.

---

## 2. Environment

- Python ≥ 3.9
- Tested on Windows 10 / 11 and Ubuntu 22.04

Install dependencies:

```bash
pip install -r requirements.txt
```

`requirements.txt`:

```
numpy
pandas
scikit-learn
xgboost
bayesian-optimization
shap
rdkit
matplotlib
scipy
openpyxl
joblib
```

---

## 3. Data availability

The experimental datasets used to train the models (`structure - H2O MgCl2.xlsx`, `structure - H2O LiCl.xlsx`, `preparation - rp.xlsx`, `preparation - D.xlsx`) are provided as **Supporting Information** of the published article. Please place the four Excel files in the `data/` directory before running the scripts (or in the project root, matching the paths used in the scripts).

---

## 4. How to reproduce the results

### 4.1 Train the four regression models

Each script can be run independently:

```bash
python structure_H2O_MgCl2.py
python structure_H2O_LiCl.py
python preparation_rp.py
python preparation_D.py
```

Each script:

- loads its dataset,
- runs Bayesian optimization to tune XGBoost hyperparameters,
- trains the final model,
- prints the selected hyperparameters and Train / Test R²,
- exports predictions, Pearson / RMSE values, SHAP plots, and PDP data to `results/`.

The console output is intentionally minimal — only the final selected hyperparameters and model performance are printed; the BO search log is suppressed.

### 4.2 Inverse design via Genetic Algorithm

Step 1 — train and save the RP and SJ models:

```bash
python train_models.py
```

This produces `models/best_rp.pkl` and `models/best_sj.pkl`.

Step 2 — run the GA for each target interval:

```bash
python run_ga_target1.py   
python run_ga_target2.py    
```

Each GA script:

- loads the saved RP and SJ models,
- searches PIP / TMC concentrations on a discrete grid,
- minimizes a weighted interval error `total_err = RP_WEIGHT * err_rp + SJ_WEIGHT * err_sj`,
- exports all visited candidates and the GA convergence history to Excel,
- additionally exports a deduplicated candidate list (sorted by `total_err` ascending, then by `PIP_X1` descending).

### 4.3 Output files

All outputs are written to `results/`:

- `structure_H2O_MgCl2_predictions.xlsx`, `structure_H2O_LiCl_predictions.xlsx`, `preparation_rp_predictions.xlsx`, `preparation_D_predictions.xlsx` — actual vs. predicted values on train and test sets.
- `pdp_<dataset>_<feature>.xlsx` — partial dependence data for the selected features.
- `GA_PIP_TMC_total_err_results_target{1,2}.xlsx` — all GA-visited candidates and the optimization history.
- `GA_unique_prediction_candidates_target{1,2}.xlsx` — deduplicated and ranked candidate recipes.

---

## 5. Reproducibility notes

- All scripts fix random seeds (`random_state` for `train_test_split`, `random_state=0` for Bayesian optimization, `SEED=42` for the GA) so that results are deterministic given the same input data and library versions.
- The hyperparameter search ranges and final selected values reported in the paper are hard-coded in the corresponding scripts.
- Minor numerical differences may occur across XGBoost / scikit-learn versions; the versions used in our experiments are pinned via `requirements.txt`.

