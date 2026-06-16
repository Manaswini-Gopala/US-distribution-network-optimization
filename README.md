# 🏭 Cost-Optimized U.S. Distribution Network Design
### Demand Forecasting + Constrained Center-of-Gravity Optimization | MIS 753: Global Supply Chain Analytics

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1HHc6JCJ_KUJNsNJMjXq4jnTSAzwjN5Yh#scrollTo=-C7k_CJ0LyD0)
![Python](https://img.shields.io/badge/Python-3.10-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-KMeans-orange)
![statsmodels](https://img.shields.io/badge/statsmodels-ETS-green)
![Folium](https://img.shields.io/badge/Folium-Map-red)

---

## Overview

This project designs a **cost-optimal U.S. distribution network** by combining time-series demand forecasting with a constrained, multi-depot Center of Gravity (CoG) optimization model — built entirely in Python.

Using four years of transactional data (2014–2017) from a retail Superstore dataset, the analysis identifies the optimal number and placement of distribution depots, enforces a **corporate Furniture routing constraint** (all Furniture flows must route through depots nearest Los Angeles or Houston), and validates the final network design through elbow analysis, sensitivity decomposition, and robustness testing.

**Bottom line:** A 5-depot network minimizes total system cost at approximately **$123.5M**, reducing average shipping distances by ~20% versus a 3-depot baseline while remaining fully constraint-compliant.

---

## Problem Statement

A U.S. retailer needs to plan its distribution network for a future planning horizon. Leadership has imposed a strategic constraint: **all Furniture category shipments must be handled exclusively through depots located nearest to Los Angeles, CA or Houston, TX** (reflecting last-mile logistics agreements and regional warehouse capabilities).

The goal is to determine:
1. **How many depots (K)** minimizes total system cost (transportation + fixed)?
2. **Where** should those depots be located?
3. Does the Furniture constraint materially change network structure?

---

## Methodology

```
Raw Data (2014–2017)
       │
       ▼
Data Cleaning & Feature Engineering
  • Missing value imputation (subcategory median)
  • Excel serial date conversion
  • Duplicate resolution by business key (Order ID × Product ID × Order Date)
       │
       ▼
Demand Aggregation
  • Monthly time series per subcategory
  • Spatial demand matrix (City × Category) for CoG
       │
       ├──────────────────────────────────────────┐
       ▼                                          ▼
Time-Series Forecasting (Top 5)         Historical CoG Matrix
  • Train: 2014–2016                     (2014–2017 actuals)
  • Validate: 2017
  • Models: Holt-Winters ETS,
    Seasonal Naïve (benchmark)
  • Metrics: MAPE, MAE, RMSE
       │
       ▼
2018 Forecast Demand Matrix
  (Top 5 subcategories × location shares)
       │
       ├──────────────────────────────────────────┘
       ▼
Multi-Depot CoG Optimization (K = 1 → 10)
  • Weighted K-Means on (lat, lon) with demand weights
  • Haversine distance for transport cost
  • Fixed depot cost: $8M per depot
  • Furniture constraint: only LA/Houston-nearest depots eligible
       │
       ▼
Network Evaluation
  • Elbow chart (total cost vs K)
  • Sensitivity decomposition (transport vs fixed)
  • Robustness test (K = 4, 5, 6)
  • Demand balance + fairness analysis
       │
       ▼
Final Design: K = 5 Depots → Interactive Folium Network Map
```

---

## Key Results

### Forecast Performance

| Subcategory | Model Selected | MAPE (%) | Naive Baseline MAPE (%) | Improvement |
|-------------|---------------|----------|--------------------------|-------------|
| Phones      | Holt-Winters  | 22.62    | 29.23                    | +11.86 pp   |
| Chairs      | Seasonal Naïve | 25.69   | 23.97                    | —           |
| Storage     | Holt-Winters  | 22.89    | 21.61                    | +9.56 pp    |
| Binders     | Holt-Winters  | 41.82    | 46.62                    | +5.88 pp    |
| Machines    | Holt-Winters  | 43.68    | 67.34                    | +22.50 pp   |

### Optimal Depot Count (K = 5)

| K | Transportation Cost | Fixed Depot Cost | **Total Cost** |
|---|--------------------:|----------------:|---------------:|
| 4 | $94.34M             | $32.00M         | $126.34M       |
| **5** | **$83.53M**     | **$40.00M**     | **$123.53M ✓** |
| 6 | $76.12M             | $48.00M         | $124.12M       |

- **K = 5 saves $3.03M (+2.27%) vs K = 4** and **$0.59M (+0.48%) vs K = 6**
- Average shipping distance reduced by ~20% vs K = 3

### Network Characteristics at K = 5

| Metric | Value |
|--------|-------|
| Total System Cost | ~$123.5M |
| Furniture depots | Los Angeles + Houston (constraint enforced, 0 violations) |
| Demand balance | 4 depots at 21–26% share; 1 niche depot ~5% |
| Max distance range | 721 km – 2,776 km (Depot 2 high due to Furniture constraint, not inefficiency) |
| Robustness | K±1 raises cost; design is stable |

---

## Visualizations

All charts are compiled in [`outputs/charts.pdf`](outputs/charts.pdf).

| Chart | What it shows |
|-------|--------------|
| Elbow Chart | Total cost vs K — steep drop K=1→3, U-shape minimum at K=5 |
| Sensitivity Chart | Transportation vs fixed depot cost tradeoff across K |
| Robustness Chart | K=4/5/6 cost comparison confirming K=5 stability |
| Demand Balance | Depot-level demand share — four depots at 21–26%, one niche at ~5% |
| Fairness Chart | Max shipping distance per depot — Depot 2 elevated by Furniture policy |
| Network Map | Interactive Folium map with flow lines, depot markers, cluster assignments |

---

## Tech Stack

| Layer | Tools |
|-------|-------|
| Data Wrangling | `pandas`, `numpy` |
| Forecasting | `statsmodels` (ETSModel / Holt-Winters), Seasonal Naïve |
| Optimization | `scikit-learn` (KMeans), custom Haversine CoG solver |
| Visualization | `matplotlib`, `folium` (interactive map) |
| Environment | Google Colab |

---

## Project Structure

```
├── distribution_network_optimization.ipynb   # Full analysis notebook
├── outputs/
│   └── charts.pdf                            # All visualization charts
└── README.md
```

> **Note:** The raw dataset is not tracked in this repo. To run the notebook, upload the Superstore dataset to your Colab environment and update `csv_path` in the first cell. The full interactive notebook is available via the **Open in Colab** badge above.

---

## How to Run

1. Click **Open in Colab** at the top of this README
2. Upload the Superstore dataset to `/content/` and update `csv_path` in Cell 1
3. Run all cells top-to-bottom — the notebook is a sequential pipeline:
   - **Cells 1–4:** Data loading, cleaning, deduplication
   - **Cells 5–7:** Monthly aggregation and top-5 subcategory selection
   - **Cells 8–12:** Forecasting (Holt-Winters ETS + Seasonal Naïve benchmarks)
   - **Cells 13–15:** Spatial demand matrix construction
   - **Cells 16–17:** Multi-depot CoG optimization (K = 1–10) with Furniture constraint
   - **Cells 18–20:** Elbow and sensitivity charts
   - **Cell 21:** Interactive Folium network map (K = 5)
   - **Cells 22–24:** Extra insights — demand balance, fairness, robustness

---

## Design Decisions & Limitations

**Why weighted K-Means for CoG?**
Standard CoG (weighted centroid) is analytically clean but limited to a single depot. Weighted K-Means extends this to multi-depot scenarios while preserving demand-weighted centroid logic — computationally tractable and interpretable for a planning horizon.

**Why Haversine, not Euclidean?**
U.S. geographic extents are large enough that straight-line distance introduces meaningful error on coastal-to-inland routes. Haversine great-circle distance is a better proxy for real freight distances at this scale.

**Furniture constraint implementation:**
At each K, the model identifies the depot nearest to LA and Houston by Haversine distance. Furniture demand points are then restricted to only those two depots during assignment — enforcing the corporate rule without manual override.

**Known limitations:**
- Transportation cost is linear ($1/km/unit); real freight rates are tiered and mode-dependent
- CoG does not model inventory holding cost, labor, or service time windows
- Forecast uncertainty (MAPE 22–44%) is not propagated into CoG — a stochastic extension would strengthen the recommendation
- The model assumes depot placement is unconstrained beyond the Furniture rule

---

## Possible Extensions

- **MILP formulation** for exact facility location with integer depot count constraints
- **Stochastic demand** — Monte Carlo sampling over forecast confidence intervals to stress-test K = 5
- **Multimodal cost modeling** — differentiate road, rail, and air freight rates by lane
- **Service-level constraints** — cap maximum distance or transit time per cluster
- **Real-time reoptimization** — connect to live sales data for rolling network design

---

*Built with Python · scikit-learn · statsmodels · Folium*
