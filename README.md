# Geopolitical Shocks and Gas-Price Propagation

**MSc Dissertation — Energy Systems and Data Analytics (UCL, BENV0096)**
Candidate RXZK4

Reproducible analysis code for the dissertation *"How Geopolitical Shocks Propagate into Gas Prices."* The project examines how geopolitical shocks reprice a globally integrated gas market through arbitrage structure, and how far the resulting price movements can be forecast.

---

## Overview

The analysis addresses two research questions:

- **Q1 (explanatory):** How do geopolitical shocks reprice the gas market, and through what mechanism? Answered with cointegration, error-correction modelling, structural-break tests, GARCH volatility, and Diebold–Yilmaz connectedness.
- **Q2 (predictive):** Can machine-learning models forecast the resulting movements, and how does skill vary by horizon and market regime? Answered with a walk-forward benchmark across model families against a random-walk and ARIMA baseline.

Three geopolitical events anchor the study: the 2022 invasion of Ukraine, the 2023–24 Red Sea shipping disruption, and the 2026 closure of the Strait of Hormuz.

---

## Repository contents

| File | Description |
|------|-------------|
| `LNG_Dissertation_Analysis_RXZK4.ipynb` | Main analysis notebook (all figures, tables and statistical results) |
| `data/` | Input data (see **Data** below) |
| `output/` | Generated figures (`.png`/`.pdf`), result tables (`.csv`), and the assembled `master_panel.parquet` |
| `README.md` | This file |

The notebook is organised into clearly headed sections matching the dissertation chapters:

1. **Setup** — environment, paths, data load (cells A1–A3)
2. **EDA and stationarity diagnostics** (ADF/KPSS)
3. **Q1 mechanism** — cointegration, error-correction, arbitrage threshold, structural breaks (Chow), GARCH(1,1), connectedness
4. **Q2 forecasting** — walk-forward engine, feature engineering, model sweeps, final evaluation
5. **Additional analysis** — Granger causality, feature ablation, SHAP, bootstrap confidence intervals, multi-horizon and regime splits, cross-hub robustness

---

## Requirements

- **Python 3.10+** (developed on Google Colab)
- Key libraries:

```
numpy, pandas, matplotlib, seaborn, scikit-learn,
statsmodels, arch, ruptures, shap, xgboost,
torch, pytorch-forecasting, lightning, requests
```

Install everything with:

```bash
pip install numpy pandas matplotlib seaborn scikit-learn statsmodels \
    arch ruptures shap xgboost torch pytorch-forecasting lightning requests
```

`cartopy` is optional (used only for the chokepoint map figure); install separately if needed.

---

## Data

The analysis draws on daily data over **January 2016 to mid-2026**:

- **Gas hub prices** (front-month futures): TTF (Europe), JKM (Asia), NBP (UK), THE (Germany), Henry Hub (US)
- **Brent and WTI crude, EU carbon (EUA)**
- **Maritime chokepoint transits** (Suez, Bab el-Mandeb, Hormuz, Cape of Good Hope; 7-day moving averages)
- **EU gas storage** fundamentals
- **Geopolitical Risk (GPR) index** (Caldara and Iacoviello, 2022) and its acts/threats sub-indices
- **EIA weekly release calendar** (gas storage and oil inventory report dates), pulled via the EIA API v2

Price and commodity series are assembled into a single `master_panel.parquet`. Because commercial price data (e.g. TTF/JKM futures) may be licensed, raw price files are **not redistributed** in this repository. The notebook loads the pre-assembled panel from `output/master_panel.parquet`; re-running the full pipeline requires the user to supply their own price sources.

### Directory layout expected by the notebook

The notebook was developed on Google Colab with Google Drive mounted. Paths are set at the top of the setup cell:

```python
BASE     = "/content/drive/MyDrive/Dissertation Analysis"
DATA_DIR = f"{BASE}/data"
OUT_DIR  = f"{BASE}/output"
```

**To run locally,** change `BASE` to your local project directory.

---

## Reproducing the results

1. Clone the repository and install the requirements above.
2. Set `BASE` (top of the notebook) to your project path.
3. Place `master_panel.parquet` in the `output/` folder (or supply raw price data and re-run the assembly step).
4. Run the notebook top to bottom. Figures and result tables are written to `output/`.

The forecasting evaluation uses **expanding-window walk-forward validation** with strict leakage controls: every feature is constructed from data dated *t−1* or earlier, imputation is forward-only and capped at 5 trading days, and feature scaling is fitted inside each training fold. A dedicated leakage-audit cell (`Task G`) verifies that no scaling, tuning, or target-construction step uses future information.

---

## Key methods and models

**Q1 (econometrics):** Engle–Granger and Johansen cointegration, error-correction model with regime interactions, a continuous arbitrage-threshold specification, Chow structural-break tests, GARCH(1,1) volatility, and Diebold–Yilmaz connectedness.

**Q2 (forecasting), benchmarked families:**

- Linear / regularised: **LinearSVR** (best performer), **ElasticNet**
- Tree ensembles: **XGBoost**, **Random Forest**
- Neural: **MLP**, feed-forward **DNN**, **LSTM**, **TCN**
- Baselines: **Random Walk** (primary), **ARIMA**

Significance is assessed with the **Diebold–Mariano** test; interpretability via **SHAP** and group ablation; uncertainty via bootstrap confidence intervals.

---

## Outputs

Running the notebook regenerates all dissertation figures and tables, including:

- Figures: hub-price series with event windows, chokepoint transits, storage, coverage, arbitrage-threshold plot, model comparison, hyperparameter sweeps, walk-forward scheme
- Tables (CSV): descriptive statistics, stationarity diagnostics, cointegration and ECM results, threshold bins, Chow F-statistics, GARCH parameters, connectedness matrix, walk-forward metrics by model and hub, ablation, SHAP, and generalisation gaps

---

## License

This code is provided for academic and review purposes. See `LICENSE` if included; otherwise all rights reserved by the author and school.
