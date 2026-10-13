# HARE: Risk-Aware Residual Estimation for Food Waste Prediction

Code, data, and results accompanying the manuscript *"Enhancing Industrial Catering Reliability Using a Risk-Aware HARE Framework for Food Waste Prediction"*.

**HARE (Heteroscedasticity-Aware Residual Estimator)** is a two-stage forecasting framework for institutional kitchens, where under-preparation (shortfall) costs more than over-preparation (surplus):

1. **Stage 1 – Base model:** a linear regression captures the macro demand trend.
2. **Stage 2 – Asymmetric "ghost" layer:** a gradient-boosted model (LightGBM; XGBoost/CatBoost variants also tested) is trained on the base model's residuals with instance weights that penalise under-prediction by a factor *w*.

Models are compared on **total operational cost** (`w × shortfall mass + surplus mass`), not RMSE alone.

## Key results (held-out test set, n = 177)

| Model | Shortfall (kg) | Surplus (kg) | Total cost (r = 3) |
|---|---|---|---|
| Baseline LM | 272.7 | 420.4 | 1238.6 |
| Random Forest | 317.6 | 502.7 | 1455.3 |
| Safety Buffer (+3.0 kg) | 111.4 | 790.1 | 1124.5 |
| Quantile LGBM (q = 0.85) | 111.4 | 916.5 | 1250.7 |
| **HARE (w = 30)** | **90.5** | 896.8 | 1168.3 |

* HARE has the lowest shortfall mass of all models (66.8 % below the linear baseline).
* Breakeven shortfall:surplus cost ratios: HARE beats Random Forest for r > 1.74, the linear baseline for r > 2.61, and the tuned safety-buffer heuristic for r > 5.09. At r = 3 the safety buffer is slightly cheaper; the advantage of HARE is **conditional on the cost ratio**, and the paper reports this explicitly.
* Ten-fold CV: HARE beats the linear baseline in 10/10 folds (Wilcoxon p = 0.00195; Diebold–Mariano p < 0.001).

## Repository structure

```
hare-food-waste-forecasting/
├── hare_pipeline.ipynb      # full pipeline: preprocessing, HARE, baselines, sweeps,
│                            # cost-ratio sensitivity, significance tests, SHAP, ESG
├── data/
│   └── train.csv            # daily kitchen-operations records (Jan 2022 – Sep 2024)
├── results/
│   ├── final_test_results.csv
│   └── HARE_results.xlsx    # all tables and charts as a workbook
├── figures/                 # Figures 1–9 used in the manuscript
├── requirements.txt
├── CITATION.cff
├── LICENSE                  # MIT
└── README.md
```

## Dataset

`data/train.csv` contains daily records with the columns `ID, date, meals_served, kitchen_staff, temperature_C, humidity_percent, day_of_week, special_event, past_waste_kg, staff_experience, waste_category, food_waste_kg`. The target is `food_waste_kg`. Some fields contain formatting noise and sensor-fault values; the cleaning and outlier-trimming steps are documented in the notebook and in Section 2.3 of the paper.

## Quick start

```bash
git clone https://github.com/<your-username>/hare-food-waste-forecasting.git
cd hare-food-waste-forecasting
pip install -r requirements.txt
cp data/train.csv .          # the notebook reads train.csv from the working directory
jupyter notebook hare_pipeline.ipynb
```

Run all cells top to bottom. Outputs (CSV tables and plots) are written to `python_output/` and `journal_output/`.

## Methodology notes

* **Chronological split** (60 / 20 / 20 train / validation / test); the test set is used once, for final reporting.
* **Outlier bounds** (IQR) are fit on the training partition only.
* **Hyperparameters** (HARE *w*, quantile *q*, safety-buffer *k*) are selected on validation by minimum total cost.
* **Determinism:** boosted models are run single-threaded with `deterministic=True` and a fixed seed (123), so reruns give identical numbers.

## Citation

If you use this work, please cite the article (details to be added on publication) or use the `CITATION.cff` file.

## Authors

* Shalini B, Yuvasri S.M, Anusha R – School of Computer Science Engineering and Information Systems, VIT Vellore
* Dr. Arun Pandian J (corresponding author, arunpandian.j@vit.ac.in) – Centre for AI Research, VIT Vellore

## License

Released under the [MIT License](LICENSE).
