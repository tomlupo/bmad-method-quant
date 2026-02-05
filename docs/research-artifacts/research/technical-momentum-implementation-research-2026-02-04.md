---
stepsCompleted: [init, technology-stack, integration-patterns, architectural-patterns, implementation-research, research-synthesis]
inputDocuments: [research-brief-momentum-factors-2026-02-04.md]
workflowType: 'research'
lastStep: 6
research_type: 'technical'
research_topic: 'Momentum Factor Implementation'
research_goals: 'Evaluate backtesting frameworks, data pipelines, and implementation architecture for momentum strategies'
date: '2026-02-04'
web_research_enabled: true
source_verification: true
---

# Technical Research: Momentum Factor Strategy Implementation

**Date:** 2026-02-04
**Author:** Quant Researcher (Elena -- Quant Research Agent)
**Research Type:** Technical
**Phase:** 1 -- Research

---

## Executive Summary

This technical research evaluates the implementation landscape for momentum factor strategies in US equities, covering backtesting frameworks, data pipeline architectures, validation methodologies, and production deployment patterns. The key finding is that a VectorBT-based backtesting core, combined with Kenneth French and yfinance data pipelines, walk-forward validation, and volatility-scaling risk management, provides the optimal balance of speed, rigor, and accessibility for this research project. The total estimated data cost is $0 using open-source data sources.

---

## Table of Contents

1. [Backtesting Framework Evaluation](#1-backtesting-framework-evaluation)
2. [Data Pipeline Architecture](#2-data-pipeline-architecture)
3. [Signal Computation Engine](#3-signal-computation-engine)
4. [Validation & Anti-Overfitting Framework](#4-validation--anti-overfitting-framework)
5. [Risk Management Implementation](#5-risk-management-implementation)
6. [Portfolio Construction Engine](#6-portfolio-construction-engine)
7. [Performance Attribution](#7-performance-attribution)
8. [Recommended Technology Stack](#8-recommended-technology-stack)
9. [Architecture Decision Records](#9-architecture-decision-records)
10. [Implementation Roadmap](#10-implementation-roadmap)

---

## 1. Backtesting Framework Evaluation

### Framework Comparison

Based on comprehensive evaluation of the major Python backtesting frameworks available in 2026:

| Framework | Speed | Ease of Use | Live Trading | Factor Research | Best For |
|-----------|-------|-------------|--------------|-----------------|----------|
| **VectorBT** | Fastest (NumPy/Numba) | Intermediate | Via add-ons (StrateQueue) | Excellent | Large-scale quant research |
| **Backtrader** | Moderate | Easiest | Built-in (IB, Oanda) | Good | Swing trading, event-driven |
| **Zipline** | Slow | Hard setup (Python 3.5-3.6 legacy) | Limited (Zipline-Reloaded) | Excellent (Quantopian heritage) | Academic factor research |
| **Backtesting.py** | Fast | Very easy | No | Basic | Quick experiments |
| **QSTrader** | Moderate | Intermediate | Simulated broker | Good | Clean architecture |

**Sources:**
- [Battle-Tested Backtesters: VectorBT, Zipline, Backtrader (Medium)](https://medium.com/@trading.dude/battle-tested-backtesters-comparing-vectorbt-zipline-and-backtrader-for-financial-strategy-dee33d33a9e0)
- [Python Backtesting Frameworks: Six Options (Pipekit)](https://pipekit.io/blog/python-backtesting-frameworks-six-options-to-consider)
- [Top 21 Python Trading Tools (Analyzing Alpha, Jan 2026)](https://analyzingalpha.com/python-trading-tools)

### Recommendation: VectorBT as Primary Engine

**Rationale:**
1. **Speed:** Fully vectorized operations with Numba JIT compilation -- critical for running thousands of walk-forward windows across 2,000 stocks over 25 years of daily data
2. **Portfolio-level analysis:** Native support for multi-asset portfolios, factor decomposition, and custom metrics
3. **Jupyter integration:** Interactive Plotly dashboards for exploratory research
4. **Extensibility:** Custom signal functions, risk metrics, and portfolio construction rules
5. **Active development:** Most actively maintained of the major frameworks in 2025-2026

**Limitations to address:**
- No native live trading (mitigated by StrateQueue bridge or custom execution layer)
- Steep learning curve for NumPy-centric API (mitigated by wrapper functions)

### Secondary: Alphalens for Factor Analysis

Quantopian's Alphalens library provides specialized factor performance analysis:
- Information coefficient (IC) computation
- Quantile return spreads
- Turnover analysis
- Factor-weighted return decomposition

**Source:** [Alphalens (GitHub)](https://github.com/quantopian/alphalens)

---

## 2. Data Pipeline Architecture

### Data Source Hierarchy

#### Tier 1: Free Academic Factor Data (Primary)

**Kenneth French Data Library**
- Monthly and daily momentum factor (UMD) returns since 1927
- Fama-French 3-factor and 5-factor model returns
- 6 size/momentum sorted portfolios (2x3)
- Construction: "They use six value-weight portfolios formed on size and prior (2-12) returns. The monthly size breakpoint is the median NYSE market equity. The monthly prior (2-12) return breakpoints are the 30th and 70th NYSE percentiles."
- **Access:** Direct CSV download or `getFamaFrenchFactors` Python package
- **Source:** [Kenneth French Data Library](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html)

**AQR Capital Datasets**
- Momentum indices for US and 23 international markets
- "Value and Momentum Everywhere" dataset (1972-present)
- Betting Against Beta factors
- **Source:** [AQR Datasets](https://www.aqr.com/Insights/Datasets)

**Open Source Asset Pricing**
- Hundreds of replicated asset pricing factors with open-source code
- Highly consistent with Ken French's data
- **Source:** [Open Source Asset Pricing](https://www.openassetpricing.com)

#### Tier 2: Free Price Data (for Custom Signal Construction)

**yfinance**
- Daily OHLCV for US equities
- Market cap, sector classification
- Free, no API key required
- **Risk:** Scrapes Yahoo Finance backend; fragile to HTML changes
- **Mitigation:** Cache all downloaded data locally with timestamps

**Stooq**
- Unlimited CSV downloads of historical data
- Good for researchers needing bulk historical data

**Polygon.io (free tier)**
- 5 API calls/minute on free plan
- Delayed data suitable for backtesting

#### Tier 3: Fundamental Data (for Earnings/Revenue Momentum)

**Financial Modeling Prep (FMP)**
- EPS actuals and estimates
- Revenue data
- Earnings announcement dates
- Free tier: 250 API calls/day

**Alpha Vantage**
- Income statements, balance sheets
- Free tier: 25 API calls/day (sufficient for quarterly data)

### Data Pipeline Design

```
[Data Sources] --> [Raw Data Store] --> [Clean Data Store] --> [Feature Store] --> [Signal Store]
     |                   |                    |                    |                   |
  yfinance          Parquet files       Point-in-time        Momentum signals    Portfolio weights
  French Data       with timestamps     adjusted data        Factor exposures    Rebalance dates
  AQR Data          Version control     Split-adjusted       Composite scores    Transaction logs
  FMP API
```

### Data Quality Controls

1. **Survivorship bias:** Use Kenneth French universe (survivorship-bias-free) as primary validation; supplement with delisted returns from CRSP-equivalent sources
2. **Point-in-time correctness:** Store raw data with `as_of_date` timestamps; never use restated financials
3. **Look-ahead bias prevention:** All feature computation uses `shift(1)` or equivalent to ensure signals use only prior-period data
4. **Split/dividend adjustment:** Consistent use of adjusted close prices; validate against known splits
5. **Data versioning:** All raw data downloads tagged with `download_date`; Parquet files stored with schema versioning

### Storage Format

**Parquet** (columnar format):
- Efficient for time-series queries (column pruning)
- Native pandas/PyArrow integration
- Compression reduces storage 5-10x vs CSV
- Schema enforcement prevents data corruption

---

## 3. Signal Computation Engine

### Signal Pipeline Architecture

```python
# Pseudocode for signal computation pipeline
class MomentumSignalEngine:
    """Computes momentum signals with strict look-ahead bias prevention."""

    def cross_sectional_momentum(self, prices, lookback=252, skip=21):
        """12-1 momentum: 12-month return, skip most recent month."""
        returns = prices.pct_change(lookback) / prices.shift(skip).pct_change(skip)
        # Rank within cross-section at each date
        ranks = returns.rank(axis=1, pct=True)
        return ranks

    def time_series_momentum(self, prices, lookback=252, risk_free=None):
        """TSMOM: sign of own excess return over lookback period."""
        excess_ret = prices.pct_change(lookback)
        if risk_free is not None:
            excess_ret -= risk_free.rolling(lookback).sum()
        signal = np.sign(excess_ret)
        return signal

    def volatility_scale(self, returns, target_vol=0.12, lookback=126):
        """Barroso & Santa-Clara volatility scaling."""
        realized_vol = returns.rolling(lookback).std() * np.sqrt(252)
        scale_factor = target_vol / realized_vol
        return scale_factor.clip(0.1, 3.0)  # Leverage bounds
```

### Critical Implementation Rules

1. **No future data in signal computation:** Every signal must be computed using only data available at time `t-1` for portfolio formation at time `t`
2. **Vectorized operations must respect temporal ordering:** Use `.shift()` consistently; never use `.rolling()` with `center=True`
3. **Random seeds:** All stochastic operations (bootstrap, random sampling) must use fixed seeds for reproducibility
4. **NaN handling:** Forward-fill only within stock's listing period; never cross-stock fill

---

## 4. Validation & Anti-Overfitting Framework

### Walk-Forward Analysis (WFA)

Walk-forward optimization is now widely considered the "gold standard" in trading strategy validation (Pardo, 1992/2008). Our implementation:

**Rolling Window Design:**
```
|--- Train (5 years) ---|--- Test (1 year) ---|
                |--- Train (5 years) ---|--- Test (1 year) ---|
                                |--- Train (5 years) ---|--- Test (1 year) ---|
```

- **In-sample window:** 5 years (1,260 trading days)
- **Out-of-sample window:** 1 year (252 trading days)
- **Step size:** 1 year (rolling, not expanding)
- **Total windows:** ~20 (2000-2025)

**Walk-Forward Efficiency (WFE) Metric:**
WFE = (Annualized OOS Return) / (Annualized IS Return)
- WFE > 50-60%: Genuine robustness
- WFE < 30%: Likely overfitting

**Sources:**
- [QuantInsti -- Walk-Forward Optimization Guide (2025)](https://blog.quantinsti.com/walk-forward-optimization-introduction/)
- [Interactive Brokers -- Deep Dive into WFA (2025)](https://www.interactivebrokers.com/campus/ibkr-quant-news/the-future-of-backtesting-a-deep-dive-into-walk-forward-analysis/)

### Combinatorial Purged Cross-Validation (CPCV)

For advanced validation, CPCV shows "marked superiority in mitigating overfitting risks, outperforming traditional methods as evidenced by its lower Probability of Backtest Overfitting (PBO) and superior Deflated Sharpe Ratio (DSR)" compared to simple walk-forward.

**Implementation:**
- Use `mlfinlab` or custom implementation
- Purge gap: 5 trading days between train/test to prevent information leakage
- Embargo: 1 trading day after test period

**Source:** [Backtest Overfitting in the ML Era (ScienceDirect, 2024)](https://www.sciencedirect.com/science/article/abs/pii/S0950705124011110)

### Multiple Testing Correction

**Deflated Sharpe Ratio (DSR):**
- Adjusts Sharpe ratio for the number of strategy variants tested
- A strategy with nominal Sharpe of 1.5 but after testing 100 variants may have DSR near 0
- Must track total number of backtests run during research

**Bonferroni Correction:**
- Divide significance threshold by number of independent tests
- Conservative but protects against false discovery

### Reproducibility Requirements

| Requirement | Implementation |
|-------------|---------------|
| Random seeds | Fixed global seed (42) + per-experiment seeds |
| Data versioning | Parquet files with download timestamps |
| Parameter logging | YAML config files for every backtest run |
| Code versioning | Git commits for every experiment |
| Environment | `requirements.txt` or `pyproject.toml` with pinned versions |

---

## 5. Risk Management Implementation

### Volatility Scaling (Barroso & Santa-Clara)

**Core Algorithm:**
```
sigma_t = realized_vol(momentum_returns, window=126)  # 6-month window
scale_t = target_vol / sigma_t  # target_vol = 12% annualized
momentum_managed_t = scale_t * momentum_raw_t
```

**Key findings from the literature:**
- Market component accounts for only 23% of total momentum risk; most risk is strategy-specific
- Out-of-sample R-squared for predicting the specific component: 47.06% vs 20.87% for market component
- This explains why market beta hedging fails -- it addresses the smaller, less predictable part of momentum risk

**Implementation details:**
- Use 6-month (126 trading day) realized volatility as the predictor
- Target 12% annualized volatility
- Clip leverage between 0.1x and 3.0x to prevent extreme positions
- Recompute monthly at rebalance

**Source:** [Barroso & Santa-Clara (2015)](https://www.sciencedirect.com/science/article/abs/pii/S0304405X14002566)

### Dynamic Momentum (Daniel & Moskowitz Extension)

Daniel and Moskowitz improved on constant vol-targeting by incorporating:
- Forecasts of momentum's conditional mean (not just variance)
- Regime-dependent scaling
- Higher Sharpe ratio achievable but more complex to implement

### Conditional Volatility Targeting (Bongaerts, Kang & van Dijk)

Three-regime approach:
1. **Low volatility state:** Increase momentum exposure
2. **Normal volatility state:** Maintain unscaled exposure
3. **High volatility state:** Reduce momentum exposure

Benefits: "Significantly reduced drawdowns and tail risks across all major equity markets and momentum factors, while also reducing turnover."

### 52-Week High Neutralization

Alternative crash protection: Neutralize exposure to distance from 52-week high.
- "The near-high-neutral momentum strategy was free of crashes and exhibited a normal-like distribution"
- Minimum return improved from -69.3% to -26.87%
- Standard deviation decreased from 7.40% to 5.08%

**Source:** [Alpha Architect -- Minimizing Cross-Sectional Momentum Crashes](https://alphaarchitect.com/cross-sectional-momentum/)

---

## 6. Portfolio Construction Engine

### Weighting Schemes

| Scheme | Description | Pros | Cons |
|--------|-------------|------|------|
| **Equal-weight within quintile** | All stocks in long/short leg get equal weight | Simple, diversified | Ignores signal strength |
| **Signal-weighted** | Weight proportional to momentum score | Tilts to strongest signals | Concentration risk |
| **Volatility-inverse weighted** | Weight inversely proportional to stock volatility | Risk parity within leg | Penalizes volatile stocks |
| **Cap-weighted** | Weight by market cap within quintile | Investable, capacity-friendly | Mega-cap dominated |

**Recommendation:** Start with equal-weight within quintile (academic standard), then test signal-weighted and volatility-inverse as sensitivity analysis.

### Rebalance Implementation

- **Frequency:** Monthly (aligned with Kenneth French factor data)
- **Trade execution:** Assume market-on-close prices at rebalance date
- **Partial rebalance:** Consider bands (+/- 20% of target weight) to reduce turnover
- **Transaction cost model:** 50 bps one-way (25 bps market impact + 25 bps commission/slippage) for conservative estimate; also test 25 bps and 100 bps

### Capacity Analysis

Momentum strategy capacity is estimated at only ~$5.8B globally (Alpha Architect). This is important context:
- Our research targets large/mid-cap ($500M+ market cap) which improves capacity
- Equal-weighting across ~400 stocks (top quintile) with 5% max per stock limits
- Monthly turnover of ~200% one-way is typical for momentum
- At $100M AUM, market impact is negligible in this universe

**Source:** [Alpha Architect -- Factor Investing and Trading Costs](https://alphaarchitect.com/wp-content/uploads/2021/08/Factor_Investing_and_Trading_Costs.pdf)

---

## 7. Performance Attribution

### Factor Decomposition

Use Fama-French-Carhart 4-factor model for attribution:

```
R_momentum = alpha + beta_MKT * MKT + beta_SMB * SMB + beta_HML * HML + beta_UMD * UMD + epsilon
```

Report:
- Alpha (intercept) with t-statistic
- Factor loadings and their stability over time
- R-squared and adjusted R-squared
- Residual analysis (autocorrelation, normality)

### Risk Metrics Dashboard

| Metric | Computation |
|--------|-------------|
| Sharpe Ratio | Annualized (mean excess return / std) |
| Sortino Ratio | Uses downside deviation only |
| Maximum Drawdown | Peak-to-trough decline |
| Calmar Ratio | Annualized return / max drawdown |
| Value at Risk (95%) | Historical and parametric |
| Conditional VaR (95%) | Expected shortfall |
| Skewness | Third moment of return distribution |
| Kurtosis | Fourth moment (excess) |
| Hit Rate | Fraction of positive months |
| Profit Factor | Gross profits / gross losses |

### Regime Analysis

Segment performance by:
1. **VIX regime:** Low (<15), Normal (15-25), High (>25), Crisis (>35)
2. **Market regime:** Bull (12M S&P 500 return > 0), Bear (< 0)
3. **Rate regime:** Rising rates, falling rates, stable
4. **Momentum regime:** Strong momentum (wide L/S spread), weak momentum (narrow spread)

---

## 8. Recommended Technology Stack

### Core Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Language** | Python | 3.11+ | Primary implementation |
| **Backtesting** | VectorBT | 0.26+ | Vectorized backtesting engine |
| **Factor Analysis** | Alphalens | 0.4+ | Factor performance analysis |
| **Data** | pandas / PyArrow | 2.0+ / 14+ | Data manipulation + Parquet I/O |
| **Numerical** | NumPy / SciPy | 1.26+ / 1.12+ | Signal computation, statistics |
| **Statistics** | statsmodels | 0.14+ | Regression, factor models |
| **Visualization** | Plotly / Matplotlib | 5.18+ / 3.8+ | Interactive + static charts |
| **Data Download** | yfinance / getFamaFrenchFactors | Latest | Price data + factor data |
| **Notebook** | Jupyter Lab | 4.0+ | Research environment |
| **Version Control** | Git | 2.40+ | Code and experiment tracking |

### Development Environment

```
project/
  src/
    data/          # Data download and cleaning
    signals/       # Signal computation
    portfolio/     # Portfolio construction
    backtest/      # Walk-forward backtesting
    risk/          # Risk management and vol-scaling
    analysis/      # Performance attribution
    utils/         # Shared utilities
  notebooks/       # Jupyter research notebooks
  data/
    raw/           # Downloaded data (Parquet)
    processed/     # Cleaned data (Parquet)
    signals/       # Computed signals (Parquet)
    results/       # Backtest results (Parquet + JSON)
  configs/         # Experiment configurations (YAML)
  tests/           # Unit and integration tests
  docs/            # Research documentation
  requirements.txt # Pinned dependencies
```

---

## 9. Architecture Decision Records

### ADR-001: VectorBT over Zipline for Backtesting

**Context:** Need a Python backtesting framework that can handle cross-sectional factor strategies with walk-forward validation across 2,000 stocks and 25 years of daily data.

**Decision:** Use VectorBT as the primary backtesting engine.

**Rationale:**
- VectorBT's vectorized architecture is 10-100x faster than Zipline's event-driven approach for cross-sectional strategies
- Zipline requires Python 3.5-3.6 workarounds; VectorBT works with modern Python
- VectorBT's portfolio-level analysis natively supports long-short factor portfolios
- StrateQueue (2025) provides a bridge to live trading if needed

**Consequence:** Steeper learning curve for NumPy-centric API; will create wrapper functions for common operations.

### ADR-002: Kenneth French + yfinance for Data

**Context:** Need reliable, free factor return data and stock price data for US equities.

**Decision:** Use Kenneth French Data Library as primary factor data source; yfinance for individual stock prices.

**Rationale:**
- Kenneth French data is the academic gold standard; survivorship-bias-free; available since 1926
- yfinance provides free daily OHLCV for all US equities
- Total cost: $0
- AQR datasets supplement with international and multi-asset momentum data

**Consequence:** yfinance is fragile (scrapes Yahoo); will cache all downloads locally and implement retry logic. Consider EODHD or Stooq as fallback.

### ADR-003: Walk-Forward with CPCV Supplement

**Context:** Need robust validation that prevents overfitting while providing realistic performance estimates.

**Decision:** Implement rolling walk-forward analysis as primary validation; add CPCV as supplementary check.

**Rationale:**
- Walk-forward is the industry gold standard and most intuitive to interpret
- CPCV provides superior false discovery prevention but is more complex
- Using both provides defense-in-depth against overfitting
- Walk-Forward Efficiency (WFE) metric gives a single number for robustness

**Consequence:** More computation required for CPCV; can be run as a final validation step rather than during iterative research.

### ADR-004: Parquet over CSV for Data Storage

**Context:** Need efficient storage for daily price data across 2,000 stocks over 25 years.

**Decision:** Use Apache Parquet format for all data storage.

**Rationale:**
- Columnar format enables efficient time-series queries (5-10x faster than CSV)
- Built-in compression reduces storage 5-10x
- Native pandas integration via PyArrow
- Schema enforcement prevents data corruption
- Industry standard for quantitative finance data

**Consequence:** Requires PyArrow dependency; CSV export available for debugging.

---

## 10. Implementation Roadmap

### Phase 1: Data Pipeline (Foundation)

| Task | Description | Dependencies |
|------|-------------|--------------|
| 1.1 | Download Kenneth French factor data (MKT, SMB, HML, UMD) | None |
| 1.2 | Download AQR momentum datasets | None |
| 1.3 | Build yfinance downloader with caching and retry | None |
| 1.4 | Download US equity universe (S&P 500 + Russell 1000 constituents) | 1.3 |
| 1.5 | Build data cleaning pipeline (split adjust, NaN handling, delisting) | 1.4 |
| 1.6 | Create Parquet data store with versioning | 1.5 |

### Phase 2: Signal Engine

| Task | Description | Dependencies |
|------|-------------|--------------|
| 2.1 | Implement cross-sectional momentum (12-1, 6-1) | 1.6 |
| 2.2 | Implement time-series momentum | 1.6 |
| 2.3 | Implement earnings momentum (SUE) | 1.6 |
| 2.4 | Implement factor momentum | 1.1, 1.2 |
| 2.5 | Build composite signal framework | 2.1-2.4 |
| 2.6 | Validate signals against Kenneth French published returns | 2.1, 1.1 |

### Phase 3: Backtesting & Validation

| Task | Description | Dependencies |
|------|-------------|--------------|
| 3.1 | Build VectorBT walk-forward backtesting harness | 2.5 |
| 3.2 | Implement volatility scaling (Barroso & Santa-Clara) | 3.1 |
| 3.3 | Run single-signal backtests with WFA | 3.1, 3.2 |
| 3.4 | Run composite-signal backtests with WFA | 3.3 |
| 3.5 | Implement CPCV validation | 3.3 |
| 3.6 | Transaction cost sensitivity analysis | 3.3, 3.4 |

### Phase 4: Analysis & Reporting

| Task | Description | Dependencies |
|------|-------------|--------------|
| 4.1 | Factor attribution (4-factor model) | 3.3, 3.4 |
| 4.2 | Regime-conditional analysis | 3.3, 3.4 |
| 4.3 | Risk metrics dashboard | 3.3, 3.4 |
| 4.4 | Capacity analysis | 3.6 |
| 4.5 | Final research report | 4.1-4.4 |

---

## Sources

- [VectorBT -- Battle-Tested Backtesters Comparison (Medium)](https://medium.com/@trading.dude/battle-tested-backtesters-comparing-vectorbt-zipline-and-backtrader-for-financial-strategy-dee33d33a9e0)
- [Top 21 Python Trading Tools -- January 2026 (Analyzing Alpha)](https://analyzingalpha.com/python-trading-tools)
- [Python Backtesting Frameworks: Six Options (Pipekit)](https://pipekit.io/blog/python-backtesting-frameworks-six-options-to-consider)
- [From Backtest to Live: VectorBT in 2025 (Medium)](https://medium.com/@samuel.tinnerholm/from-backtest-to-live-going-live-with-vectorbt-in-2025-step-by-step-guide-681ff5e3376e)
- [Python Roadmap 2026 for Traders (MarketCalls)](https://www.marketcalls.in/python/python-roadmap-2026-a-strategic-guide-for-traders-and-investors.html)
- [Kenneth French Data Library](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html)
- [AQR Datasets](https://www.aqr.com/Insights/Datasets)
- [Open Source Asset Pricing](https://www.openassetpricing.com)
- [Alphalens (GitHub)](https://github.com/quantopian/alphalens)
- [Walk-Forward Optimization (QuantInsti, 2025)](https://blog.quantinsti.com/walk-forward-optimization-introduction/)
- [Walk-Forward Analysis Deep Dive (Interactive Brokers, 2025)](https://www.interactivebrokers.com/campus/ibkr-quant-news/the-future-of-backtesting-a-deep-dive-into-walk-forward-analysis/)
- [Backtest Overfitting Comparison -- CPCV (ScienceDirect, 2024)](https://www.sciencedirect.com/science/article/abs/pii/S0950705124011110)
- [Rigorous Walk-Forward Validation Framework (arXiv, Dec 2025)](https://arxiv.org/html/2512.12924v1)
- [Barroso & Santa-Clara -- Momentum Has Its Moments (2015)](https://www.sciencedirect.com/science/article/abs/pii/S0304405X14002566)
- [Alpha Architect -- Minimizing Momentum Crashes](https://alphaarchitect.com/cross-sectional-momentum/)
- [Alpha Architect -- Factor Investing and Trading Costs](https://alphaarchitect.com/wp-content/uploads/2021/08/Factor_Investing_and_Trading_Costs.pdf)
- [On Alpha Decay and Transaction Costs (arXiv, Feb 2025)](https://arxiv.org/abs/2502.04284)
- [Garleanu & Pedersen -- Dynamic Trading with Predictable Returns](http://docs.lhpedersen.com/DynamicTrading.pdf)
- [Research Affiliates -- Can Momentum Investing Be Saved?](https://www.researchaffiliates.com/publications/articles/637-can-momentum-investing-be-saved)
