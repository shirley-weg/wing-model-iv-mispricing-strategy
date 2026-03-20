# A Delta-Hedged Implied Volatility Mispricing Strategy Based on WLS Calibration of a Quadratic Wing Model

## Project Overview
This project studies implied-volatility (IV) mispricing in SPX options using a weighted least squares (WLS) calibration of a parsimonious quadratic wing model. The main idea is to fit a theoretical IV smile to out-of-the-money (OTM) option data, identify statistically significant deviations between market IV and fitted IV, and construct a delta-hedged long/short options strategy designed to profit from mean reversion.

The full workflow includes:
1. constructing an OTM IV dataset for each trading date and expiration,
2. calibrating a quadratic IV curve using WLS,
3. detecting mispricing through residual-based standardized signals,
4. opening long or short option positions based on the signal,
5. rebalancing the underlying hedge daily to maintain approximate delta neutrality,
6. evaluating the strategy through historical backtesting.

This repository is organized as a research-style quantitative trading project, containing the notebook implementation, report, and performance figures.

---

## Research Objective
The objective of this project is to test whether deviations between observed option implied volatility and a fitted cross-sectional IV curve can be systematically exploited in a trading strategy.

More specifically, the project asks:

- Can a simple quadratic wing model capture the shape of the IV smile sufficiently well for signal generation?
- Can volume-weighted least squares improve calibration quality by emphasizing more actively traded contracts?
- Can large residual deviations from the fitted IV curve serve as useful mispricing signals?
- Does a delta-hedged long/short options strategy based on these signals generate economically meaningful performance after transaction costs?

---

## Methodology

### 1. OTM IV Construction
To reduce put-call inconsistencies and focus on the most informative option quotes, the OTM implied-volatility series is constructed as follows:

- use **put IV** for strikes below or equal to spot,
- use **call IV** for strikes above spot.

This produces a cleaner cross-sectional IV smile for each trading date and expiration.

### 2. Signal Universe and Execution Universe
The project uses two related universes:

- **Signal universe**: a filtered moneyness band used for calibration and signal generation.
- **Execution universe**: a broader set of valid quotes retained for trade execution, valuation, and position unwinding.

This separation helps maintain cleaner signal estimation while still allowing positions to be closed using available market quotes.

### 3. Quadratic Wing Model
For each date and expiration, the theoretical implied-volatility curve is modeled as:

\[
\sigma_{t,e}^{model}(K) = a_{0,t,e} + a_{1,t,e} K + a_{2,t,e} K^2
\]

where \(K\) is the strike and the coefficients are estimated separately for each date-expiration slice.

### 4. WLS Calibration
The IV curve is estimated using **volume-weighted least squares (WLS)**.  
Compared with ordinary least squares, WLS gives higher influence to contracts with greater trading activity, which may better reflect market consensus and improve calibration robustness.

### 5. Mispricing Detection
After fitting the IV curve, residuals are computed as:

- market IV minus fitted IV.

These residuals are then standardized to form z-scores. The strategy interprets the signal as:

- **negative z-score**: the option appears underpriced in IV terms,
- **positive z-score**: the option appears overpriced in IV terms.

Large-magnitude z-scores are treated as potential IV mispricing opportunities.

### 6. Trading Logic
The strategy forms positions according to the signal:

- **Buy option** when IV appears underpriced
- **Sell option** when IV appears overpriced

The implementation focuses on OTM options for signal generation.

### 7. Daily Delta Hedging
To reduce directional exposure to the underlying, the strategy computes Black-Scholes delta for each option and rebalances the underlying hedge daily to target approximate delta neutrality.

This means the strategy is designed to focus more on relative IV reversion than on outright market direction.

### 8. Exit Rules
Positions are closed when any of the following conditions is met:

- the z-score mean-reverts inside the exit threshold,
- the holding period exceeds the maximum holding window,
- the remaining days to expiration fall below the minimum DTE threshold.

### 9. Transaction Costs
The backtest includes simplified but explicit trading frictions:

- option trading commissions at entry and exit,
- underlying hedge slippage during daily rebalancing.

This makes the performance evaluation more realistic than a frictionless backtest.

---

## Backtest Protocol
A typical daily backtest loop proceeds as follows:

1. preprocess option data and apply liquidity / validity filters,
2. calibrate the IV curve for each expiration using WLS,
3. compute residuals and z-scores,
4. select mispriced candidates,
5. open long/short positions subject to trading constraints,
6. rebalance the underlying hedge,
7. evaluate exit conditions,
8. update daily P&L, cash, and equity curve.

---

## Key Parameter Settings
The report uses the following representative backtest settings:

| Parameter | Symbol | Value |
|---|---:|---:|
| Entry threshold (z-score) | \(z_{enter}\) | 2.0 |
| Exit threshold (z-score) | \(z_{exit}\) | 0.5 |
| Max trades per expiry per day | \(N_{tr}\) | 1 |
| Max holding days | \(H\) | 10 |
| Minimum DTE | \(DTE_{min}\) | 3 |
| Contracts per trade | \(q_0\) | 1 |
| Minimum volume filter | \(V_{min}\) | 1 |
| Minimum option price filter | \(p_{min}\) | 0.01 |
| Initial cash | \(E_0\) | 1,000,000 |
| Option multiplier | \(M\) | 100 |
| Option commission | \(c_{opt}\) | 0.50 |
| Underlying slippage (bps) | \(b\) | 1.0 |

---

## Backtest Results
The reported strategy results are:

| Metric | Value |
|---|---:|
| Start Equity | 1,000,000.00 |
| End Equity | 1,042,027.99 |
| Total Return | 4.20% |
| Annualized Sharpe Ratio | 0.438 |
| Maximum Drawdown (MDD) | 14.02% |

### Interpretation
The backtest shows positive total return, but only moderate risk-adjusted performance. A likely reason is that some detected IV deviations do not revert quickly within the allowed holding window. In addition, the quadratic smile is only an approximation, so persistent skew or volatility risk premia may be labeled as mispricing. Daily hedge rebalancing and transaction costs also reduce net performance.

---

## Repository Structure
```text
wing-model-iv-mispricing-strategy/
├─ README.md
├─ requirements.txt
├─ .gitignore
├─ notebooks/
│  └─ Wing_Model_IV_Calibration.ipynb
├─ reports/
│  └─ SPX_trading_strategy.pdf
└─ figures/
   ├─ equity_curve.png
   ├─ exposure_chart.png
   ├─ cum_return_mdd.png
   └─ mispricing_example.png
