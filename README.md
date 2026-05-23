# The War on Cash: India's Digital Payment Ecosystem (2019–2025)

A time-series analysis of RBI Payment Systems Indicator data covering six payment modes across six fiscal years, with a 12-month forward forecast.

<img width="1384" height="684" alt="image" src="https://github.com/user-attachments/assets/dd7156a0-4080-4a59-a456-d721ff413b2f" />

UPI’s transaction value steadily expands over time, increasingly driving the growth of India’s retail digital payment ecosystem.

---

## Project Overview

India's payment landscape has undergone a structural shift since 2019. This project tracks that shift quantitatively — from UPI's exponential rise to the declining share of debit cards and cash proxies — and builds forecasting models to project where transaction volumes are headed through FY2026.

**Primary question:** Is UPI displacing traditional payment rails, and what does that mean for card networks, banks, and fintechs operating in the Indian market?

**Target audience:** Banks, fintech startups, payment network analysts, policy researchers.

**Data source:** [RBI Payment Systems Indicator](https://m.rbi.org.in/Scripts/PSIUserView.aspx?Id=10) — monthly tables, Nov 2019 – Mar 2025.

---

## Dataset

Raw data extracted from RBI's pivot-style Excel tables and flattened into a long-format panel:

| Column | Description |
|---|---|
| `Date` | Month start date (monthly frequency) |
| `Payment_Mode` | UPI / NEFT / IMPS / RTGS / Credit / Debit |
| `Volume` | Transaction count |
| `Value` | Transaction value (₹) |
| `ATS` | Average ticket size = Value / Volume |
| `MoM` | Month-on-month growth (%) |
| `YoY` | Year-on-year growth (%) |
| `log_value` | log(Value) — used for SARIMA modeling |
| `is_covid_shock` | Binary: Mar–Oct 2020 lockdown window |
| `is_policy_change` | Binary: Jan 2020 Zero-MDR, Apr 2023 PPI cap |
| `is_festive_month` | Binary: Oct–Nov (Diwali/festive season) |
| `is_fy_end` | Binary: March (fiscal year-end spike) |
| `MoM_Anomaly` | Z-score based outlier flag |

390 rows × 6 modes × ~65 months.

---

## Project Structure

```
war_on_cash/
├── data/
│   └── rbi_payments_clean.csv       # cleaned long-format panel
├── notebooks/
│   ├── Phase_1_2.ipynb              # ETL + EDA
│   ├── Phase_3.ipynb                # diagnostic analysis
│   └── Phase_4_forecasting.ipynb   # forecasting models
└── README.md
```

---

## Phases

### Phase 1 — ETL & Data Cleaning

The RBI source data comes as multi-row headers in a pivot layout, with separate monthly sheets. The pipeline avoids header-based parsing and instead uses deterministic column indexing with keyword validation to handle schema inconsistencies across years.

Key steps:
- Flatten pivot tables into a strict `(Date, Payment_Mode, Volume, Value)` long format
- Standardise units across modes (RTGS reports in ₹ Crore; retail modes in ₹)
- Engineer ATS, MoM, YoY, log transforms, and binary event flags

### Phase 2 — Exploratory Data Analysis

- **Trend analysis:** Log-scale comparison of UPI vs debit cards confirms a substitution effect. UPI volume grew ~13× from FY2020 to FY2025; debit card volume stagnated.
- **Seasonality:** Monthly boxplots confirm Oct–Nov festive spikes across UPI and credit cards (stronger in UPI by volume, stronger in credit cards by value).
- **Correlation:** UPI–debit card correlation ≈ −0.91. UPI, NEFT, RTGS move together (+0.98–0.99), consistent with the same underlying macro growth.
- **ATS divergence:** UPI ATS declining (₹1,800 → ₹1,354), credit card ATS rising (₹3,000 → ₹4,393). Classic mass-market vs premium segmentation.

### Phase 3 — Diagnostic Analysis

- **COVID impact (Mar 2020):** Trend-corrected Δ% analysis (Welch t-test, ±6-month window) shows credit cards hit hardest (−35%), UPI smallest deviation (−18%), recovered within 3 months. Behavioural lock-in happened here.
- **Policy events:**
  - Zero-MDR mandate (Jan 2020): directionally consistent but statistically inconclusive — COVID arrived two months later and contaminated the post-event window.
  - PPI interchange cap (Apr 2023): clearest structural shift in the dataset. IMPS shows significant positive deviation post-cap; credit card shows corresponding negative. Wallet flows migrated to bank-account-based rails.
- **Pareto:** UPI alone crosses the 80% cumulative transaction volume threshold in FY2025.
- **Structural breaks:** Chow test + PELT algorithm confirm two system-wide break periods: COVID shock (2020) and PPI cap (Apr 2023).
- **Granger causality:** UPI → IMPS significant at multiple lags (shared IMPS rail). UPI → NEFT present at short lags, weakens post-2023, suggesting decoupling as modes serve increasingly distinct use cases.

### Phase 4 — Forecasting

**Train:** Nov 2019 – Dec 2022 | **Test:** Jan 2023 – Dec 2024 (24 months) | **Horizon:** Apr 2025 – Mar 2026

Three models compared against a naive seasonal baseline (ŷ(t) = y(t−12)):

| Model | Test MAPE | Notes |
|---|---|---|
| Holt-Winters ETS(A,A,A) | 3.45% | Primary model — additive trend+seasonal, fits UPI's near-linear growth |
| SARIMA(0,2,1)×(0,1,0)[12] | 5.78% | Log-scale fit, AIC-selected, Ljung-Box residuals = white noise |
| Prophet | 10.01% | Multiplicative seasonality + COVID/policy regressors; underperforms at test tail |
| Naive seasonal | 40.21% | Benchmark |

**Primary model:** Holt-Winters ETS(A,A,A) — lowest MAPE, interpretable parameters, appropriate for linear growth series.

**Scenario A:** Baseline CAGR continuation (Holt-Winters output).  
**Scenario B:** +15% value uplift over Scenario A, modelling Credit-on-UPI rollout. The 15% is derived from approximately half the differential between credit card CAGR (~18.5%) and UPI baseline CAGR — treat as a sensitivity, not a point estimate.

UPI forecast (Apr 2025 – Mar 2026):

| | Scenario A | Scenario B |
|---|---|---|
| Min monthly | ₹24.07T | ₹27.68T |
| Peak monthly | ₹28.99T (Mar 2026) | ₹33.33T (Mar 2026) |
| 12-month uplift | — | +₹47.5T |

Per-mode forecasts use auto-selected SARIMA per mode with Holt-Winters fallback on convergence failures.

### Phase 5 — Power BI Dashboard 

Four views planned:
1. **Executive view** — total digital transaction value (monthly), MoM growth %
2. **Category war** — race bar chart: UPI vs NEFT vs cards over time
3. **Ticket size tracker** — CC ATS rising (premiumisation) vs UPI ATS falling (mass adoption)

---

## Key Findings

1. **UPI is dominant by volume but not revenue-generating for card networks.** UPI P2M MDR is 0% (regulatory cap). Card networks earn on transactions above ~₹5,000 - the segment where UPI's average ticket size (₹1,354) doesn't compete.

2. **Credit card ATS is rising (+28.7% since FY2019).** The market is self-segmenting: UPI captures micro-payments, cards capture high-value discretionary spend. This is Visa/Mastercard's structural moat.

3. **Credit-on-UPI is the convergence point.** If credit rails extend onto UPI's QR infrastructure, card networks can earn MDR on UPI-initiated credit transactions. This is the key scenario to watch in FY2026.

4. **The 2023 PPI cap was more consequential than the 2020 Zero-MDR mandate.** IMPS gained; credit cards lost. Wallet-funded card spend migrated to bank-account-based rails.

5. **UPI's COVID resilience cemented behavioural adoption.** Recovery took ~3 months vs 12+ months for cards. Essential payment behaviour (groceries, utilities, P2P) locked in on UPI during lockdown and hasn't reverted.

---
## Plots
<img width="1184" height="583" alt="image" src="https://github.com/user-attachments/assets/d6539e10-6065-44bc-91bc-777dc60000f6" />
UPI’s exponential post-2020 growth sharply diverges from the steady decline in debit card transaction volume, highlighting a structural shift toward real-time digital payments in India.

<img width="1184" height="583" alt="image" src="https://github.com/user-attachments/assets/880a7416-20d9-495a-9832-bdfdfb13d06e" />
UPI is increasingly dominating low-value retail payments, while credit cards are shifting toward higher-ticket spending.

<img width="1384" height="684" alt="image" src="https://github.com/user-attachments/assets/d55b382a-a3bc-4a67-8bbe-3d9ee6fcc51e" />
UPI’s transaction value steadily expands over time, increasingly dominating India’s retail digital payment ecosystem.

<img width="2479" height="1487" alt="image" src="https://github.com/user-attachments/assets/7eb8b9d5-1420-45f1-970a-05134f31ff8d" />
UPI’s post-2020 acceleration sharply diverges from every major payment mode, revealing a structural shift in India’s payment ecosystem driven by COVID recovery and policy changes.

<img width="2776" height="1268" alt="image" src="https://github.com/user-attachments/assets/b3c82202-0224-44cf-a3b7-37849ef02315" />
UPI rapidly consolidates transaction dominance over FY2020–FY2025, crossing the 80% volume-share threshold and increasingly centralizing India’s retail payment ecosystem.

---

## Limitations

- Only 65 months of monthly data per mode — seasonal estimates at the tail of the HW model are sensitive to the last 12 months.
- COVID shock is handled as a binary flag in Prophet; SARIMA/HW treat it as a structural shock absorbed into residuals. A proper intervention model (e.g., ARIMAX with pulse/step dummies) would be more principled.
- Scenario B (+15%) is a working assumption, not modelled endogenously. No demand elasticity or adoption curve is estimated.
- Debit card data in RBI tables includes ATM withdrawals, making it a noisy proxy for POS card usage. The UPI–debit substitution signal is real but the magnitude is overstated.
- No macro covariates (GDP, internet penetration, smartphone adoption) included. These are correlated with UPI growth but would require a longer sample to estimate reliably.
