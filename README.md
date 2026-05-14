# Flight-LTO-Emission

# ✈️ Aircraft LTO Emissions Cycle Analysis
### CE 160D · Spring 2026 · UC Berkeley

> A data-driven evaluation of aircraft Landing and Take-Off (LTO) emissions and their direct impact on fenceline communities near major U.S. airports.


---

## 📌 Overview

This project analyzes **7.7 million domestic flights across 40 U.S. airports in 2025** to quantify LTO-cycle emissions (CO₂ and NOx), identify airport emission profiles, and build predictive models using pre-flight information alone.

LTO-cycle emissions — generated during approach, taxi, takeoff, and climb-out phases below 3,000 ft — are localized near airports and have documented health impacts on surrounding communities, including elevated risks of cardiovascular disease and respiratory distress from NOx exposure.

---

## 📂 Repository Structure

```
ce160d-lto-emissions/
├── README.md
├── report/
│   └── CE160D_Final_Report.pdf
├── notebooks/
│   ├── 01_Data_collecting.ipynb
│   ├── 02_EDA.ipynb
│   ├── 03_clustering_pca.ipynb
│   └── 04_Regression_modeling.ipynb
├── images/
│   └── (charts and figures)
├── data/
│   └── README.md              ← download instructions
└── .gitignore
```

---

## 🗂️ Data Sources

| Source | Description |
|--------|-------------|
| [Bureau of Transportation Statistics (BTS)](https://www.transtats.bts.gov/) | 2025 Airline On-Time Performance — 7.7M domestic flights |
| [ICAO Engine Emissions Databank (via EASA)](https://www.easa.europa.eu/en/domains/environment/icao-aircraft-engine-emissions-databank) | Engine-specific fuel flow and emissions indices (turbojet/turbofan, >26.7 kN) |
| [FAA Civil Aircraft Registry](https://www.faa.gov/licenses_certificates/aircraft_certification/aircraft_registry/releasable_aircraft_download) | Tail number → aircraft type → engine configuration mapping |
| [Open-Meteo Historical Weather API](https://open-meteo.com) | Hourly temperature and pressure for fuel flow correction |

**Final merged dataset:** 1,532,364 flights · 40 airports · 21 carriers · 97 aircraft types

> ⚠️ Raw data files are not included in this repo due to size. See [`data/README.md`](data/README.md) for download instructions.

---

## ⚙️ Methodology

### Emissions Calculation
LTO emissions per flight are calculated using Boeing's Fuel Flow Method 2, correcting for ambient temperature and pressure at each airport's elevation:

$$FC = \sum_{i \in F} \left(\frac{T}{518.67}\right)^{3.5} \cdot \frac{\psi}{14.696} \cdot FF_i \cdot t_i \cdot n$$

$$E_{F,P} = FC \cdot EI$$

Where `T` = Rankine temperature, `ψ` = pressure (psi), `FF` = fuel flow, `t` = time-in-mode, `n` = number of engines.

### Airport Clustering (Unsupervised)
- Features aggregated to airport level (mean emissions, fleet composition, taxi time, route distance, weather)
- Standardized with `StandardScaler` → reduced to 2 PCs via **PCA**
- **K-Means clustering** (k=3) applied separately for CO₂ and NOx profiles

### Predictive Modeling (Supervised)
- **Target variables:** Total CO₂ (ton/flight), Total NOx (kg/flight)
- **Input features:** Route distance, origin/destination, airline, month, route type
- **Models evaluated:** Linear Regression, Decision Tree, XGBoost, LightGBM
- **Feature importance:** SHAP values on LightGBM

---

## 📊 Key Results

### Model Performance

| Model | CO₂ R² | CO₂ MAPE | NOx R² | NOx MAPE |
|-------|--------|----------|--------|----------|
| Linear Regression | 0.149 | 26.98% | 0.231 | 42.50% |
| Decision Tree | 0.484 | 17.42% | 0.626 | 19.77% |
| XGBoost | 0.547 | 15.86% | 0.682 | 17.78% |
| **LightGBM** | **0.563** | **15.31%** | **0.698** | **16.70%** |

**Best model:** LightGBM — R² 0.56 (CO₂), 0.70 (NOx)

### SHAP Feature Importance
Top predictors for both CO₂ and NOx: **Airline > Distance > Month**

Airline identity serves as a strong proxy for fleet type and route network. Distance directly drives total fuel burn. Month captures seasonal temperature and pressure variation affecting engine efficiency.

### Airport Clusters (CO₂)

| Cluster | Airports | Avg CO₂/flight | CO₂ Intensity |
|---------|----------|----------------|---------------|
| 0 — Small Regional (e.g., AVL, BGR) | 86.2% short-haul | 16.91 ton | 0.0469 ton/km |
| 1 — Major Hubs (e.g., JFK, SFO) | Long-haul dominant | **25.7 ton** | **0.0284 ton/km** ✅ |
| 2 — Pacific/Alaska/Hawaii (e.g., OGG, OAK) | 84.9% Boeing | 21.41 ton | 0.0706 ton/km ❌ |

Hawaii airports (KOA, OGG) disproportionately drive Cluster 2's high intensity — Boeing 717-200 accounts for ~34% of flights on ultra-short inter-island routes.

### Notable Findings
- **NOx** is primarily generated during **Take-Off and Climb-Out** (high thrust phases)
- **CO₂** shows higher contributions from **ground operations** (Taxi-Out, Taxi-In)
- Major hub airports average **21.2 min taxi-out** vs. **12.2 min** at remote airports
- Modern high-efficiency engines (LEAP series) show a **NOx–fuel efficiency trade-off**: lower fuel flow but higher NOx emission index
- Older aircraft (Boeing 717-200) show significantly higher CO₂ intensity vs. newer narrowbodies (A321neo)

---

## 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)
![Pandas](https://img.shields.io/badge/Pandas-2.x-150458?logo=pandas)
![Scikit-learn](https://img.shields.io/badge/scikit--learn-PCA%20%7C%20KMeans-F7931E?logo=scikit-learn)
![LightGBM](https://img.shields.io/badge/LightGBM-best%20model-brightgreen)
![XGBoost](https://img.shields.io/badge/XGBoost-0.547%20R²-orange)
![SHAP](https://img.shields.io/badge/SHAP-feature%20importance-blueviolet)

```
pandas · numpy · scikit-learn · lightgbm · xgboost · shap · matplotlib · seaborn
```

---

## 🔭 Future Work

- Include **gate emissions** (currently omitted from LTO cycle calculation)
- Add **seat capacity normalization** for per-passenger emissions comparison
- Incorporate **wind speed, visibility, fuel type, and peak travel periods**
- Investigate whether airlines systematically shift high-emitting aircraft to smaller regional hubs to mask emissions mandates

---

## 📄 Report

Full report available here: [`report/CE160D_Final_Report.pdf`](report/CE160D_Final_Report.pdf)

---

## 📚 References

- FAA (2016). *Aviation Environmental Design Tool (AEDT) Technical Manual*
- Zou et al. (2025). "A pathway to sustainable aviation." *Energy, 319*, 135074.
- ATSDR (2024). *ToxFAQs for Nitrogen Oxides*
