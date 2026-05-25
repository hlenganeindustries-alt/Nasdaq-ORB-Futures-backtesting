# 📈 NQ Futures — Opening Range Breakout Backtest

A Python-based backtesting framework for an **Opening Range Breakout (ORB)** strategy on NQ (Nasdaq 100 Futures), built with [`backtesting.py`](https://kernc.github.io/backtesting.py/) and run in Google Colab.

> ⚠️ **Data limitation:** Results are based on ~2 years of hourly data (2024–2026). This is the maximum available through the yfinance API for 1H intervals. Longer-term conclusions should be treated with caution.

> ⚠️ Disclaimer
Simulated results only — not financial advice.

>This strategy has not been tested with live funds. All backtests were conducted using Python and Yahoo Finance 1-hour price data. The system remains a work in progress as I continue researching optimal entry criteria. The logic shown is purely educational.
---
## Repository Structure



>backtest reasults csv/

>/

>Python code use for System/




## 🧠 Strategy Overview

The strategy trades the NY session open on NQ Futures using **close-only signals** — no High/Low data is used anywhere in the entry logic.

### ⏰ Entry Rules

| Time (UTC) | Time (ET) | Time (SAST) | Event |
|------------|-----------|-------------|-------|
| 13:00 | 08:00 | 15:00 | Pre-open bar — close recorded |
| 14:00 | 09:00 | 16:00 | ORB bar — trigger level set |
| 15:00 | 10:00 | 17:00 | Entry bar — long or short signal |

- The **14:00 UTC close** is the trigger level
- If the **15:00 UTC close** is **above** the trigger → 🟢 **LONG**
- If the **15:00 UTC close** is **below** the trigger → 🔴 **SHORT**
- If flat → no trade that day

### 🛡️ Risk Management

- **SL** = `sl_atr_mult × ATR14` below/above entry
- **Scale 1** — close 33% of position when price moves 1× ATR14 in your favour → SL moves to breakeven
- **Scale 2** — close another 33% when price moves 2× ATR14 in your favour
- **Remainder** — final 34% runs to full TP at `risk_reward × ATR14`
- No EOD force-close — positions held until SL or TP is hit

### 🔍 Filters

- **Entry filter** — 15:00 close must move at least `entry_pct × ATR14` past the ORB close to avoid noise entries
- **Volatility filter** — skips low-volatility days where ATR14 percentile rank is below `atr_pct_min`
- **Day-of-week filter** — optionally skip Mondays and/or Fridays

---

## 📐 What is ATR?

**ATR (Average True Range)** measures how much NQ typically moves per day. ATR14 is the average of the last 14 days' true ranges.

It adapts SL/TP to current market conditions:
- 📊 High volatility day → wider SL/TP so you're not shaken out by noise
- 📉 Quiet day → tighter SL/TP so you're not risking more than you need to

This is why you'll see the **ATR14 panel** spike sharply around **April 2025** in the equity curve chart — that was an extreme volatility event where NQ dropped from ~21,000 to ~17,000 and back.

---

## 📊 Results

### Baseline Stats (default parameters)

The stats below show the strategy running on default parameters with no optimisation. Key numbers to note:

- ✅ **Return: +8.63%** over the backtest period
- ✅ **Win Rate: 59.9%** — more winners than losers
- ✅ **Profit Factor: 3.03** — for every $1 lost, $3.03 was made
- ✅ **Avg Trade: +0.97%** — positive expectancy per trade
- ⚠️ **Max Drawdown: -18.4%** — the account fell 18.4% from its peak at one point
- ⚠️ **Sharpe Ratio: 0.38** — returns relative to volatility, ideally above 1.0

<img width="337" height="752" alt="Baseline stats" src="https://github.com/user-attachments/assets/942a41e5-d19f-4efa-8c1b-291c123dff9e" />

---

### 📉 Equity Curve

The chart below shows the full picture of the strategy over ~2 years:

- **Top panel** — equity curve (blue line). Peak was **+15%**, finishing at **+9%**. The red dot marks where the deepest drawdown occurred (around April 2025 during an extreme NQ sell-off)
- **Middle panel** — individual trade profit/loss. Green triangles = winning trades, red = losing trades. You can see the scale-out working — some trades show multiple exit points (the grey lines connecting triangle pairs)
- **Bottom panels** — ATR14 and ATR percentile rank. Notice ATR14 spikes to ~400pts during the April 2025 crash — the strategy automatically widened its SL/TP in response

<img width="1435" height="760" alt="Equity curve" src="https://github.com/user-attachments/assets/e57c79d7-477f-4878-bc3f-2f37520a7b78" />

---

### ⚙️ After Optimisation

After running a grid search across all parameters (1,024 combinations), the optimiser found the best settings:

```
risk_reward  : 2.5
sl_atr_mult  : 1.0
entry_pct    : 0.1
atr_pct_min  : 0.0
skip_monday  : False
skip_friday  : False
```

The improvement over baseline is significant:

- 🚀 **Return jumped from +8.6% → +48.3%**
- 📈 **Sharpe Ratio: 1.51** — now above the 1.0 threshold, indicating good risk-adjusted returns
- ✅ **Win Rate: 66.2%** — 2 in every 3 trades profitable
- ✅ **Profit Factor: 4.05** — for every $1 lost, $4.05 was made
- ⚠️ **Max Drawdown: -13.4%** — reduced from -18.4%
- 📊 **216 trades** taken over the period

<img width="450" height="500" alt="Optimised stats" src="https://github.com/user-attachments/assets/485ffdde-2c77-4e1c-a00e-b64d591c1e51" />

### ✅ Best Parameter Set

| Parameter | Value |
|-----------|-------|
| `risk_reward` | 2.5 |
| `sl_atr_mult` | 1.0 |
| `entry_pct` | 0.1 |
| `atr_pct_min` | 0.0 |
| `skip_monday` | False |
| `skip_friday` | False |

---

### 📅 Day-of-Week Breakdown

The table and charts below show how the strategy performs on each day of the week.

The standout finding is **Tuesday** — it's the only losing day with a **negative avg PnL of -$166** and a win rate of exactly 50% (coin flip). Every other day is profitable, with **Thursday** being the strongest at **+$375 avg PnL and 73% win rate**.

| Day | Legs | Win Rate | Avg PnL | Total PnL |
|-----|------|----------|---------|-----------|
| Monday | 42 | 67% | +$298.75 | +$12,547 |
| **Tuesday** | 32 | **50%** | **-$166.23** | **-$5,319** |
| Wednesday | 48 | 73% | +$241.54 | +$11,593 |
| Thursday | 52 | 73% | +$375.58 | +$19,530 |
| Friday | 42 | 62% | +$212.84 | +$8,939 |

> 💡 Consider setting `skip_tuesday = True` in a future version to filter out the weakest day.

<img width="932" height="398" alt="Day of week breakdown" src="https://github.com/user-attachments/assets/b09427a1-e4de-479f-99ba-db10ba2e0f77" />

---

## 🔧 Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `risk_reward` | 3.0 | Final TP as a multiple of ATR14 |
| `sl_atr_mult` | 1.0 | Initial SL as a multiple of ATR14 |
| `entry_pct` | 0.1 | Min move past ORB close as fraction of ATR14 |
| `atr_pct_min` | 0.3 | Skip days below this ATR percentile (0–1) |
| `skip_monday` | 0 | Set to 1 to skip all Monday trades |
| `skip_friday` | 0 | Set to 1 to skip all Friday trades |

---

## 🚀 Setup & Usage

### Requirements

```
yfinance
backtesting
pandas_ta
```

### Running in Google Colab

1. Open the notebook in Google Colab
2. Run **Cell 1** to install dependencies
3. Run **Cell 2** to download NQ data
4. Run **Cells 3–5** to build ATR, define the strategy, and run the baseline backtest
5. Run **Cell 6** for the interactive equity curve and trade chart
6. Run **Cell 7** for the full optimisation sweep *(allow 10–20 min)*
7. Run **Cells 8–9** for the best-params run and day-of-week breakdown
8. Run **Cell 10** to push the notebook to this repository

### 🔑 Pushing to GitHub

Cell 10 will prompt for a **GitHub Personal Access Token**. To generate one:

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click *Generate new token (classic)*
3. Tick the `repo` scope
4. Copy the token — you only see it once

---

## 🗂️ Version History

| Version | Description |
|---------|-------------|
| v1 | Initial ORB using High/Low for breakout detection |
| v2 | Close-only signals, dynamic range size from 1H close diff |
| v3 | ATR-based SL/TP, entry filter, volatility filter, DOW filter |
| v4 | Scale-out (33%/33%/34%), breakeven SL, no EOD force-close |

---

## 📝 Notes & Observations

**Tuesday is a problem day** — after running the day-of-week breakdown, Tuesday stands out as the only consistently losing day with a 50% win rate and an average loss of -$166 per trade. Every other day is profitable. This could be related to post-Monday repositioning or macro data releases that commonly fall on Tuesdays. The next step is to filter out Tuesday entries entirely and see how much it improves the overall performance.

**The scale-out structure made the biggest difference** — the original strategy, which used the opening range High/Low for breakout detection, came in at -20.8% return. Once the strategy was rebuilt around close-only signals and a three-stage scale-out (33% at 1× ATR, 33% at 2× ATR, remainder at full TP with the stop moved to breakeven), the return moved to +8.6% on default parameters and +48.3% after optimisation. That single structural change — scaling out rather than exiting all at once — was responsible for the turnaround.

**The strategy held up during the April 2025 NQ crash** — NQ dropped from around 21,000 to 17,000 during this period, which is visible in the equity curve as the sharp dip in the price chart. Because SL and TP distances are based on ATR14, the strategy automatically widened its levels during that high-volatility period rather than getting stopped out repeatedly by the noise. The equity curve dips during this period but recovers, which is an encouraging sign of robustness under stress conditions.

**The optimisation results need to be treated carefully** — the jump from +8.6% to +48.3% after running the parameter grid search looks impressive, but this backtest only covers approximately 2 years of data. With 1,024 parameter combinations tested on a relatively short dataset, there is a real risk that the best parameters are overfitted to this specific period rather than representing a genuine edge. The plan is to forward test the optimised parameters on live data going forward to validate whether the edge holds.

**Next steps** — skip Tuesday trades and re-run to measure the impact, test the same strategy logic on ES (S&P 500 Futures) to see if the edge generalises beyond NQ, and explore sourcing longer historical data through a paid provider to run the backtest over a larger sample size and reduce the overfitting risk.

---

## ⚠️ Disclaimer

This project is for **educational and research purposes only**. Nothing here constitutes financial advice. Backtested results do not guarantee future performance. Results are based on approximately 2 years of data due to yfinance API limitations on 1-hour intervals — this is a relatively short sample size and live performance may differ significantly. **Trade at your own risk.**
