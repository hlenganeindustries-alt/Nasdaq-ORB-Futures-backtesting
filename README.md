# NQ Futures — Opening Range Breakout Backtest

A Python-based backtesting framework for an **Opening Range Breakout (ORB)** strategy on NQ (Nasdaq 100 Futures), built with [`backtesting.py`](https://kernc.github.io/backtesting.py/) and run in Google Colab.

---

## Strategy Overview

The strategy trades the NY session open on NQ Futures using close-only signals — no High/Low data is used anywhere in the entry logic.

### Entry Rules

| Time (UTC) | Time (ET) | Time (SAST) | Event |
|------------|-----------|-------------|-------|
| 13:00 | 08:00 | 15:00 | Pre-open bar — close recorded |
| 14:00 | 09:00 | 16:00 | ORB bar — trigger level set |
| 15:00 | 10:00 | 17:00 | Entry bar — long or short signal |

- The **14:00 UTC close** is the trigger level
- If the **15:00 UTC close** is above the trigger → **LONG**
- If the **15:00 UTC close** is below the trigger → **SHORT**
- If flat → no trade that day

### Risk Management

- **SL** = `sl_atr_mult × ATR14` below/above entry
- **Scale 1** = close 33% of position at entry + 1× ATR14 → SL moves to breakeven
- **Scale 2** = close 33% of position at entry + 2× ATR14
- **Remainder** = final 34% runs to full TP at `risk_reward × ATR14`
- No EOD force-close — positions held until SL or TP is hit

### Filters

- **Entry filter** — 15:00 close must move at least `entry_pct × ATR14` past the ORB close
- **Volatility filter** — skips days where ATR14 percentile rank is below `atr_pct_min`
- **Day-of-week filter** — optionally skip Mondays and/or Fridays

---

## What is ATR?

ATR (Average True Range) measures how much NQ typically moves per day. ATR14 = average of the last 14 days' true ranges. It adapts SL/TP to current volatility — wider on high-vol days, tighter on quiet days — so you're never stopped out by normal noise.

---

## Results

> _Paste your baseline stats output here from Cell 5_


<img width="337" height="752" alt="Screenshot (215)" src="https://github.com/user-attachments/assets/942a41e5-d19f-4efa-8c1b-291c123dff9e" />



### After Optimisation

> _Paste your best-params stats output here from Cell 7_


<img width="450" height="500" alt="Screenshot (216)" src="https://github.com/user-attachments/assets/485ffdde-2c77-4e1c-a00e-b64d591c1e51" />



### Equity Curve

<img width="1435" height="760" alt="Screenshot (217)" src="https://github.com/user-attachments/assets/e57c79d7-477f-4878-bc3f-2f37520a7b78" />


---

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `risk_reward` | 3.0 | Final TP as a multiple of ATR14 |
| `sl_atr_mult` | 1.0 | Initial SL as a multiple of ATR14 |
| `entry_pct` | 0.1 | Min move past ORB close as fraction of ATR14 |
| `atr_pct_min` | 0.3 | Skip days below this ATR percentile (0–1) |
| `skip_monday` | 0 | Set to 1 to skip all Monday trades |
| `skip_friday` | 0 | Set to 1 to skip all Friday trades |

---


```

### Best Parameter Set Found

> _Fill in after running Cell 7_

| Parameter | Value |
|-----------|-------|
| `risk_reward` | |
| `sl_atr_mult` | |
| `entry_pct` | |
| `atr_pct_min` | |
| `skip_monday` | |
| `skip_friday` | |

---

## Day-of-Week Breakdown

> _Paste the DOW table output from Cell 9 here_

```
[PASTE DOW TABLE HERE]


<img width="932" height="398" alt="Screenshot (218)" src="https://github.com/user-attachments/assets/b09427a1-e4de-479f-99ba-db10ba2e0f77" />


---

## Setup & Usage

### Requirements

```
yfinance
backtesting
pandas_ta
```

### Running in Google Colab

1. Open the notebook in Google Colab
2. Run Cell 1 to install dependencies
3. Run Cell 2 to download NQ data (falls back to synthetic data if yfinance is unavailable)
4. Run cells 3–5 in order to build ATR, define the strategy, and run the baseline backtest
5. Run Cell 6 for the interactive Bokeh chart
6. Run Cell 7 for the full optimisation sweep *(allow 10–20 min)*
7. Run cells 8–9 for the best-params run and day-of-week breakdown
8. Run Cell 10 to push the notebook to this repository

### Pushing to GitHub

Cell 10 will prompt you for a **GitHub Personal Access Token**. To generate one:

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click *Generate new token (classic)*
3. Tick the `repo` scope
4. Copy the token — you only see it once

---

## Versions

| Version | Description |
|---------|-------------|
| v1 | Initial ORB using High/Low for breakout detection |
| v2 | Close-only signals, dynamic range size from 1H close diff |
| v3 | ATR-based SL/TP, entry filter, volatility filter, DOW filter |
| v4 | Scale-out (33%/33%/34%), breakeven SL, no EOD force-close |

---

## Notes & Observations

> _Add your own notes here as you run experiments — what worked, what didn't, what you want to try next_

---

## Disclaimer

This project is for **educational and research purposes only**. Nothing here constitutes financial advice. Backtested results do not guarantee future performance. Trade at your own risk.
