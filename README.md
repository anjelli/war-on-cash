# THE WAR ON CASH
## A Time-Series Analysis of India’s Digital Payment Ecosystem
*with Future Adoption Forecasting*

---

# Executive Summary
This project analyzes India’s retail payment ecosystem using official monthly transaction data from the Reserve Bank of India. The analysis breaks down into four phases: data engineering, exploratory analysis, diagnostic testing, and forecasting with scenario planning.

The data reveals a structural shift. UPI did not merely grow alongside other payment methods. It permanently displaced them at the retail level and captured over 89% of all transaction volume by FY2025. The forecasting model projects UPI monthly value will reach ₹28.99T under baseline conditions and ₹33.33T under a credit-embedded scenario by March 2026.

### Key Metrics at a Glance (March 2025)
* **UPI Monthly Value:** ₹24.77T | Volume: 18.3 Billion transactions
* **UPI Volume Share:** 89.4% of all retail transactions (FY2025)
* **UPI Value CAGR:** 62.0% (FY2020 to FY2025)
* **Credit Card ATS:** ₹4,393 vs UPI ATS: ₹1,354 (3.2× gap, widening)
* **Primary Forecast Model:** Holt-Winters ETS(A,A,A) | Test MAPE: 3.45%
* **FY26 Baseline Peak:** ₹28.99T/month (Mar 2026)
* **FY26 Upside Peak:** ₹33.33T/month (Mar 2026) [Credit-on-UPI +15% scenario]

---

# Phase 1: Data Engineering and ETL
The raw RBI dataset presented structural problems like multi-row headers and inconsistent layouts across years. The ETL pipeline handles this automatically using deterministic column indexing and keyword validation instead of fragile header parsing.

### Data Scope
* **Source:** RBI Payment System Indicators (Official)
* **File:** `rbi_payments_clean.xlsx` to `rbi_payments_clean.csv`
* **Period:** November 2019 to March 2025 (65 months)
* **Observations:** 390 rows (6 payment modes × 65 months)
* **Payment Modes:** UPI, NEFT, IMPS, RTGS, Credit Card, Debit Card

### Output Schema
The raw data was flattened into a strict long-format panel:
* `Date`: first day of each calendar month
* `Payment_Mode`: one of six modes
* `Volume`: number of transactions (standardised)
* `Value`: transaction value in ₹ Crore (standardised)

### Feature Engineering
The following derived variables were constructed and validated:
* Average Ticket Size (ATS) = Value ÷ Volume
* Month-on-Month Growth % (MoM)
* Year-on-Year Growth % (YoY)
* Log-transformed Volume and Value (for modelling stability)

Binary event flag matrix introduced for downstream modelling:
* `is_covid_shock`: lockdown periods (Mar to Jun 2020)
* `is_policy_change`: Zero-MDR (Jan 2020), PPI Interchange Cap (Apr 2023)
* `is_festive_month`: October and November (Diwali season)
* `is_fy_end`: March (fiscal year-end settlement spike)
* `MoM_Anomaly`: z-score based statistical outlier flag

---

# Phase 2: Exploratory Data Analysis

### 1. Trend Analysis: UPI vs Debit Cards (Log Scale)
On a log scale, UPI’s growth trajectory is visibly steeper than all peer modes across the entire study period. Debit card volume remained flat or declined, confirming that buyers directly substituted debit cards with UPI at the point of sale.

### 2. Seasonality
Monthly boxplots confirmed strong seasonal demand shocks in October and November during Diwali. The pattern is present in both UPI and credit cards. However, the festive uplift is quantifiably stronger in UPI volume (+4.2%), while credit cards capture a higher-value festive premium (+9.9%). Both modes spike in March due to fiscal year-end settlement behavior.

### 3. Correlation Matrix
The correlation matrix reveals the structural split in India’s payment market:
* **UPI ↔ Debit Card:** r ≈ -0.91 (strong negative correlation indicating direct substitution)
* **UPI ↔ NEFT:** r ≈ +0.99 (co-movement driven by shared economic growth)
* **UPI ↔ RTGS:** r ≈ +0.91 (digital ecosystem expansion)
* **UPI ↔ Credit:** r ≈ +0.98 (parallel premium-segment growth)

### 4. Average Ticket Size (ATS) Divergence
ATS trends expose the market segmentation that defines the strategic landscape:
* **UPI ATS:** ₹1,800 to ₹1,354 (12.8% decline) as UPI penetrates daily, low-value retail.
* **Credit Card ATS:** ₹3,000 to ₹4,393 (28.7% growth) as cards shift toward premium, discretionary spending.
The 3.2× and widening ATS gap is the single most strategically significant metric for card networks.

### 5. Value Distribution
Stacked area analysis confirms that UPI’s rise represents value migration, not just volume migration. Total digital transaction value expanded materially. UPI’s share rose at the expense of NEFT and IMPS within the retail layer. RTGS remains unchallenged in the enterprise settlement tier.

---

# Phase 3: Diagnostic Analysis
Phase 3 applies rigorous econometric methods to test hypotheses about the structural forces driving India’s payment transition. All tests were conducted on log-transformed volume series.

### P1: The Big Picture Timeline
On a log scale, three inflection points are structurally visible and analytically verified:
* **April 2020 (COVID Full Lockdown):** Sharp simultaneous drop across all modes. UPI recovered within 3 months. Debit card volume never resumed its prior growth trajectory.
* **Post-unlock 2021:** UPI re-accelerated meaningfully. All other modes did not.
* **April 2023 (PPI Interchange Cap):** A visible and statistically confirmed kink in IMPS and credit card trajectories, explored in P3.

### P2: COVID-19 Impact (Trend-Corrected Event Study)
Trend-corrected Δ% bars measure deviation from each mode’s own pre-event growth trajectory, removing baseline differences. Key findings:
* UPI saw the smallest drop during the lockdown because essential payments continued digitally.
* Credit cards took the largest hit due to the collapse of retail and travel spending.
* Debit cards recovered in absolute terms but never regained their pre-COVID slope.
This period marks the behavioral lock-in. It is the exact point when UPI became the default payment habit for most Indian consumers.

### P3: PPI Interchange Cap Impact (Apr 2023)
The April 2023 RBI cap on interchange fees for digital wallets produced the clearest policy-driven structural shift in the dataset:
* **IMPS:** Significant positive trend-corrected Δ% post-cap. Flow migrated from wallets to bank-account-based IMPS as wallet providers lost their commercial incentive.
* **Credit Card:** Corresponding negative deviation, as wallet top-ups via credit cards were suppressed.
* The Zero-MDR mandate (Jan 2020) was directionally consistent but statistically inconclusive because COVID arrived two months later and contaminated the post-event window.

### P4: Pareto: Who Owns the Volume?
By FY2025, the Indian payment market is effectively a two-mode story by transaction count:
* UPI alone exceeds 80% cumulative volume share in the FY2025 Pareto chart.
* UPI, NEFT, and IMPS combined account for ~96.8% of total retail transaction volume.
* UPI’s share increases continuously. No fiscal year shows a reversal between FY2020 and FY2025.
* Card share is declining in relative terms even as absolute values grow.

### Structural Break Detection (Chow Test and PELT Algorithm)
Two independent methods confirm the same two system-wide structural break periods:
* **Chow Test:** Hypothesis-based test comparing model fit before and after known event dates.
* **PELT Algorithm:** Data-driven change-point detection requiring no prior assumptions.
Both methods converge on the 2020 COVID shock and the April 2023 policy shift as the two definitive structural breaks in India’s payment ecosystem.

### Granger Causality Analysis
Granger analysis tests whether past UPI values improve forecasts of other modes beyond those modes’ own history (predictive directionality, not causation):
* **UPI → IMPS:** Significant at multiple lags. This makes economic sense since UPI uses the IMPS inter-bank settlement rail.
* **UPI → NEFT:** Present at short lags but weakens post-2023. This likely reflects shared macro drivers rather than substitution.
* **UPI → Debit (ATM):** Weak and inconsistent. Proxy noise from POS and online transactions obscures the cash-displacement signal.
The Granger relationships weaken significantly post-2023, which aligns with the structural decoupling identified by the break detection analysis.

---

# Phase 4: Forecasting and Scenario Planning
Phase 4 contains the primary deliverables of this project. Three production-grade forecasting models were evaluated against a naïve seasonal benchmark on a strictly held-out 24-month test set (January 2023 to December 2024). The winning model then generated 12-month forward projections under two strategic scenarios.

### Stationarity Analysis
ADF and KPSS tests determined differencing orders for each mode. UPI required d=2 (double differencing) due to its sustained superlinear growth trend. All other modes required d=1.

### Naïve Seasonal Benchmark
The zero-parameter benchmark yielded a test MAPE of 40.21%. All production models had to beat this threshold to be accepted.

### Model Comparison: 24-Month Hold-Out Test Set (Jan 2023 to Dec 2024)

| Model | MAPE (%) | RMSE (₹ Cr) | MAE (₹ Cr) | Rank | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Holt-Winters ETS(A,A,A)** | **3.45** | **₹85,074** | **₹65,891** | **#1** | **PRIMARY ✓** |
| SARIMA(0,2,1)×(0,1,0,12) | 5.78 | ₹1,46,372 | ₹1,14,295 | #2 | Accepted ✓ |
| Prophet (COVID + Policy) | 10.01 | ₹2,38,215 | ₹1,86,504 | #3 | Accepted ✓ |
| Naïve Seasonal (Benchmark) | 40.21 | ₹7,88,531 | ₹7,40,882 | – | Baseline |

### Primary Model: Holt-Winters ETS(A,A,A)
Specification: Additive Error, Additive Trend, Additive Seasonal. We selected this because UPI exhibits near-linear value growth over the study period, making the additive specification statistically optimal.
* **Smoothing parameters:** α (level) = 0.9722, β (trend) = 0.1342, γ (seasonal) = 0.0000
The Ljung-Box test passed at all lags, confirming the residuals are white noise. This is the production model for all Scenario A projections.

### Supporting Models
* **SARIMA(0,2,1)×(0,1,0,12):** Selected by AIC via stepwise auto_arima. Passed Ljung-Box white-noise test. MAPE 5.78%.
* **Prophet (COVID + Policy Regressors):** changepoint_prior_scale = 0.3 (tuned on validation). Incorporates COVID and policy exogenous regressors. MAPE 10.01%.

## Scenario Construction

### Scenario A: Baseline CAGR (Holt-Winters Primary Output)
Scenario A represents the model’s forward projection from the production Holt-Winters model refitted on the full dataset (Nov 2019 to Mar 2025). It incorporates the existing growth trajectory, confirmed seasonal pattern, and trend momentum.

### Scenario B: Credit-on-UPI Uplift (+15%)
Scenario B applies a 15% uplift multiplier to Scenario A. This reflects the incremental GMV impact of embedding credit rails into UPI’s QR-code transaction network. Current UPI P2M MDR is zero, while credit-on-UPI carries a nascent ~0.3% MDR. This scenario relies on active market trends. Credit-on-UPI is a live, NPCI-supported product gaining broad merchant adoption.

### 12-Month Forecast: Scenario A vs B (Apr 2025 to Mar 2026)

| Month | Scen A (Base) | 95% CI | Scen B (+15%) | Uplift |
| :--- | :--- | :--- | :--- | :--- |
| Apr 2025 | ₹24.07T | ₹23.23T to ₹24.90T | ₹27.68T | +₹3.61T |
| Jul 2025 | ₹25.36T | ₹23.69T to ₹27.02T | ₹29.16T | +₹3.80T |
| Oct 2025 | ₹27.26T | ₹25.06T to ₹29.46T | ₹31.35T | +₹4.09T |
| Jan 2026 | ₹27.95T | ₹25.31T to ₹30.58T | ₹32.14T | +₹4.19T |
| Mar 2026 | ₹28.99T | ₹26.10T to ₹31.87T | ₹33.33T | +₹4.34T |

* **12-month aggregate uplift (Scenario B minus Scenario A): ₹47.5 Trillion**
* **Scenario A peak (Mar 2026):** ₹28.99T/month
* **Scenario B peak (Mar 2026):** ₹33.33T/month

## All-Modes Forecast: SARIMA + BSTS per Mode (FY2026 Range)

| Mode | Model | SARIMA FY26 | BSTS FY26 | Characterisation |
| :--- | :--- | :--- | :--- | :--- |
| **UPI** | SARIMA(0,2,1) | ₹23.84T to ₹26.90T | ₹24.07T to ₹28.99T | High growth |
| **NEFT** | SARIMA(0,1,2) | ₹37.02T to ₹53.71T | ₹33.98T to ₹54.80T | Stable enterprise |
| **Credit** | SARIMA(0,1,2) | ₹1.85T to ₹2.35T | ₹1.79T to ₹2.56T | Premium growth |
| **Debit** | SARIMA(0,1,2) | ₹0.24T to ₹0.35T | ₹0.33T to ₹0.43T | Structural decline |
| **IMPS** | SARIMA(2,1,0) | ₹5.59T to ₹6.59T | ₹5.99T to ₹8.19T | Steady rail |
| **RTGS** | SARIMA(0,1,1) | ₹169T to ₹260T | ₹157T to ₹234T | High-value anchor |

*Note: BSTS (Bayesian Structural Time Series) models use state-space decomposition with stochastic local linear trend, trigonometric seasonal component (period=12), and COVID/policy regressors. MinT hierarchical reconciliation is applied to eliminate summation drift across scenarios. The RTGS FY26 range reflects high-value settlement volatility, not forecast instability.*

---

# Bonus Models

### Bonus A: DLinear Neural Decomposition with Exogenous Fusion
The architecture decomposes each input series into a moving-average trend component (Tₜ) and a residual seasonal component (Sₜ), then fuses context features:
* **Context vector:** [ATS_scaled | is_covid | is_policy | is_festive | is_fy_end]
* **Training:** OLS closed-form on log(Value)
* **Confidence intervals:** 500-sample bootstrap
* **Horizon:** 6 months (Apr to Sep 2025)
DLinear P50 projections for Apr to Sep 2025 span ₹24.60T to ₹26.04T, matching the Holt-Winters baseline.

### Bonus B: BSTS and MinT Hierarchical Reconciliation
State-space decomposition per mode includes a stochastic local linear trend, trigonometric seasonal component (period=12), and regression on COVID and policy regressors. MinT reconciliation applies a matrix G = (S’W⁻¹S)⁻¹ S’W⁻¹ to ensure reconciled sub-mode forecasts sum exactly to the aggregate forecast. This eliminates summation drift across scenarios.

---

# Strategic Implications
Three data-grounded strategic recommendations emerge from the full analytical pipeline:

### Recommendation 1: Capture Mid-Ticket ATS Compression via Credit Embedding
* UPI ATS has compressed 12.8% (₹1,553 to ₹1,354) between FY2019 and FY2025.
* UPI value CAGR (62.0%) trails volume CAGR. Growth comes from low-denomination payments rather than high-value transactions.
* Scenario B projects a peak monthly UPI value of ₹33.3T compared to the ₹29.0T baseline. This yields an incremental ₹47.5T annually from Credit-on-UPI.
* **Action:** Embed credit rails in UPI ₹500 to ₹10,000 transactions. Every 1% ATS recovery on UPI’s ₹200T+ annual throughput generates ₹2T in incremental GMV.

### Recommendation 2: Festive Surge Monetization (Oct and Mar Seasonality)
* SARIMA and Prophet both confirm October and November (Diwali) seasonal peaks where UPI volume runs ~18 to 22% above trend.
* Credit cards capture a 2× stronger festive uplift in value (+9.9% vs UPI +4.2%) due to high-ticket gifting spend.
* The forecast model projects an October 2025 UPI value peak at ₹27.26T (Scenario A).
* **Action:** Pre-negotiate merchant-funded EMI-at-POS incentives for the September and October window. Route high-ATS Diwali purchases (electronics, jewellery, travel) through credit rails. Prior festive campaigns demonstrate 12-month behavioral stickiness.

### Recommendation 3: B2B and Cross-Border Corridor Defense
* RTGS maintains the highest ATS of all modes, averaging ₹66L per transaction.
* Credit Card CAGR (25.5%) signals premium-segment robustness. This cohort is structurally insulated from UPI compression.
* BSTS analysis confirms NEFT and RTGS have distinct trend components. B2B flows remain structurally separate from the retail UPI migration.
* **Action:** Target the RTGS-to-real-time SME payment corridor. Cross-border rails provide an uncontested moat. 100% of high-value cross-border flows remain on SWIFT and card networks, completely insulated from domestic UPI disruption. Priority corridors: UAE, US, Singapore (top 3 by inbound FDI volume).

---

# Methodology Notes
All statistical tests use α = 0.05 significance threshold. Welch’s t-test used for mean comparisons (unequal variance). Chow test uses F-statistic with comparison of restricted vs unrestricted model residuals. PELT (Pruned Exact Linear Time) algorithm uses an L2 cost function with β penalty calibrated by BIC.

Forecast confidence intervals are computed analytically for SARIMA and Holt-Winters; via 500-sample bootstrap for DLinear; via MCMC sampling from the posterior for BSTS. All value figures reported in Indian Rupees (INR); T = Trillion (₹1T = ₹100,000 Crore).

---
*The War on Cash • RBI Payment System Analysis • FY2020 to FY2026*
*Source: Reserve Bank of India Payment System Indicators (Table 45) 
