# ✈️ Amazing Airlines Customer Analysis

**Customer segmentation and loyalty program redesign for Amazing International Airlines (AIAI)**

Data Mining project · Master in Data Science and Advanced Analytics · NOVA Information Management School (NOVA IMS) · 2025–2026
**Group 29:** Moritz Baldauf · Libero Biagi · Jenny Cubelo · Marcell Hajdú

---

## 📌 Overview

AIAI's current three-tier loyalty program (Aurora, Nova, Star) does not reflect any real difference in customer behaviour. Our exploratory analysis showed that median flights, travel distance, income and lifetime value are nearly identical across tiers.

This project builds a **data-driven, multi-perspective customer segmentation** and uses it to propose a new loyalty program.

The work is documented in two reports:
- **Report 1** ([`docs/DM_report 2.pdf`](docs/DM_report%202.pdf)): data cleaning, feature engineering, external data integration and EDA.
- **Report 2** ([`docs/DM_report.pdf`](docs/DM_report.pdf)): clustering, fuzzy validation, final segments and loyalty strategy.

**Key findings from the EDA (Report 1)**
- Loyalty tiers show no behavioural differentiation.
- Program membership grew steadily, but Customer Lifetime Value (CLV) stagnated.
- Three hub cities (Toronto, Vancouver, Montreal) account for ~47% of flights.

**Result (Report 2):** 4 customer segments and a 3-tier loyalty program (*Current < Crest < Deep Blue*).

## 🗂️ Data

| Dataset | Description |
|---|---|
| `CustomerDB` | 16,921 customers, 20 columns (demographics, income, loyalty status, CLV, enrolment/cancellation) |
| `FlightsDB` | 608,436 rows, monthly flight activity per customer (Jan 2019 – Dec 2021): flights, companions, distance, points earned/redeemed |

The two tables are joined on `Loyalty#`. The cleaned and feature-engineered table is provided in [`customer_data`](customer_data). Column descriptions are in `Data/DM_AIAI_Metadata.csv`.

**Cleaning:** duplicate IDs removed (327), 2019 flight counts normalised (multiples of 0.9), impossible distances (> 13,230 km average) and negative subscription durations dropped.

**External data**
- Income for college-educated customers imputed from [Statistics Canada – 2016 Census](https://www12.statcan.gc.ca/census-recensement/2016/as-sa/98-200-x/2016024/98-200-x2016024-eng.cfm) (median earnings by province, education, sex).
- Rural postal codes geocoded with [geocoder.ca](https://geocoder.ca/) to compute distance from the airport.

**Engineered features:** subscription duration, flights in subscription, companion ratio, mean spend per flight, points redemption rate (PRR), average flight distance, distance from airport, **Recency, Frequency, Monetary value (RFM)**.

## 🔬 Methodology

```
Cleaning → Feature engineering → Outlier removal (DBSCAN) → Scaling
        → Behavioural clustering ┐
        → Value-based clustering ┴→ Combined (6) → Hierarchical merge (4)
        → Fuzzy C-Means validation → Loyalty program design
```

| Perspective | Features | Algorithm | Clusters | R² | Silhouette |
|---|---|---|---|---|---|
| Behavioural | total flights, frequency, mean spent, recency, avg. distance, companion ratio | DBSCAN (eps 0.65, min_samples 12) | 3 | 17.2% | 0.33 |
| Value-based | CLV, mean spent, PRR | K-Means++ (15 inits) | 2 | 38.8% | 0.52 |
| Combined | all of the above | Hierarchical (Ward) | 4 | 0.21 | 0.23 |

K-Means, DBSCAN and Self-Organizing Maps were compared for each perspective. DBSCAN was chosen for the behavioural view because it was the only method capturing a visible pattern (two groups with different flights/frequency relationships).

**Fuzzy C-Means** (via `scikit-fuzzy`) confirmed 2 clusters per perspective (FPC / Partition Entropy). About **34%** of customers have mixed behavioural membership versus **21%** for value-based, so value-based labels are more reliable for targeting.

## 👥 Final segments & loyalty program

Full description in Report 2 ([`docs/DM_report.pdf`](docs/DM_report.pdf)).

| Segment | Profile | Loyalty program |
|---|---|---|
| Regular Subscribers ("Mom & Pop Fliers") | loyal, low spending | **Current** |
| Loyal, Long-Time Spenders ("Die-Hard Fans") | long-time customers, high CLV | **Current** (in part) / **Deep Blue** |
| Enthusiastic Spenders ("New Business Bros") | high spend per flight, low loyalty | **Crest** |
| New High Spenders | new, high spending, high attachment | **Deep Blue** |

- **Current** – maintain loyalty and grow spend (comfort purchases, seat upgrades, lounge access)
- **Crest** – build loyalty in customers who fly rarely but spend a lot (bundle deals, points for early bookings)
- **Deep Blue** – premium tier (exclusive events, higher points multipliers)

Rollout in 3 phases: migrate existing customers → prepare campaigns → launch.

## 📊 Dashboards

| File | Framework | Content |
|---|---|---|
| [`dashboard.py`](dashboard.py) | Streamlit | EDA dashboard: filters (province, education, loyalty status), KPIs, geographic map, loyalty distribution |
| [`dashboard_v2.py`](dashboard_v2.py) | Dash | Cluster dashboard: 3D scatter, radar plot, boxplots, demographic filters, fuzzy membership per customer, CSV export |

```bash
pip install -r requirements.txt

# EDA dashboard (Streamlit)
streamlit run dashboard.py

# Cluster dashboard (Dash) → http://127.0.0.1:8050
python dashboard_v2.py
```

`dashboard_v2.py` reads `Data/clustered_df.csv`. This file is produced by the last section of `data_mining.ipynb`. If it is not found locally, it falls back to the copy on GitHub.

## 📁 Repository structure

```
.
├── README.md
├── requirements.txt
├── .gitignore
├── customer_data                  # cleaned customer-level dataset (used by dashboard.py)
├── Data/
│   ├── DM_AIAI_CustomerDB.csv     # raw customer data
│   ├── DM_AIAI_FlightsDB.csv      # raw monthly flight data
│   ├── DM_AIAI_Metadata.csv       # column descriptions
│   └── clustered_df.csv           # clustering output (used by dashboard_v2.py)
├── data_mining.ipynb              # full pipeline: cleaning → clustering → fuzzy → dashboard
├── poster_viz.ipynb               # figures for the poster + extra analysis
├── dashboard.py                   # Streamlit EDA dashboard
├── dashboard_v2.py                # Dash cluster dashboard
└── docs/
    ├── DM_report 2.pdf            # Report 1: data cleaning, feature engineering, EDA
    ├── DM_report.pdf              # Report 2: clustering, segments and loyalty strategy
    └── Poster_v1.pdf
```

## ▶️ Reproducing the analysis

1. Clone the repo and install the dependencies: `pip install -r requirements.txt`
2. Open `data_mining.ipynb` and run the cells top to bottom. The raw course files (`DM_AIAI_CustomerDB.csv`, `DM_AIAI_FlightsDB.csv`) are expected in `Data/`.
3. The last section exports `Data/clustered_df.csv` and starts the Dash dashboard.

## ⚠️ Limitations

- Segmentation is based on historical data (2019–2021) and does not capture changing behaviour or seasonality.
- Final combined clustering has a low silhouette (0.23), below the initial 50–70% target. Behavioural features form a continuum rather than discrete groups, which is why we added fuzzy clustering.
- Results depend on feature selection and hyperparameters.

**Future work:** dynamic/real-time booking data, survey feedback, and predictive models of customer movement between segments.

## 🛠️ Tech stack

Python · pandas · NumPy · scikit-learn · scikit-fuzzy · MiniSom · SciPy · UMAP · matplotlib · seaborn · Plotly · Dash · Streamlit

## 🤖 AI usage

Claude and ChatGPT were used for visualization ideas, syntax help and code troubleshooting; Grammarly for proofreading. All analytical insights and recommendations are the group's own work.

## 📚 References

- Statistics Canada (2017). *Does education pay?* Census in Brief, Cat. No. 98-200-X2016024.
- McKinsey & Company (2025). *The state of aviation: 2025 industry outlook.*
- Avram, B. (2019). Airlines Customer Segmentation in the Hyper-Competition Era. *Expert Journal of Marketing, 7(2)*, 137–143.
