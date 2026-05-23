# The War on Cash

## A Time-Series Analysis of India’s Digital Payment Ecosystem & Future Adoption Forecasting

> UPI went from a convenient payment method to national infrastructure in less than a decade.
> This project studies that transformation using RBI payment system data, statistical forecasting, and business-oriented analysis.

---

## Overview

India’s payment ecosystem has changed dramatically since 2020.

QR codes replaced cash for small-ticket transactions.
UPI volumes exploded.
ATM withdrawals slowed relative to digital growth.
Cards evolved toward higher-value usage while real-time payment systems became dominant.

This project analyzes that shift using RBI Payment Systems Indicator data from 2019–2025.

The work combines:

* ETL and data engineering
* exploratory analysis
* event-based diagnostics
* statistical forecasting
* dashboard-driven business storytelling

The goal was not just to forecast transactions, but to understand how payment behavior itself is changing in India.

---

## Project Objectives

* Analyze the growth trajectory of Indian digital payments
* Compare digital rails against traditional payment systems
* Study the impact of structural events like COVID-19
* Forecast payment transaction trends for FY2026
* Translate raw financial data into actionable business insights

---

## Target Audience

* Banks
* Fintech startups
* Payment infrastructure teams
* Policy makers
* BFSI analysts
* Financial data science enthusiasts

---

## Key Findings

* UPI showed near-exponential growth post-2020
* Debit card POS growth stagnated after COVID
* ATM withdrawal growth weakened relative to digital payments
* Seasonal spikes consistently appeared during festive periods
* UPI increasingly behaves like a network-adoption curve rather than a standard financial time series
* Holt-Winters and SARIMA consistently outperformed naive seasonal baselines
* Credit card ticket sizes increased while UPI ticket sizes decreased, suggesting simultaneous premiumization + mass adoption dynamics

---

## Dataset

Source: [RBI Payment Systems Indicators](https://m.rbi.org.in//Scripts/PSIUserView.aspx?Id=10&utm_source=chatgpt.com)

Coverage:

* Nov 2019 → Mar 2025
* Monthly frequency
* Multiple retail and institutional payment systems

Payment modes analyzed:

* UPI
* IMPS
* NEFT
* RTGS
* Credit Cards
* Debit Cards
* ATM Withdrawals
* PPI Wallets

---

# Project Workflow

---

## Phase 1 — Data Preparation & ETL

The RBI data is not available in a clean ML-ready format.

The raw source contains:

* monthly tables,
* inconsistent layouts,
* pivoted structures,
* and changing category names over time.

The ETL pipeline:

* extracted RBI payment tables,
* flattened monthly panels,
* standardized payment categories,
* cleaned missing values,
* and created a unified time-series dataset.

### Engineered Features

| Feature            | Description                       |
| ------------------ | --------------------------------- |
| `ATS`              | Average Ticket Size               |
| `MoM`              | Month-over-month growth           |
| `YoY`              | Year-over-year growth             |
| `log_value`        | Log-transformed transaction value |
| `is_covid_shock`   | COVID disruption indicator        |
| `is_policy_change` | Policy event flag                 |
| `is_festive_month` | Festival-season indicator         |

Notebook:

```bash
notebooks/phase1_2_etl_eda.ipynb
```

---

## Phase 2 — Exploratory Data Analysis

The EDA phase focused on understanding behavioral and structural changes in Indian payments.

### Analyses Performed

### UPI vs Traditional Rails

Compared:

* UPI growth
* Debit card POS activity
* ATM withdrawal trends
* NEFT and IMPS adoption

### Seasonality Detection

Observed recurring spikes during:

* Diwali
* festive shopping periods
* fiscal year-end cycles

### Ticket Size Analysis

Used Average Ticket Size (ATS) to distinguish:

* micro-payment systems
* institutional settlement systems
* premium consumer spending behavior

### Correlation Analysis

Tested the “cashless economy” hypothesis:

* Does increasing UPI adoption correlate with declining ATM withdrawals?

### Visualizations Included

* Time-series plots
* Correlation heatmaps
* CAGR comparisons
* Distribution analysis
* Growth decomposition charts

Notebook:

```bash
notebooks/phase1_2_etl_eda.ipynb
```

---

## Phase 3 — Diagnostic Analysis

This phase focused on understanding *why* trends changed.

### Event Impact Analysis

Major economic and behavioral events were mapped onto the timeline.

| Event                    | Observed Impact                                                |
| ------------------------ | -------------------------------------------------------------- |
| COVID-19 Lockdown (2020) | ATM withdrawals dropped while digital transactions accelerated |
| UPI Expansion            | Sustained increase in retail transaction volume                |
| Policy Changes           | Temporary shifts in wallet/PPI behavior                        |

### Pareto Analysis

Used Pareto-style contribution analysis to determine:

* which payment modes dominate total transaction volume,
* and whether the ecosystem is becoming increasingly concentrated around UPI.

### Business Interpretation

This phase translated raw trends into strategic implications such as:

* declining dependence on cash infrastructure,
* increasing need for scalable payment servers,
* and shifting consumer transaction behavior.

Notebook:

```bash
notebooks/phase3_diagnostic_analysis.ipynb
```

---

## Phase 4 — Forecasting & Scenario Planning

The forecasting phase predicts transaction trends for FY2026.

### Forecasting Models Used

| Model            | Purpose                               |
| ---------------- | ------------------------------------- |
| Seasonal Naive   | Benchmark baseline                    |
| SARIMA           | Seasonal autoregressive forecasting   |
| Holt-Winters ETS | Trend + seasonality modelling         |
| Prophet          | Structural break/changepoint handling |

### Train/Test Split

| Split | Range               |
| ----- | ------------------- |
| Train | Nov 2019 → Dec 2022 |
| Test  | Jan 2023 → Dec 2024 |

### Evaluation Metric

* MAPE (Mean Absolute Percentage Error)

### Scenario Forecasting

#### Scenario A — Baseline Continuation

Assumes current CAGR trends continue.

#### Scenario B — Credit-on-UPI Expansion

Simulates increased UPI transaction value growth due to accelerated credit-linked UPI adoption.

### Forecasting Questions Explored

* Can classical forecasting models capture rapid fintech adoption?
* At current growth rates, when does UPI surpass traditional card systems in transaction value?
* How stable are forecasts after structural shocks like COVID?

Notebook:

```bash
notebooks/phase4_forecasting.ipynb
```

---

## Phase 5 — Business Dashboard

The final deliverable converts the analytical pipeline into a business-facing dashboard.

### Dashboard Views

### Executive View

* Monthly digital transaction value
* MoM growth rates
* ecosystem-wide KPIs

### Category War

A race-style visualization showing:

* UPI overtaking cards and NEFT over time

### Ticket Size Tracker

Tracks:

* UPI mass adoption
* credit-card premiumization
* transaction behavior evolution

### Intended Use

Designed for:

* business teams,
* policy discussions,
* strategic presentations,
* and payment trend monitoring.

Dashboard:

```bash
dashboard/phase5_dashboard.pbix
```

---

# Repository Structure

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

# Technical Stack

## Languages & Libraries

* Python
* Pandas
* NumPy
* Matplotlib
* Seaborn
* Statsmodels
* Prophet
* pmdarima

## Tools

* Jupyter Notebook
* Power BI / Tableau

---

# Running the Project

```bash
pip install -r requirements.txt
jupyter notebook notebooks/phase4_forecasting.ipynb
```

---

# Future Extensions

Potential next steps:

* Transformer-based time-series forecasting
* Bayesian structural models
* State-wise decomposition
* Real-time RBI API ingestion
* Causal inference for policy interventions
* UPI network saturation modelling

---

# Limitations

* Only ~5 years of monthly observations
* Structural breaks from COVID-19
* No macroeconomic exogenous variables
* No demographic segmentation
* Scenario forecasts are assumption-driven rather than causal simulations

---

# Final Note

India’s payment ecosystem is evolving faster than many traditional financial forecasting assumptions can adapt.

This project attempts to study that transition using publicly available RBI data, statistical modelling, and business-oriented analysis — somewhere between a financial systems study, a fintech growth analysis, and a time-series forecasting experiment.
