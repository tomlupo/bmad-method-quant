---
stepsCompleted: [vision, universe, data-requirements, risk-constraints, performance-targets, scope]
workflowType: 'research-brief'
project_name: 'Momentum Factor Research'
date: '2026-02-04'
status: 'complete'
---

# Research Brief: Momentum Factor Strategies in US Equities

**Date:** 2026-02-04
**Author:** Quant Researcher (Elena -- Quant Research Agent)
**Phase:** 1 -- Research
**Status:** Complete

---

## Table of Contents

1. [Investment Thesis](#investment-thesis)
2. [Research Motivation](#research-motivation)
3. [Universe Definition](#universe-definition)
4. [Signal Definitions](#signal-definitions)
5. [Data Requirements](#data-requirements)
6. [Risk Constraints](#risk-constraints)
7. [Performance Targets](#performance-targets)
8. [Research Scope & Boundaries](#research-scope--boundaries)
9. [Key Research Questions](#key-research-questions)
10. [Next Steps](#next-steps)

---

## Investment Thesis

**Core Hypothesis:** Cross-sectional and time-series momentum signals in US equities, when combined with dynamic volatility scaling and multi-signal composites, can produce risk-adjusted returns materially superior to unmanaged momentum -- specifically by reducing crash risk (the Achilles heel of traditional momentum) while preserving the core premium.

**Theoretical Basis:** Momentum is one of the most robust and persistent anomalies in empirical asset pricing. Jegadeesh and Titman (1993) established the cross-sectional momentum premium. Moskowitz, Ooi, and Pedersen (2012) documented time-series momentum across asset classes. Barroso and Santa-Clara (2015) demonstrated that volatility scaling nearly doubles the Sharpe ratio of momentum strategies.

**Edge Source:** The alpha derives from behavioral biases (underreaction to information, herding, disposition effect) and institutional frictions (slow capital reallocation, benchmark constraints). Recent research (Baltussen et al., 2026) confirms this premium persists across 46 countries and 150+ years of data, is not a product of data mining, and survives transaction costs when properly managed.

**Key Insight from Recent Literature:** An equal-weighted composite combining price momentum with ten alternative momentum signals demonstrates superior returns and risk-adjusted performance relative to price momentum alone (Baltussen et al., 2026). This multi-signal approach is a primary focus of this research.

---

## Research Motivation

### Why Momentum Now?

1. **2024-2025 Performance:** Momentum was the best-performing equity factor in the US, international developed, and emerging markets in 2024. By year-end 2024, excess returns of US momentum strategies were in the 96th percentile of all periods in the last half century (MSCI).

2. **Crash Risk Mitigation Advances:** Risk-managed momentum (Barroso & Santa-Clara, 2015; Daniel & Moskowitz, 2016) has matured. The Sharpe ratio improves from 0.53 to 0.97 with constant volatility targeting. Maximum drawdown improves from -96.69% to -45.20%.

3. **Multi-Signal Composites:** The factor zoo offers dozens of momentum-adjacent signals (earnings momentum, factor momentum, revenue momentum) that can be combined for more stable performance.

4. **Open-Source Data & Tools:** Kenneth French Data Library, AQR datasets, and Python frameworks (VectorBT, Backtrader) make rigorous momentum research accessible without institutional data subscriptions.

---

## Universe Definition

### Primary Universe

| Parameter | Specification |
|-----------|--------------|
| **Asset Class** | US Equities |
| **Exchange** | NYSE, NASDAQ, AMEX |
| **Market Cap** | > $500M (large + mid cap) |
| **Liquidity** | 20-day ADTV > $5M |
| **Sectors** | All GICS sectors |
| **Exclusions** | ADRs, REITs, SPACs, stocks < $5 |

### Universe Rationale

- **$500M cap floor:** Avoids microcap illiquidity that inflates paper returns but cannot be captured in practice.
- **$5M ADTV floor:** Ensures positions can be built and liquidated without excessive market impact.
- **Price > $5:** Eliminates penny stock noise and bid-ask spread distortion.
- **Estimated universe size:** ~1,500-2,000 stocks.

### Extension Universe (Phase 2)

- International developed markets (MSCI World ex-US)
- Emerging markets (MSCI EM)
- Multi-asset (equity indices, fixed income, commodities, currencies)

---

## Signal Definitions

### Primary Signals

#### 1. Cross-Sectional Price Momentum (XSMOM)
- **Definition:** 12-month cumulative return, skipping the most recent month (12-1 momentum)
- **Construction:** Rank stocks by 12-1 return. Long top quintile, short bottom quintile.
- **Rebalance:** Monthly
- **Source:** Jegadeesh & Titman (1993)

#### 2. Time-Series Momentum (TSMOM)
- **Definition:** Each stock's own 12-month excess return sign determines long/short positioning
- **Construction:** Long if 12-month excess return > 0, short if < 0. Position size proportional to signal strength.
- **Rebalance:** Monthly
- **Source:** Moskowitz, Ooi & Pedersen (2012)

#### 3. Intermediate-Term Momentum (6-1)
- **Definition:** 6-month cumulative return, skipping the most recent month
- **Rebalance:** Monthly
- **Rationale:** Captures faster momentum dynamics; can be blended with 12-1 for intermediate-speed strategies with better skewness properties

### Secondary Signals (Composite)

#### 4. Earnings Momentum (SUE)
- **Definition:** Standardized Unexpected Earnings -- actual EPS minus consensus estimate, normalized by standard deviation of past surprises
- **Source:** Lattimore & Li (2023); shown to be robust across 30 years in US, Europe, Japan

#### 5. Revenue Momentum
- **Definition:** Year-over-year revenue growth rate acceleration
- **Rationale:** "Confirmed momentum" (price + operating momentum) limits downside volatility while capturing trend-following upside (Lord Abbett, 2025)

#### 6. Factor Momentum
- **Definition:** Past returns of equity factors themselves (value, quality, low-vol) predict future factor returns
- **Source:** Ehsani & Linnainmaa (2022) -- factor momentum transmits into the cross-section of stocks

### Risk Management Overlay

#### 7. Volatility-Scaled Momentum
- **Definition:** Scale long-short portfolio exposure inversely proportional to realized volatility over prior 6 months, targeting constant 12% annualized volatility
- **Source:** Barroso & Santa-Clara (2015)
- **Impact:** Sharpe 0.53 -> 0.97; max DD -96.69% -> -45.20%; kurtosis 18.24 -> 2.68

---

## Data Requirements

### Price & Volume Data

| Data Field | Source | Frequency | History |
|-----------|--------|-----------|---------|
| Adjusted close prices | yfinance / Stooq / EODHD | Daily | 2000-present |
| Volume | yfinance / Stooq / EODHD | Daily | 2000-present |
| Market cap | yfinance / FMP | Monthly | 2000-present |
| Sector/industry classification | GICS via data provider | Static + changes | Current |

### Factor Return Data

| Data Field | Source | Frequency | History |
|-----------|--------|-----------|---------|
| Fama-French 3 factors (MKT, SMB, HML) | Kenneth French Data Library | Daily + Monthly | 1926-present |
| Momentum factor (UMD) | Kenneth French Data Library | Daily + Monthly | 1927-present |
| Carhart 4-factor returns | getFamaFrenchFactors (Python) | Monthly | 1927-present |
| AQR Momentum indices (US + intl) | AQR Datasets | Monthly | 1927-present |
| Value & Momentum Everywhere | AQR Datasets | Monthly | 1972-present |

### Fundamental Data (for Earnings/Revenue Momentum)

| Data Field | Source | Frequency | History |
|-----------|--------|-----------|---------|
| EPS actuals & estimates | FMP / Alpha Vantage | Quarterly | 2010-present |
| Revenue actuals & estimates | FMP / Alpha Vantage | Quarterly | 2010-present |
| Earnings announcement dates | FMP | Event-based | 2010-present |

### Benchmark Data

| Benchmark | Source | Frequency |
|-----------|--------|-----------|
| S&P 500 Total Return | yfinance (^SP500TR) | Daily |
| Russell 1000 | yfinance (^RUI) | Daily |
| Risk-free rate (T-bill) | Kenneth French | Daily + Monthly |

### Data Quality Requirements

- **Survivorship bias:** Use delisted stock returns where available; complement with Kenneth French universe which is survivorship-bias-free
- **Point-in-time correctness:** Fundamental data must use original reporting dates, not restated values
- **Look-ahead bias prevention:** All signals computed using only data available at time of portfolio formation
- **Split/dividend adjustment:** Use adjusted close prices consistently
- **Data versioning:** Pin all data downloads with timestamps for reproducibility

---

## Risk Constraints

### Hard Constraints

| Constraint | Limit | Rationale |
|-----------|-------|-----------|
| Maximum drawdown (unmanaged) | Monitoring only | Baseline measurement |
| Maximum drawdown (risk-managed) | -30% | Barroso & Santa-Clara showed -45% achievable; target tighter |
| Single-stock weight | <= 5% of portfolio | Concentration risk |
| Sector exposure | <= 30% net in any sector | Avoid sector bets masquerading as momentum |
| Minimum holding period | 1 month | Align with monthly rebalance; avoid excessive turnover |
| Leverage | <= 2x gross | Practical constraint for risk-managed version |

### Soft Constraints

| Constraint | Target | Notes |
|-----------|--------|-------|
| Annual turnover | < 200% one-way | Momentum is inherently high-turnover |
| Transaction cost budget | < 100 bps round-trip | Covers market impact + commissions |
| Beta to market | 0.0 +/- 0.3 (L/S) | Long-short should be approximately market-neutral |
| Tracking error (long-only vs benchmark) | 3-8% | If long-only variant is constructed |

---

## Performance Targets

### Risk-Adjusted Return Targets

| Metric | Unmanaged Baseline | Risk-Managed Target | Source |
|--------|-------------------|---------------------|--------|
| **Annualized Sharpe Ratio** | 0.5-0.6 | > 0.8 | Barroso & Santa-Clara achieved 0.97 |
| **Information Ratio** | 0.3-0.5 | > 0.5 | Relative to market benchmark |
| **Annualized Alpha (4-factor)** | 5-8% | 5-8% (preserved) | Net of Fama-French + Momentum factors |
| **Maximum Drawdown** | -50% to -90% | < -30% | Vol-scaling primary mechanism |
| **Skewness** | -2.5 (negative) | > -0.5 | Vol-scaling improves from -2.47 to -0.42 |
| **Kurtosis** | > 15 | < 5 | Vol-scaling improves from 18.24 to 2.68 |

### Statistical Significance Requirements

| Test | Threshold | Purpose |
|------|-----------|---------|
| t-statistic on alpha | > 2.0 | Standard significance |
| Deflated Sharpe Ratio | > 0 | Adjusts for multiple testing |
| Walk-Forward Efficiency | > 50% | OOS/IS ratio confirms robustness |
| Out-of-sample Sharpe | > 60% of in-sample | Stability check |

### Alpha Decay Awareness

- Short-term signals (< 1 month): half-life measured in days; high turnover requirement
- Medium-term signals (1-12 months): half-life ~3-6 months; this is our primary signal regime
- **Stale momentum warning:** Stocks with momentum > 24 months are both expensive and tired; stale momentum reverses and destroys value (Research Affiliates)

---

## Research Scope & Boundaries

### In Scope

- Cross-sectional momentum (12-1, 6-1) in US equities
- Time-series momentum in US equities
- Volatility-scaled / risk-managed momentum
- Multi-signal momentum composites (price + earnings + revenue + factor momentum)
- Walk-forward backtesting with out-of-sample validation
- Transaction cost modeling (market impact, slippage, commissions)
- Regime-conditional analysis (bull/bear/crisis)
- Statistical significance testing (t-stats, deflated Sharpe, CPCV)

### Out of Scope (Phase 1)

- International equities (deferred to Phase 2)
- Multi-asset momentum (futures, FX, fixed income)
- Machine learning signal generation (feature engineering only)
- Live trading / execution system
- High-frequency momentum (intraday)
- Options-based momentum strategies

### Key Assumptions

1. We can access daily price data for ~2,000 US equities from 2000-present via free data sources
2. Monthly rebalance frequency is sufficient for medium-term momentum signals
3. 100 bps round-trip transaction cost is conservative for liquid large/mid-cap US equities
4. Kenneth French factor data provides a reliable survivorship-bias-free benchmark

---

## Key Research Questions

### Primary Questions

1. **Does risk-managed momentum (volatility-scaled) materially reduce crash risk in US equities over 2000-2025, consistent with Barroso & Santa-Clara's findings?**
   - Metric: Sharpe ratio improvement, maximum drawdown reduction, skewness improvement

2. **Does a multi-signal composite (price + earnings + factor momentum) outperform single-signal momentum on a risk-adjusted basis?**
   - Metric: Composite vs. individual signal Sharpe ratios, information ratios, and stability

3. **How sensitive are results to portfolio construction choices (quintile vs. decile, equal-weight vs. cap-weight, rebalance frequency)?**
   - Metric: Walk-forward efficiency across construction variants

4. **What is the realistic capacity of these strategies after transaction costs?**
   - Metric: Net-of-cost returns at various AUM levels; breakeven turnover analysis

### Secondary Questions

5. **Do intermediate-speed strategies (blending 6-1 and 12-1) improve skewness and reduce turning-point losses?**
6. **Is factor momentum a distinct alpha source or simply stock momentum in disguise (Ehsani & Linnainmaa debate)?**
7. **How does momentum performance vary across market regimes (high/low volatility, bull/bear)?**

---

## Next Steps

Following the Quant Method Full Research Path:

1. **Domain Research** -- Deep dive into academic momentum literature, factor zoo analysis, and statistical methods
2. **Technical Research** -- Evaluate backtesting frameworks, data pipelines, and implementation architecture
3. **Strategy Design** -- Full signal specification, risk constraints, and performance targets
4. **Architecture** -- Data pipeline, backtesting infrastructure, and monitoring system design
5. **Research Plan** -- Break into prioritized implementation tasks

---

## Sources

- [Baltussen et al. (2026) -- Momentum Factor Investing: Evidence and Evolution](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5561720)
- [Barroso & Santa-Clara (2015) -- Momentum Has Its Moments](https://www.sciencedirect.com/science/article/abs/pii/S0304405X14002566)
- [Moskowitz, Ooi & Pedersen (2012) -- Time Series Momentum](https://www.aqr.com/Insights/Research/Journal-Article/Time-Series-Momentum)
- [Jegadeesh & Titman (1993) -- Returns to Buying Winners and Selling Losers](https://doi.org/10.1111/j.1540-6261.1993.tb04702.x)
- [Daniel & Moskowitz (2016) -- Momentum Crashes](https://www.nber.org/system/files/working_papers/w18169/w18169.pdf)
- [Ehsani & Linnainmaa (2022) -- Factor Momentum and the Momentum Factor](https://www.nber.org/system/files/working_papers/w25551/w25551.pdf)
- [Lord Abbett (2025) -- Price and Operating Momentum in Equity Portfolios](https://www.lordabbett.com/en-us/financial-advisor/insights/investment-objectives/2025/the-benefits-of-price-and-operating-momentum-in-equity-portfolios.html)
- [Mamais (2025) -- Explaining and Predicting Momentum Performance Shifts](https://onlinelibrary.wiley.com/doi/full/10.1002/for.3232)
- [Research Affiliates -- Can Momentum Investing Be Saved?](https://www.researchaffiliates.com/publications/articles/637-can-momentum-investing-be-saved)
- [Kenneth French Data Library](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html)
- [AQR Datasets](https://www.aqr.com/Insights/Datasets)
- [Alpha Architect -- Momentum Factor Investing](https://alphaarchitect.com/momentum-factor-investing/)
- [MSCI -- Factor Indexing Through the Decades](https://www.msci.com/downloads/web/msci-com/research-and-insights/paper/factor-indexing-through-the-decades/factor-indexing-through-the-decades.pdf)
