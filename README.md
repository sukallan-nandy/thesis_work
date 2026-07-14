# Settlement Prediction of Braced Excavations — ML Pipeline

Physics-informed machine learning pipeline for predicting maximum ground surface settlement (δv,max) and maximum lateral wall deflection (δh,max) in braced deep excavations in Dhaka soils, using an 80-run PLAXIS 2D finite element dataset.

## Contents

| File | Description |
|---|---|
| `Settlement_ML_Pipeline.ipynb` | Jupyter notebook containing the full analysis pipeline (source of truth — editable and re-runnable). |
| `Settlement_ML_Pipeline.html` | Static HTML export of the executed notebook, with all figures and outputs rendered, for viewing without a Python environment. |

## What the notebook does

1. **Data loading** — reads the `All Runs (80)` sheet of the PLAXIS parametric study workbook and cleans/renames columns (soil stiffness, friction angle, layer thickness, water table depth, and the two output displacements).
2. **Exploratory data analysis** — correlation heatmap, feature-vs-target scatter plots, distribution plots, and standardized boxplots of the input parameters.
3. **Model definitions** — three regression pipelines, each with scaling fit only inside cross-validation folds to avoid leakage:
   - Linear Regression
   - Polynomial (degree 2) + Ridge, with an inner grid search over the Ridge penalty
   - XGBoost (falls back to scikit-learn's Gradient Boosting if `xgboost` isn't installed)
4. **Model evaluation** — Repeated K-Fold cross-validation (5 folds × 10 repeats) as the headline metric, reporting R², RMSE, MAE, and a train-vs-test R² gap to flag overfitting on the small (n = 80) dataset.
5. **Model comparison table** — saved as `model_comparison.csv`.
6. **Parity and residual plots** — out-of-fold predictions for each model (each point predicted while held out of training).
7. **Feature importance** — permutation importance for the boosted model and standardized coefficients for the linear model.
8. **Sensitivity analysis** — one input swept across its observed range at a time (others fixed at the dataset median), using the fitted Poly-2 + Ridge model.
9. **Governing equation extraction** — the full 21-term standardized polynomial equation, plus a recommended **reduced 10-term equation in raw engineering units** (dominant linear, squared, and interaction terms) that generalizes better than the full polynomial at this sample size.

## Inputs and outputs

**Input features:** soft/fill clay stiffness (E50), friction angle (φ), Madhupur clay stiffness (E50), water table depth, and soft clay layer thickness.

**Targets:**
- δv,max — maximum ground surface settlement
- δh,max — maximum lateral wall deflection

## Requirements

```bash
pip install numpy pandas matplotlib seaborn scikit-learn xgboost openpyxl
```

`xgboost` is optional; the notebook automatically substitutes `GradientBoostingRegressor` if it isn't available.

## Usage

1. Open `Settlement_ML_Pipeline.ipynb` in Jupyter.
2. Update `DATA_DIR` and `FNAME` in the "Paths & output folder" cell to point to your local copy of `PLAXIS_Complete_80runs.xlsx`.
3. Run all cells. Figures (300 dpi) and the model comparison table are written to an auto-created `figures/` folder next to the data file.

## Notes on methodology

Given the small sample size (n = 80), the pipeline is deliberately built to avoid overfitting: cross-validated (not single-split) headline metrics, in-fold-only scaling, cross-validated Ridge regularization, shallow boosted trees, and no additional engineered features. The reduced 10-term polynomial equation is the recommended model for reporting, as it achieves higher cross-validated R² than the full 21-term polynomial while being far more interpretable.
