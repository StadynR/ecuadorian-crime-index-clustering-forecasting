# Canton-Level Crime Risk Index and National Forecasting - Ecuador

This repository contains the full analytical pipeline for a master's thesis research project on crime risk in Ecuador. It covers two stages: an **exploratory data analysis (EDA)** that constructs a canton-level composite risk index, and a **forecasting** study that projects the national crime risk index up to 2025 using multiple time-series models.

---

## Research Questions

1. How consistent is a shrinkage-adjusted, rate-based composite index compared to raw counts and unadjusted rates at the canton level?
2. What is the optimal structural segmentation of cantons by risk profile?
3. Which forecasting model best predicts the national monthly crime risk index on a strict temporal hold-out?

---

## Repository Structure

```
.
├── eda.ipynb                          # Canton-level risk index construction
├── forecasting.ipynb                  # National index forecasting (2024–2025)
├── data/
|   ├── armas ilicitas 2017-2026.xlsx      # Canton-level data of seized weapons (Ministerio del Interior)
|   ├── detenidos 2019 - 2026.xlsx         # Canton-level data of arrests (Ministerio del Interior)
|   ├── homicidios 2014-2026.xlsx          # Canton-level data of homicides (Ministerio del Interior)
│   └── poblacion_cantones_2019-2025.csv   # Canton-level population estimates (INEC)
├── requirements.txt
└── README.md
```

---

## Notebooks

### `eda.ipynb` — Canton-Level Risk Index

Builds and validates a composite crime risk index for all Ecuadorian cantons.

| Section | Contents |
|---|---|
| 1. Data Sources | Loading and quality audit of all sources |
| 2. Cleaning | Key normalization, date extraction, monthly merge with population |
| 3. Index | Rates per 10 000 inh. → Bayesian shrinkage → PCA weights → z-score (`indice_riesgo`) |
| 4. KMeans | Multi-metric evaluation (Elbow, Silhouette, CH, DB), k selection (k = 6), cluster assignment |
| 5. Visualizations | Distributions, correlations, temporal evolution, canton profiles |
| 6. Robustness | Comparison: shrunk rates vs. unadjusted rates vs. absolute counts |

**Key methodological choices:**
- Bayesian shrinkage for small-population cantons: $\hat{r}_i = w_i \cdot r_i + (1 - w_i) \cdot \bar{r}$, with $w_i = \frac{n_i}{n_i + m}$ and $m$ equal to the median canton population.
- PCA weights derived from PC1 loadings on shrinkage-adjusted rates.
- Composite index as a weighted z-score; no fixed threshold bands.
- Canton identifier uses the compound key `provincia_key | canton_key` to handle homonymous cantons (e.g., *Olmedo*).

### `forecasting.ipynb` — National Index Forecasting

Forecasts the monthly national crime risk index for 2024–2025 (trained on 2019–2023).

**Fixed temporal split:** train `2019-01 → 2023-12`, test `2024-01 → 2025-12`.  
No test-period data leaks into PCA weights, normalization parameters, or early stopping.

| Model | Type | Library |
|---|---|---|
| **Ensemble** | Weighted average of top models | — |
| LSTM | Recurrent neural network | TensorFlow / Keras |
| XGBoost | Gradient boosting with lag features | xgboost |
| ETS | Holt-Winters seasonal exponential smoothing | statsmodels |
| Prophet | Additive trend + seasonality | prophet |
| ARIMA | Classical linear autoregressive model | statsmodels / pmdarima |

**Results (test period 2024–2025):**

| Model | RMSE | MAE |
|---|---|---|
| **Ensemble** | **0.0871** | **0.0679** |
| LSTM | 0.1169 | 0.0963 |
| XGBoost | 0.1340 | 0.1102 |
| ETS | 0.1361 | 0.1221 |
| Prophet | 0.1501 | 0.1362 |
| ARIMA | 0.1609 | 0.1486 |

---

## Setup and Reproduction

### 1. Clone the repository

```bash
git clone https://github.com/StadynR/ecuadorian-crime-index-clustering-forecasting
cd ecuadorian-crime-index-clustering-forecasting
```

### 2. Create a virtual environment and install dependencies

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

### 3. Run the notebooks in order

```
1. eda.ipynb
2. forecasting.ipynb
```

Run all cells top to bottom within each notebook. The notebooks are self-contained and reproducible given the input data.

---

## Requirements

See [`requirements.txt`](requirements.txt). Main dependencies:

- Python ≥ 3.10
- pandas, numpy, matplotlib, seaborn
- scikit-learn (PCA, KMeans, scalers)
- statsmodels, pmdarima (ETS, ARIMA, STL, stationarity tests)
- prophet (Facebook/Meta Prophet)
- xgboost
- TensorFlow ≥ 2.13 (LSTM)

---

## Citation

If you use this code or methodology in your work, please cite:

```
Román-Niemes, S.; Morocho-Cayamcela, M. E.. (2026). Measuring Territorial Risk in Ecuador: An Integrated Framework for Crime Indexing, Clustering, and Forecasting. Yachay Tech University.
```

---

## License

This project is released for academic and research use. The analytical code is available under the [MIT License](LICENSE).
