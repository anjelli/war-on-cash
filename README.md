# The War on Cash: Time-Series Analysis of India's Digital Payment Ecosystem & Forecasting

> A multi-phase analytical pipeline — ETL, EDA, diagnostic econometrics, and forecasting — applied to India's RBI payment systems data to model structural payment transitions and predict FY2026 transaction behavior across six payment rails.

---

## Executive Summary

India's digital payment ecosystem has undergone a structural reconfiguration, not merely a cyclical growth phase. Between November 2019 and March 2025, UPI grew from a nascent rail to the dominant retail payment instrument, accounting for 89.4% of total transaction volume by FY2025. This transition is irreversible by any standard metric: volume share is monotonically increasing, debit card displacement is continuous, and two confirmed structural breaks — COVID-19 (2020) and the RBI's PPI interchange cap (April 2023) — have permanently altered mode-level trajectories.

This project models that transition end-to-end. Using 65 months of RBI Payment System Indicators data, the pipeline progresses from raw pivot-table ingestion through feature engineering, correlation and seasonality analysis, econometric break detection, Granger causality testing, and a multi-model forecasting competition — culminating in scenario-based projections for FY2026.

The strategic implications extend beyond volume statistics. UPI dominates by transaction count (40× credit cards) but generates zero MDR revenue for card networks. Credit cards, by contrast, command a 3.2× average ticket size premium over UPI (₹4,393 vs ₹1,354 as of March 2025), a gap that is widening. The convergence thesis — Credit-on-UPI as the structural bridge between network monetization and UPI's distribution — is modeled as Scenario B and projects a ₹47.5T annual incremental value uplift.

---

## Core Research Questions

- Is UPI structurally displacing traditional rails, or are these markets parallel?
- Are debit cards in secular decline, and is that decline causally linked to UPI adoption?
- How do regulatory shocks — Zero-MDR (2020), PPI Interchange Cap (2023) — reshape payment flow distributions?
- What does India's digital payment mix look like in FY2026 under baseline and credit-expansion scenarios?
- Does UPI growth Granger-cause volume shifts in adjacent modes (IMPS, NEFT, Debit)?
- Where does Visa's revenue moat lie in a UPI-dominant ecosystem?

---

## Dataset

**Source:** RBI Payment System Indicators — Table 45 (publicly available)  
**Frequency:** Monthly  
**Coverage:** November 2019 – March 2025 (65 months)  
**Rows after cleaning:** 390 (long-format panel)

**Payment modes covered:**

| Mode | Description |
|------|-------------|
| UPI | Unified Payments Interface — retail P2P and P2M |
| IMPS | Immediate Payment Service — bank-to-bank, 24×7 |
| NEFT | National Electronic Funds Transfer — batch settlement |
| RTGS | Real Time Gross Settlement — high-value institutional |
| Credit Cards | Card network transactions (Visa, Mastercard, RuPay) |
| Debit Cards | ATM proxy + POS (structural limitations noted) |

**Engineered features:**

| Feature | Description |
|---------|-------------|
| `ATS` | Average Ticket Size = Value / Volume |
| `MoM_Growth` | Month-on-month % change |
| `YoY_Growth` | Year-on-year % change |
| `log_volume`, `log_value` | Log-transformed series for modeling stability |
| `is_covid_shock` | Binary flag for lockdown window (Mar–Jun 2020) |
| `is_policy_change` | Flags for Zero-MDR (Jan 2020) and PPI Cap (Apr 2023) |
| `is_festive_month` | Oct–Nov indicator for Diwali seasonality |
| `is_fy_end` | March fiscal year-end flag |
| `MoM_Anomaly` | Z-score based outlier flag |

The raw data used a multi-row pivot header structure with schema inconsistencies across periods. Parsing relied on deterministic column indexing rather than header-based inference.

---

## Project Pipeline

### Phase 1 — ETL & Data Engineering

The RBI dataset was not analysis-ready. The ingestion pipeline addressed:

- **Pivot flattening:** Multi-row headers collapsed into a strict long-format panel `(Date, Payment_Mode, Volume, Value)`
- **Deterministic parsing:** Column positions indexed programmatically to handle schema drift across reporting periods
- **Schema validation:** Dynamic keyword validation against expected mode labels
- **Standardization:** All value fields normalized to consistent units (₹ Crore)

After cleaning: 390 rows, 6 modes, November 2019 – March 2025.

---

### Phase 2 — Exploratory Analysis

**UPI vs. Debit Card divergence (log-scale)**

On a log scale where equal slopes indicate equal growth rates, UPI's trajectory is visibly steeper than every peer across the full period. Debit card volume — used here as an ATM/cash proxy — plateaued after 2020 and has not recovered its pre-lockdown growth slope. The correlation matrix confirms this: UPI and debit carry a −0.91 Pearson correlation across the full sample, the strongest negative relationship in the dataset.

**ATS divergence**

The most strategically significant finding in Phase 2: UPI's average ticket size has been declining (₹1,800 → ₹1,354), while credit card ATS has been rising (₹3,000 → ₹4,393). These trends are moving in opposite directions simultaneously, signaling behavioral market segmentation rather than direct competition. UPI is absorbing mass-market, low-denomination transactions. Cards are concentrating on premium, high-value spend.

**Seasonality**

Monthly boxplots confirm a persistent October–November peak across both UPI and credit cards, consistent with Diwali demand seasonality. The festive uplift is approximately +4.2% for UPI and +9.9% for credit cards — credit's stronger response reflecting higher-ticket gifting, electronics, and travel purchases.

**Correlation structure**

UPI, NEFT, and RTGS are positively correlated (r > 0.90), reflecting shared macro tailwinds. Debit's strongly negative correlation with digital modes confirms it as the primary substitution target, not a complementary rail.

---

### Phase 3 — Diagnostic Analysis

#### Timeline and Structural Narrative

Three inflection points define the dataset:

- **April 2020 (Full Lockdown):** Sharp simultaneous drop across all modes. UPI recovered to trend within 3 months. Debit never resumed its prior growth slope.
- **Post-unlock 2021:** UPI re-accelerated. NEFT, IMPS, and cards did not match its rate.
- **April 2023 (PPI Interchange Cap):** Visible kink in IMPS (positive) and Credit Card (negative) trajectories — the clearest policy-driven flow shift in the dataset.

**What this means strategically:** The 2020 lockdown was not the cause of UPI's dominance — it was the accelerant. Essential payments (utilities, groceries, P2P transfers) ran on UPI even when physical infrastructure shut. Behavioral lock-in happened during that window, and the subsequent trajectory confirms permanence.

#### COVID-19 Impact Analysis

Trend-corrected Δ% analysis (Welch t-tests, ±6-month window, α = 0.05):

| Mode | Full Lockdown Impact | Recovery |
|------|---------------------|----------|
| Credit Card | Largest negative Δ% (statistically significant) | Slow — retail and travel collapsed |
| Debit Card | Significant negative Δ% | Recovered but never regained slope |
| UPI | Smallest negative Δ% | Back to trend within ~3 months |
| NEFT | Moderate negative | Gradual — institutional use case |

UPI's resilience was structural, not random. Its digital-native architecture had no physical dependency, and its use case (P2P, essential retail) was non-discretionary during lockdown.

#### Interchange Regulation Impact (PPI Cap, April 2023)

The April 2023 RBI cap on PPI interchange fees is the most statistically clean policy event in the dataset. IMPS shows a significant positive trend-corrected deviation post-cap; Credit Card shows a corresponding negative deviation. Mechanism: wallet providers lost commercial incentive to route via PPI rails, migrating flow to bank-account-based IMPS. Since wallets were funded by cards, the cap indirectly softened card volumes as well.

The Zero-MDR mandate (January 2020) produced directionally consistent signals but is statistically inconclusive — COVID arrived two months later and contaminated the post-event window.

**What this means strategically:** Regulatory intervention in India does not merely affect the targeted mode — it reshapes the entire flow distribution. Any fintech strategy that ignores policy risk in Indian payments is incomplete.

#### Structural Break Detection (Chow + PELT)

Two system-wide break regimes are confirmed by independent methods:

- **Chow tests** (hypothesis-based, event-date anchored): Statistically significant breaks at known events
- **PELT** (data-driven, no prior assumptions): Detects additional change-points without specifying dates

Results per mode:
- **UPI & IMPS:** Multiple breaks — post-COVID acceleration and a clear regime shift at April 2023
- **NEFT & Credit:** Strong COVID-related breaks (March–April 2020), followed by gradual adjustments
- **RTGS:** Smooth structural evolution; weaker shock sensitivity (institutional inertia)
- **Debit:** No sharp COVID break; shows steady structural decline with later-stage shifts

The ecosystem exhibits a multi-regime structure: initial disruption (2020), stabilization (2021–22), and broad structural reconfiguration (2023 onward).

#### Granger Causality

Granger analysis tests whether lagged UPI values improve forecasts of other modes beyond those modes' own history (predictive directionality, not causation):

- **UPI → IMPS:** Significant at multiple lags across full sample. Economically logical — UPI rides the IMPS inter-bank rail, so UPI volume provides a short-term signal for IMPS activity.
- **UPI → Debit (ATM proxy):** Weak and inconsistent. POS/online noise in the debit series obscures cash-displacement signal.
- **UPI → NEFT:** Present at short lags in the full sample; weakens post-2023. Likely reflects shared macro drivers rather than direct substitution.

The UPI–IMPS Granger relationship weakens post-2023, consistent with the structural decoupling detected by PELT. This was incorporated into Phase 4 model design — UPI was treated as a standalone forecast target rather than a component of a joint system.

#### FY2025 Market Share

By FY2025, UPI alone crosses 80% of total retail transaction volume (89.4% by the final observation). The top three modes — UPI, NEFT, IMPS — collectively account for 96.8% of all retail volume. Card share has declined in relative terms every fiscal year since FY2020, even as absolute card volumes grow.

---

### Phase 4 — Forecasting

**What this means:** The forecasting phase does not merely project UPI growth. It models the pace of India's transition to a UPI-centric payment architecture and quantifies the value implications of Credit-on-UPI adoption — a policy question with direct revenue consequences for global card networks.

#### Experimental Design

| Window | Period |
|--------|--------|
| Training | November 2019 – December 2022 (37 months) |
| Test (hold-out) | January 2023 – December 2024 (24 months) |
| Forecast | April 2025 – March 2026 (12 months, FY2026) |

The 24-month hold-out was selected to span the post-PPI Cap regime, ensuring model performance is evaluated on structurally shifted data — a conservative and realistic test of out-of-sample validity.

#### Stationarity Analysis (ADF + KPSS)

Both Augmented Dickey-Fuller (ADF) and KPSS tests were applied at d=0, d=1, and d=2 to determine differencing order for SARIMA:

| Mode | Verdict |
|------|---------|
| UPI | d=2 (double-integrated; accelerating growth trend) |
| NEFT, Credit, Debit, IMPS, RTGS | d=1 |

UPI required second-order differencing — reflecting not just a trend but an accelerating trend. This informed SARIMA order selection and confirmed that additive specifications (which assume linear trend) are appropriate for the post-differenced series.

ACF/PACF analysis of log(UPI Volume): significant autocorrelation at seasonal lag s=12 confirmed annual periodicity. PACF decay guided AR order selection for SARIMA.

#### Baseline Benchmark

A zero-parameter naive seasonal model (ŷ(t) = y(t − 12)) established the minimum performance threshold:

- **Naive MAPE: 40.21%** — all candidate models must clear this bar

#### Model 1: SARIMA

Auto-selected via stepwise AIC search over the full SARIMA order space.

- **Selected order:** SARIMA(0, 2, 1)×(0, 1, 0, 12)
- **AIC (train):** −27.05
- **Ljung-Box residual test:** p = 0.568 at lag 10 — residuals are white noise ✓
- **Test MAPE:** 5.78% | RMSE: ₹1,463.7 Bn | MAE: ₹1,142.9 Bn
- **FY26 production range:** ₹23.84T – ₹26.90T/month

SARIMA's strength here is parsimony — the (0,2,1) specification captures the double-integrated trend cleanly while the seasonal differencing handles annual periodicity without overparameterization. Residual diagnostics confirm a well-specified model.

#### Model 2: Holt-Winters ETS(A,A,A) — Primary Model

UPI exhibits near-linear value growth after second-order differencing, making an additive error, additive trend, additive seasonal specification appropriate. Multiplicative ETS collapsed (alpha → 1.0, convergence failure on the short training window).

**Smoothing parameters:**
- α (level) = 0.9722 — high responsiveness to recent observations
- β (trend) = 0.1342 — moderate trend persistence
- γ (seasonal) = 0.0000 — stable seasonal pattern, no updating required

- **Test MAPE: 3.45%** | RMSE: ₹850.7 Bn | MAE: ₹658.9 Bn
- **FY26 production range:** ₹24.07T – ₹28.99T/month

The near-zero gamma parameter is informative: UPI's seasonal pattern has remained structurally stable despite dramatic level changes, validating the additive seasonal assumption and eliminating the need for seasonal parameter updates during the forecast horizon.

#### Model 3: Prophet

Tuned with `changepoint_prior_scale = 0.3` on a validation window. Regressors: `is_covid_shock`, `is_policy_change`. Seasonality mode: multiplicative.

- **Test MAPE: 10.01%** | RMSE: ₹2,382.1 Bn | MAE: ₹1,865.0 Bn
- **FY26 range:** ₹24.50T – ₹29.65T/month

Prophet underperformed relative to SARIMA and Holt-Winters. The likely cause: multiplicative seasonality imposed by default interacts poorly with UPI's second-order trend, and the changepoint mechanism introduced flexibility where structural stability was more appropriate. Prophet's explicit regressor framework adds value in event-heavy datasets, but the RBI series is better characterized by trend dominance than event punctuation after controlling for known breaks.

#### Model Comparison (24-Month Hold-Out)

| Rank | Model | MAPE | RMSE (₹ Cr) | MAE (₹ Cr) | Beats Naive? |
|------|-------|------|------------|-----------|-------------|
| 1 | **Holt-Winters ETS(A,A,A)** | **3.45%** | 85,074 | 65,891 | ✓ |
| 2 | SARIMA(0, 2, 1)×(0, 1, 0, 12) | 5.78% | 1,46,372 | 1,14,295 | ✓ |
| 3 | Prophet (COVID + Policy regressors) | 10.01% | 2,38,215 | 1,86,504 | ✓ |
| 4 | Naive Seasonal (baseline) | 40.21% | 7,88,531 | 7,40,882 | — |

**Primary model: Holt-Winters ETS(A,A,A) — MAPE 3.45% on 24-month hold-out**

A 3.45% MAPE on a 24-month out-of-sample test spanning a structural regime change (post-PPI Cap) is a strong result. It suggests the additive trend-seasonal decomposition is capturing the genuine underlying data-generating process rather than overfitting to the training window.

#### Scenario Modeling (FY2026)

**Scenario A — Baseline CAGR:** Holt-Winters ETS(A,A,A) production forecast, trained on full dataset (November 2019 – March 2025). Captures historical CAGR and seasonal structure with no additional assumption.

**Scenario B — Credit-on-UPI Uplift:** Scenario A × 1.15. Models a +15% value uplift from Credit-on-UPI adoption expanding into mid-ticket transactions (₹500–₹10,000 range). Grounded in the observed ATS compression and the nascent RuPay Credit Card on UPI product.

| Month | Scenario A (₹T) | 95% CI | Scenario B (₹T) |
|-------|----------------|--------|----------------|
| Apr 2025 | 24.07 | 23.23 – 24.90 | 27.68 |
| Jul 2025 | 25.36 | 23.69 – 27.02 | 29.16 |
| Oct 2025 | 27.26 | 25.06 – 29.46 | 31.35 |
| Dec 2025 | 27.70 | 25.20 – 30.19 | 31.85 |
| Mar 2026 | **28.99** | 26.10 – 31.87 | **33.33** |

**12-month Scenario B uplift over Scenario A: ₹47.5T**

**What this means:** The ₹47.5T incremental value in Scenario B is not speculative — it quantifies the monetizable delta from Credit-on-UPI adoption at current UPI throughput levels. At a conservative 0.3% MDR, this implies approximately ₹1,400 Bn in potential network fee revenue that currently generates zero income for card networks. This is the strategic opportunity Credit-on-UPI represents for Visa and Mastercard.

#### All-Mode SARIMA Forecasts (FY2026)

| Mode | SARIMA Order | FY26 Monthly Range |
|------|-------------|-------------------|
| UPI | (0, 2, 1)×(0, 1, 0, 12) | ₹23.84T – ₹26.90T |
| NEFT | (0, 1, 2)×(0, 1, 0, 12) | ₹37.02T – ₹53.71T |
| Credit | (0, 1, 2)×(0, 1, 0, 12) | ₹1.85T – ₹2.35T |
| Debit | (0, 1, 2)×(0, 1, 0, 12) | ₹0.24T – ₹0.35T |
| IMPS | (2, 1, 0)×(0, 1, 0, 12) | ₹5.59T – ₹6.59T |
| RTGS | (0, 1, 1)×(0, 1, 0, 12) | ₹169.18T – ₹260.45T |

NEFT and RTGS ranges are wide by design — both modes carry inherent settlement volatility driven by institutional behavior that is difficult to forecast at monthly granularity. The Debit range confirms structural decline continuation into FY2026.

---

### Advanced Modeling

#### DLinear — Neural Decomposition with Exogenous Fusion

Architecture: Explicit trend-seasonal decomposition where X_t = T_t (moving-average trend) + S_t (residual). Both components are projected via separate linear heads and fused with a context vector containing ATS, COVID, policy, festive, and fiscal year-end signals.

Training via OLS closed-form on log(Value); confidence intervals via 500-sample bootstrap. Horizon: April – September 2025 (6-month).

| Month | P50 Forecast (₹T) |
|-------|------------------|
| Apr 2025 | 25.07 |
| Jun 2025 | 24.85 |
| Sep 2025 | 26.04 |

DLinear's exogenous fusion is its primary value-add: unlike SARIMA or Holt-Winters, it explicitly incorporates policy regime and behavioral signals as inputs. The short horizon reflects the bootstrap confidence interval's rapid expansion beyond 6 months.

#### BSTS — Bayesian Structural Time Series

State-space decomposition per mode:

```
y_t = μ_t (local linear trend, stochastic)
    + τ_t (trigonometric seasonal, period = 12)
    + β·X_t (regressors: COVID shock, policy change)
    + ε_t
```

BSTS outputs posterior credible intervals rather than frequentist confidence intervals, providing probabilistic forecast bands that better represent genuine parameter uncertainty. AIC values confirm model fit is substantially better than naive baselines for all modes.

BSTS FY2026 ranges: UPI ₹24.29T–₹32.30T, NEFT ₹33.98T–₹54.80T, RTGS ₹157.02T–₹234.22T.

#### MinT Hierarchical Reconciliation

Applied post-BSTS to eliminate summation drift across modes. The MinT estimator computes a reconciliation matrix G = (S′W⁻¹S)⁻¹S′W⁻¹ from the residual covariance matrix W, ensuring that reconciled sub-mode forecasts sum to the aggregate forecast. This is critical when scenarios are compared across modes — naive aggregation of independently-fit models produces inconsistent totals.

---

## Key Findings

**1. UPI dominates volume, not monetization.** UPI processes 40× credit card transaction volume but generates zero MDR revenue for card networks. The aggregate value figure (12.3× credit cards) is misleading — it is dominated by low-denomination P2P transfers at ₹1,354 average ticket size. The monetizable layer of India's payment ecosystem remains card-native.

**2. ATS divergence is the most strategically informative trend.** UPI ATS has compressed −12.8% since November 2019. Credit card ATS has expanded +28.7% over the same period, now commanding a 3.2× premium. The gap is widening. This signals natural market segmentation: UPI absorbs mass-market frequency, cards retain premium-spend stickiness. Card networks operating on MDR are therefore not in direct volume competition with UPI — they occupy structurally distinct transaction segments.

**3. Credit-on-UPI is the convergence point.** The only mechanism by which card networks can participate in UPI's scale is credit rail embedding — RuPay Credit on UPI QR. Scenario B quantifies this at ₹47.5T annual incremental value, implying ~₹1,400 Bn in potential network revenue at prevailing MDR rates.

**4. The 2023 PPI Cap created durable flow shifts.** IMPS volumes shifted upward post-April 2023 as wallet providers migrated away from PPI rails. Credit card volumes softened. Both effects persisted with 6+ month half-life. Regulatory events in India produce structural, not transient, behavioral changes.

**5. COVID accelerated behavioral lock-in.** UPI's lockdown resilience was functional, not coincidental — essential payments did not require physical infrastructure. The behavioral habit established during lockdown explains why UPI's post-unlock trajectory is steeper than its pre-COVID trend, not a reversion to it.

**6. RTGS still dominates enterprise settlement value.** Average ticket size hierarchy by March 2025: RTGS (~₹66L/txn) >> NEFT >> IMPS >> Credit >> Debit >> UPI. Enterprise B2B payment value remains entirely on RTGS/NEFT rails. UPI disruption is a retail phenomenon; institutional settlement is structurally separate and insulated.

**7. Debit cards are in structural, not cyclical, decline.** Volume share has declined every fiscal year since FY2020 with no reversal. This is not a COVID artifact — the post-unlock recovery failed to restore the pre-COVID slope. UPI substitution at point-of-sale is the mechanism, confirmed by the −0.91 cross-mode correlation.

---

## Dashboard — Power BI Visualization Layer

Four dashboards translate analytical outputs into business-consumable views:

**Executive View** — Headline KPIs (Total Digital Value, Volume, UPI Value Share %) with retail and enterprise payment trend panels segmented by fiscal year and period flags (COVID, Festive, FY-End, Policy Change).

**Category War** — Multi-mode value share evolution from FY2020–FY2025. Annotated with the structural narrative: UPI dominates retail scale, RTGS controls enterprise value, cards are premiumizing.

**Ticket Size Tracker** — ATS time series (UPI vs. Credit Cards), structural divergence scatter, ATS hierarchy by rail (log-scale), and YoY ATS heatmap. Designed to communicate Visa's structural moat: the widening ATS gap is not closing.

**Scenario Analysis** — Forecast overlays (Scenario A vs. B), model comparison on test set, and mode-level SARIMA panels for FY2026. Enables interactive scenario navigation by fiscal year and payment mode.

---

## Forecasting Results

### UPI Value — 24-Month Hold-Out (Jan 2023 – Dec 2024)

| Rank | Model | MAPE | RMSE (₹ Cr) | MAE (₹ Cr) |
|------|-------|------|------------|-----------|
| 1 | Holt-Winters ETS(A,A,A) | **3.45%** | 85,074 | 65,891 |
| 2 | SARIMA(0, 2, 1)×(0, 1, 0, 12) | 5.78% | 1,46,372 | 1,14,295 |
| 3 | Prophet (COVID + Policy regressors) | 10.01% | 2,38,215 | 1,86,504 |
| 4 | Naive Seasonal (benchmark) | 40.21% | 7,88,531 | 7,40,882 |

### FY2026 UPI Scenario Projections

| Scenario | Monthly Range | Peak Month | Peak Value |
|----------|--------------|------------|-----------|
| A — Baseline CAGR | ₹24.07T – ₹28.99T | March 2026 | ₹28.99T |
| B — Credit-on-UPI (+15%) | ₹27.68T – ₹33.33T | March 2026 | ₹33.33T |
| Scenario B – A uplift | — | — | **₹47.5T (12-month cumulative)** |

---

## Future Directions

- **Sequential deep learning:** LSTM or Temporal Fusion Transformer for multi-mode joint forecasting with shared latent structure
- **Real-time monitoring:** Streaming pipeline on RBI monthly releases with automated anomaly alerts
- **Macroeconomic covariates:** GDP growth, CPI, repo rate, smartphone penetration as external regressors
- **Merchant segmentation:** Disaggregation of UPI P2M by merchant category to isolate high-ATS segments
- **Graph-based transaction modeling:** Network analysis of inter-mode flow migration following policy shocks
- **Causal inference:** Synthetic control methods to isolate the true policy effect of Zero-MDR and PPI Cap, independent of concurrent macro trends
- **Cross-border extension:** Integrating SWIFT/RDA data to model RTGS in the context of India's export-driven FDI corridors

---

## Conclusion

India's payment ecosystem is not converging on a single rail — it is stratifying across rails with distinct behavioral, economic, and regulatory characteristics. UPI has structurally captured the retail mass market. RTGS remains the uncontested backbone of enterprise value settlement. Credit cards are premiumizing in response to UPI pressure, concentrating on high-ATS discretionary spend where network economics are sustainable.

The forecasting evidence supports a clear trajectory: UPI transaction value will reach approximately ₹29T/month by March 2026 under baseline conditions, with an additional ₹47.5T annual potential unlocked by Credit-on-UPI adoption. The structural break analysis confirms that this transition is not cyclical — two independent regime shifts have permanently altered the payment mix, and the Granger evidence suggests UPI growth is itself a leading signal for adjacent rail behavior.

For card networks and fintechs operating in India, the strategic question is not whether UPI displaces cards — it does not, at the segment level that generates MDR revenue. The question is whether credit embedding on UPI infrastructure can be captured before an NPCI-native credit product forecloses that window. The data makes the urgency of that question quantitatively explicit.

---

*Data source: Reserve Bank of India — Payment System Indicators, Table 45. Analysis covers November 2019 – March 2025.*
