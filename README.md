## 🧾 Attribution (Use of AI Citation)  

> This README was generated with assistance from an **AI system (GPT-5 Thinking)** and subsequently reviewed/edited by the author, who is responsible for the final content.  

# Explainable-Techniques-II
Explored feature correlations and applied PDP, ICE, and ALE to interpret model predictions

# Explainable Techniques II — PDP, ICE, and ALE on Beijing PM2.5

This repo contains a reproducible notebook that trains a tree-based model to predict **PM2.5 (µg/m³)** from meteorology and time features using the **PRSA (Beijing) hourly air-quality dataset, 2010–2014**. It then explains the model with **Partial Dependence Plots (PDP)**, **Individual Conditional Expectation (ICE)**, and **Accumulated Local Effects (ALE)**, and discusses how **feature correlation** impacts these techniques.

---

## Highlights

- **Task:** Regression — predict PM2.5 from `TEMP, DEWP, PRES, Iws, Is, Ir` + time signals (`hour, month, dow, is_weekend`) and categoricals (`cbwd` wind direction, `season`).
- **Model:** `RandomForestRegressor` (`n_estimators=600`, `min_samples_leaf=2`) in a `sklearn` **Pipeline** with one-hot encoding for categoricals.
- **Performance (test set):** **R² = 0.690**, **RMSE = 51.94 µg/m³**.
- **Top drivers (impurity importance):** `DEWP`, `Iws`, `season_winter`, `PRES`, `TEMP`, then `month`, `hour`, `dow`.
- **Correlation snapshot (Pearson):** `TEMP–PRES ≈ -0.83`, `TEMP–DEWP ≈ +0.82`, `DEWP–PRES ≈ -0.78`.  
  **VIFs** (numerics): `TEMP ≈ 4.9`, `DEWP ≈ 4.3`, `PRES ≈ 3.7` (moderate multicollinearity).
- **PDP/ICE takeaways:**
  - `DEWP` ↑ → PM2.5 ↑ (strong, monotone).
  - `Iws` ↑ → PM2.5 ↓ (dispersion) with hints of plateau.
  - `PRES` shows a mild hump near ~1010–1012 hPa then declines.
  - `TEMP` ↑ → PM2.5 ↓ (decline).
- **ALE takeaways (on-manifold):** Confirms `DEWP` positive and `Iws` negative with **saturation** at high wind; `TEMP` negative then **plateaus**; `PRES` peaks locally then declines.  
  **2D effects (`TEMP × DEWP`):** worst regime is **cold + humid** (winter haze signature).

---

## Why PDP vs. ALE?

- **PDP** averages predictions over the **marginal** feature distribution and can evaluate the model at **unrealistic combinations** when predictors are correlated (off-manifold bias).
- **ALE** aggregates **local differences** only where data exist, making it **more reliable under correlation** (e.g., the TEMP–DEWP–PRES block here).

---

## 🗂 Repository Structure
```
Interpretable_ML_Assignment2/
│── Assignment4_Explainable_Techniques_2.ipynb   
│── README.md                            
```
### 🚀 Getting Started

#### ✅ Requirements
- **Python:** 3.9+
- **Packages:** `numpy`, `pandas`, `scikit-learn`, `matplotlib`, `seaborn`, `statsmodels`, `PyALE`, `jupyter` *(or use Google Colab)*

Install everything with:
```bash
pip install numpy pandas scikit-learn matplotlib seaborn statsmodels PyALE jupyter
```

### ✅ Results — Short Narrative

- **Moisture dominates:** Higher **dew point** strongly raises predicted **PM2.5**; **ALE** shows the effect **accelerates above ~0–10 °C**.
- **Wind disperses then saturates:** Increasing **Iws** reduces PM2.5, with **diminishing returns** at high values.
- **Pressure regime matters:** A **local peak near ~1010–1012 hPa** indicates stable episodes that **elevate PM2.5**; beyond that, **higher pressure trends cleaner**.
- **Temperature reduces risk (to a point):** **Warming lowers** predictions until the effect **plateaus**.
- **Methodological insight:** With **correlated meteorology**, **ALE** provides **smoother, on-manifold local effects** than **PDP** (e.g., removes PDP’s small **off-support artifact** around wind).
