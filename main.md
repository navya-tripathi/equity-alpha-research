# Overview

This project implements an end-to-end systematic equity research
pipeline investigating whether classical factor signals of momentum,
value, quality, and earnings surprise predict forward returns in the S&P
500. Built independently to mirror institutional buy-side research
workflows: signal construction, rolling backtesting, OLS cross-sectional
regression, and rigorous performance attribution.

The research evaluates three model specifications across different
universes and data availability constraints, with a focus on
understanding which factors drive returns and whether combining signals
improves risk-adjusted performance.

# Key Results

  **Model**                     **Universe**   **Months**   **Ann. Return**   **Sharpe**   **Max DD**   
  ----------------------------- -------------- ------------ ----------------- ------------ ------------ --
  A --- Four-Factor (Core)      358 tickers    59           8.9%              0.49         -26.8%       
  B --- Momentum Only           498 tickers    136          27.2%             **1.33**     -20.3%       
  C --- Four-Factor (Overlap)   325 tickers    59           6.9%              0.39         -25.5%       

  : Performance summary across all three model specifications

**Headline finding:** A simple price-momentum strategy (Model B)
significantly outperformed multi-factor fundamental models across a
broader universe and longer time horizon, achieving a Sharpe ratio of
1.33 gross of transaction costs over 11+ years. Adding value, quality,
and earnings surprise signals to momentum actually *reduced* the Sharpe
ratio in the test period which is a non-obvious result with meaningful
implications for future factor combination.

# Signals Researched

## Price Momentum (12-1 month)

Cumulative return from month $t-12$ to $t-2$, skipping the most recent
month to avoid short-term reversal. Ranked cross-sectionally and
z-scored monthly. Highly significant across all model specifications
($t = 7.95$ in Model B, $p < 0.001$).

## Book-to-Market Value

Book value per share divided by price, constructed from quarterly
fundamental data with point-in-time reporting lag applied. Loaded
*negatively* and significantly ($p \approx 0.02$) and consistent with
the documented post-2010 deterioration of the value premium among
large-cap U.S. equities.

## Gross Profitability / Quality

Revenue minus cost of goods sold divided by total assets (Novy-Marx,
2013). Positive coefficient ($\approx 0.030$) across both four-factor
models but statistically weak ($p \approx 0.056$), suggesting a quality
premium that does not clear conventional significance thresholds in this
sample.

## Standardized Unexpected Earnings (SUE)

Earnings surprise was normalized by historical surprise volatility.
Near-zero coefficient and statistically insignificant ($p \approx 0.81$)
after controlling for other factors and no incremental predictive power
in this universe and period.

# Methodology

## Universe Construction

Three universes tested: a core 358-ticker universe with full fundamental
data coverage (Models A and C), and a broader 498-ticker price-only
universe enabling a longer backtest history (Model B).

## Signal Construction

All signals constructed with strict point-in-time discipline and only
data available as of each rebalance date used. Fundamental signals use
the most recently *reported* quarter, not the most recently *ended*
quarter, to prevent look-ahead bias in accounting data.

## Backtesting

Monthly rebalance frequency. Long-short quintile portfolios: long top
20% of signal, short bottom 20%, equal-weighted within each leg.
Performance calculated as the return spread between long and short legs.

## Factor Validation

OLS cross-sectional regressions run on standardized signals and
standardized forward returns. The model estimated is:

$$\begin{equation}
r_{i,t+1} = \alpha + \beta_1 \text{MOM}_{i,t} + \beta_2 \text{VAL}_{i,t} + \beta_3 \text{QUAL}_{i,t} + \beta_4 \text{SUE}_{i,t} + \varepsilon_{i,t}
\end{equation}$$

where all variables are cross-sectionally standardized (z-scored) at
each rebalance date. Coefficients, standard errors, t-statistics, and
95% confidence intervals reported for all factors across all models.

## Performance Metrics

Annualized return, annualized volatility, Sharpe ratio (annualized,
risk-free rate = 0 for simplicity), and maximum peak-to-trough drawdown
reported for each model.

# Statistical Findings

  **Factor**   **Model A Coef.**   **p-value**   **Model B Coef.**   **p-value**                 **Model C Coef.**   **p-value**
  ------------ ------------------- ------------- ------------------- --------------------------- ------------------- -------------
  Momentum     +0.031              0.030         +0.031              $<$`<!-- -->`{=html}0.001   +0.028              0.058
  Value        -0.036              0.021         ---                 ---                         -0.036              0.021
  Quality      +0.030              0.056         ---                 ---                         +0.030              0.057
  SUE          -0.004              0.808         ---                 ---                         -0.003              0.823

  : OLS cross-sectional regression results across all model
  specifications

Momentum is the only factor significant at the 5% level across all
specifications. Value loads negatively, statistically significant but
directionally opposite to classical Fama-French predictions, which is
consistent with post-2010 large-cap value underperformance documented in
the literature.

# Limitations & Honest Assessment

**Survivorship bias:** This backtest uses S&P 500 constituents as of the
data pull date, not historical constituent lists. Companies removed from
the index due to bankruptcy or delisting are excluded, which overstates
performance. A production-quality backtest would use a point-in-time
constituent database (e.g., Compustat).

**Transaction costs:** All results are gross of transaction costs.
Monthly rebalancing of a long-short equity portfolio involves meaningful
turnover. At 10bps per trade, a realistic estimate would reduce the
Model B Sharpe ratio from 1.33 to approximately 0.90--1.05 depending on
realized turnover.

**Look-ahead bias in universe construction:** The three model universes
were defined based on data availability observed ex-post, which may
introduce subtle selection bias in the fundamental universe (Models A
and C).

**Short backtest for fundamental models:** Models A and C cover only 59
months ($\approx$`<!-- -->`{=html}5 years) due to fundamental data
availability constraints --- too short to draw strong conclusions about
factor robustness across market regimes.

# Repository Structure

    equity-alpha-research/
    |
    |-- README.md
    |-- requirements.txt
    |
    |-- notebooks/
    |   |-- 01_data_pipeline.ipynb
    |   |-- 02_factor_A_core_358.ipynb
    |   |-- 03_factor_B_price_only_500.ipynb
    |   |-- 04_factor_C_price_plus_fundamentals.ipynb
    |   |-- 05_portfolio_construction.ipynb
    |   |-- 06_backtest_A_core_358.ipynb
    |   |-- 07_backtest_B_price_only_500.ipynb
    |   |-- 08_backtest_C_overlap.ipynb
    |   |-- 09_fama_macbeth_analysis.ipynb
    |   |-- 10_ols_validation.ipynb
    |   |-- 11_final_evaluation.ipynb
    |   |-- 12_figures_and_report.ipynb
    |

# Tech Stack

Python $\cdot$ pandas $\cdot$ NumPy $\cdot$ SciPy $\cdot$ statsmodels
$\cdot$ yfinance $\cdot$ SimFin $\cdot$ plotly $\cdot$ matplotlib
$\cdot$ seaborn

# Background & Motivation

This project was built independently as part of a deliberate transition
from academic quantitative research into systematic quantitative
finance. The goal was to apply the same end-to-end research methodology:
hypothesis formation, data pipeline construction, model implementation,
rigorous validation, and reporting of results and limitations.

The research question was deliberately simple: do the most well-studied
equity factors actually work in practice, on real data, with a real
backtest? The answer is nuanced: momentum does, robustly. The others are
weaker, regime-dependent, or directionally inconsistent with classical
predictions in the post-2010 large-cap universe. That finding is, in
itself, the result.

*All data sourced from yfinance (price/return data) and SimFin
(fundamental data). Results are for research purposes only.*
