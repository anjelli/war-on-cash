# The War on Cash  
### Tracking how India stopped swiping, started scanning, and quietly rebuilt its payment system.

India’s digital payments story is usually told through headlines:
- “UPI crosses record transactions”
- “Cashless economy”
- “Fintech boom”

This project looks underneath those headlines.

Using RBI Payment Systems Indicator data from 2019–2025, the analysis explores how payment behaviour in India structurally changed after COVID, policy interventions, and large-scale UPI adoption.

The project combines:
- ETL pipelines
- exploratory analysis
- structural break diagnostics
- forecasting models
- and dashboard-driven business analysis

The objective was not just forecasting.

It was understanding whether India’s payment ecosystem is simply digitizing — or reorganizing itself around entirely different transaction behavior.

---

# What This Project Studies

This repository analyzes:
- the rise of UPI,
- the decline of debit-card dependence,
- changing transaction-size behavior,
- the effect of COVID and regulation,
- and future payment adoption trends.

The analysis spans:
- retail payments,
- enterprise settlement systems,
- consumer behavior,
- and macro-level transaction infrastructure.

---

# Core Questions

- Is UPI replacing traditional payment rails or merely coexisting with them?
- Did COVID accelerate permanent payment behavior changes?
- Are cards becoming premium-only instruments?
- Can classical forecasting models handle near-exponential fintech adoption?
- What happens to transaction value if Credit-on-UPI scales aggressively?

---

# Dataset

Source: RBI Payment Systems Indicators

Coverage:
- Nov 2019 → Mar 2025
- Monthly frequency
- 390 observations across 6 major payment systems

Modes analyzed:
- UPI
- IMPS
- NEFT
- RTGS
- Credit Cards
- Debit Cards

The raw RBI files were not analysis-ready.

The original data contained:
- pivoted tables,
- inconsistent layouts,
- multi-row headers,
- and changing formatting across reports.

The ETL pipeline converted this into a clean long-format panel dataset suitable for:
- time-series analysis,
- forecasting,
- and dashboarding.

---

# Project Structure

```bash
the-war-on-cash/
│
├── data/
│   ├── raw/
│   └── processed/
│
├── notebooks/
│   ├── phase1_2_etl_eda.ipynb
│   ├── phase3_diagnostic_analysis.ipynb
│   └── phase4_forecasting.ipynb
│
├── dashboard/
│   └── phase5_dashboard.pbix
│
├── outputs/
│   ├── figures/
│   ├── forecasts/
│   └── reports/
│
├── src/
│   ├── preprocessing.py
│   ├── forecasting.py
│   └── utils.py
│
├── requirements.txt
├── README.md
└── .gitignore
```

---

# Phase 1 — ETL & Data Preparation

The first phase focused on building a robust processing pipeline for RBI payment data.

### Tasks Completed
- Extracted RBI payment system tables
- Flattened pivot-style monthly reports
- Standardized payment categories
- Cleaned inconsistent schema structures
- Built a long-format panel dataset

### Feature Engineering

Derived variables created:
- Average Ticket Size (ATS)
- MoM Growth %
- YoY Growth %
- Log-transformed transaction values
- COVID shock indicators
- Policy-event markers
- Festive season flags
- Fiscal year-end indicators

One important realization from this phase:

UPI behaves very differently from traditional banking rails.  
Its transaction growth resembles platform adoption more than standard financial growth.

---

# Phase 2 — Exploratory Data Analysis

The EDA phase focused on identifying structural trends in Indian payment behavior.

## UPI vs Debit Cards

UPI transaction growth accelerated continuously across the dataset.

Debit card usage, meanwhile:
- flattened,
- weakened post-COVID,
- and steadily lost relative importance.

This suggests a direct substitution effect:
QR-based payments increasingly replaced card-based retail behavior.

---

## Seasonality

Strong seasonal spikes appeared consistently during:
- October–November,
- festive shopping periods,
- and fiscal year-end cycles.

The seasonality was strongest in:
- UPI
- Credit Cards

This became important later during SARIMA modelling.

---

## Correlation Analysis

The correlation matrix revealed:
- strong positive correlation among digital payment rails,
- and strong negative correlation between UPI and Debit Cards.

Interpretation:
digital systems are growing together — but UPI is absorbing the majority of behavioral migration.

---

## Average Ticket Size (ATS)

One of the most interesting findings in the dataset.

### UPI
ATS steadily declined over time:
- indicating deeper penetration into low-value daily retail transactions.

### Credit Cards
ATS increased significantly:
- indicating movement toward premium and discretionary spending.

This creates a behavioral split:

| System | Dominant Use Case |
|---|---|
| UPI | High-frequency retail |
| Credit Cards | Premium/high-ticket spending |
| RTGS | Enterprise-scale settlement |

---

# Phase 3 — Diagnostic Analysis

This phase focused on understanding *why* trends shifted.

---

## COVID Shock Analysis

COVID disrupted every payment mode.

But the recovery behavior differed sharply.

### UPI
- smallest long-term deviation,
- fastest recovery,
- returned to trend within ~3 months.

### Debit Cards
- recovered partially,
- but never regained pre-COVID slope.

### Credit Cards
- experienced the sharpest retail collapse.

This suggests that UPI’s dominance came less from immunity and more from resilience.

Essential transactions:
- groceries,
- utilities,
- peer-to-peer transfers

continued running through UPI even during lockdown periods.

That appears to be the moment long-term behavioral lock-in occurred.

---

## Policy Impact — PPI Interchange Cap

The April 2023 PPI regulation created one of the clearest structural shifts in the dataset.

Observed effects:
- IMPS strengthened,
- wallet-linked flows weakened,
- card transaction trajectories softened.

The broader implication:
India’s payment ecosystem increasingly shifted away from closed wallet infrastructure toward interoperable account-based rails.

---

## Structural Break Detection

Structural break analysis was performed using:
- Chow Tests
- PELT change-point detection

Two major regime shifts were consistently detected:
1. COVID Shock (2020)
2. Policy Shift (Apr 2023)

UPI and IMPS displayed multiple breakpoints, indicating phased adoption acceleration.

---

## Pareto Analysis

By FY2025:
- UPI alone crossed the 80% cumulative volume threshold.

The market increasingly became concentrated around:
- UPI,
- NEFT,
- and IMPS.

Debit and Credit systems continued growing in absolute value but lost relative transaction share.

---

# Phase 4 — Forecasting & Scenario Planning

The forecasting phase focused on predicting FY2026 transaction trends.

---

# Forecasting Models

| Model | Purpose |
|---|---|
| Seasonal Naive | Baseline benchmark |
| SARIMA | Seasonal autoregressive forecasting |
| Holt-Winters ETS | Trend + seasonality modelling |
| Prophet | Changepoint-aware forecasting |

---

# Forecast Setup

| Split | Range |
|---|---|
| Train | Nov 2019 → Dec 2022 |
| Test | Jan 2023 → Dec 2024 |

Forecast horizon:
- Apr 2025 → Mar 2026

Evaluation metric:
- MAPE (Mean Absolute Percentage Error)

---

# Model Results

| Model | MAPE |
|---|---|
| Holt-Winters ETS(A,A,A) | 3.45% |
| SARIMA(0,2,1) | 5.78% |
| Prophet | 10.01% |
| Naive Seasonal | 40.21% |

Holt-Winters consistently produced the most stable forecasts.

One unexpected finding:
classical statistical models generalized surprisingly well despite structural shocks and rapid adoption growth.

---

# Scenario Forecasting

Two scenarios were modeled.

## Scenario A — Baseline Continuation
Assumes existing CAGR trends continue.

## Scenario B — Credit-on-UPI Expansion
Applies a +15% uplift assumption to simulate aggressive Credit-on-UPI adoption.

Projected impact:
- ₹47.5T additional annual transaction value under accelerated adoption.

---

# Advanced Extensions

Additional experimental models included:

### DLinear
Neural decomposition + exogenous fusion model using:
- ATS
- policy indicators
- festive flags
- COVID markers

### BSTS + MinT Reconciliation
Implemented:
- Bayesian structural time-series decomposition
- hierarchical reconciliation
- covariance-based aggregation correction

These were exploratory additions rather than production-primary models.

---

# Phase 5 — Business Dashboard

The final dashboard translates the analysis into business-facing insights.

Views included:
- Executive KPI monitoring
- Retail vs enterprise payment trends
- Category competition analysis
- ATS segmentation tracking
- MoM growth monitoring

The dashboard was designed for:
- BFSI teams,
- strategy discussions,
- fintech presentations,
- and policy interpretation.

---

# Technical Stack

## Languages & Libraries
- Python
- Pandas
- NumPy
- Statsmodels
- Prophet
- pmdarima
- Matplotlib
- Seaborn

## Tools
- Jupyter Notebook
- Power BI
- Tableau

---

# Running the Project

```bash
pip install -r requirements.txt
jupyter notebook notebooks/phase4_forecasting.ipynb
```

---

# Key Takeaways

- UPI is behaving more like a national-scale platform than a conventional payment product.
- India’s payment ecosystem is increasingly bifurcated:
  - low-ticket retail via UPI,
  - high-ticket spending via cards,
  - enterprise settlement via RTGS.
- COVID accelerated behavioral shifts that appear structurally persistent.
- Policy interventions meaningfully altered payment-routing behavior.
- Classical forecasting methods remain highly competitive on medium-horizon fintech forecasting problems.

---

# Limitations

- Limited historical depth (~5 years)
- Structural discontinuities from COVID
- No macroeconomic exogenous variables
- No demographic segmentation
- Forecast scenarios are assumption-driven rather than causal simulations

---

# Final Observation

India’s payment ecosystem is no longer just becoming digital.

It is reorganizing around:
- real-time infrastructure,
- interoperable rails,
- and frictionless retail behavior.

This project attempts to measure that transition through time-series modelling, structural diagnostics, and business-oriented financial analysis.
