# Trader Performance vs Market Sentiment
### Primetrade.ai — Data Science Intern Assignment

---

## Overview

This project analyzes how Bitcoin market sentiment (Fear/Greed Index) relates to trader behavior and performance on the Hyperliquid DEX. The analysis covers ~211K trades from 32 unique trader accounts across 479 overlapping days (May 2023 – May 2025).

---

## Repository Structure

```
primetrade_assignment/
├── analysis_notebook.ipynb     ← Full analysis (executed, with outputs & charts)
├── fear_greed_index.csv        ← Bitcoin Fear/Greed dataset
├── historical_data.csv         ← Hyperliquid trader data
├── charts/                     ← All saved figures (PNG)
│   ├── fig1_performance_by_sentiment.png
│   ├── fig2_behavior_by_sentiment.png
│   ├── fig3_segment_frequency.png
│   ├── fig4_segment_size.png
│   ├── fig5_segment_winners_losers.png
│   ├── fig6_pnl_vs_sentiment_timeseries.png
│   ├── fig7_heatmap_trader_sentiment.png
│   ├── fig8_longshort_volume.png
│   └── fig9_predictive_model.png
└── README.md                   ← This file
```

---

## Setup & How to Run

### Requirements
```
Python 3.9+
pandas, numpy, matplotlib, seaborn, scikit-learn, jupyter, nbformat
```

### Install dependencies
```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter nbformat
```

### Run the notebook
```bash
jupyter notebook analysis_notebook.ipynb
```
Or execute headlessly:
```bash
jupyter nbconvert --to notebook --execute analysis_notebook.ipynb --output analysis_notebook.ipynb
```

> Both CSV files must be in the same directory as the notebook.

---

## Methodology

### Data Preparation (Part A)
- **Fear/Greed Index**: 2,644 rows, no missing values. Dates parsed and classification simplified to a 3-class schema (Fear / Neutral / Greed) in addition to the original 5-class for granular analysis.
- **Historical Trader Data**: 211,224 rows across 32 accounts. Timestamps in `DD-MM-YYYY HH:MM` (IST) format, parsed and aligned to daily dates. Zero missing values.
- **Merge**: Inner join on `date` yielding 479 matched days.
- **Key metrics constructed**: daily PnL per trader, win rate, trade frequency, average & total position size (USD), long/short ratio, drawdown proxy (worst daily PnL), leverage tier (by median trade size quartile).

### Analysis (Part B)
- Aggregated performance (PnL, win rate) and behavior (frequency, size, long ratio) by sentiment phase.
- Three trader segments: Frequent vs Infrequent (by total trade count tertiles), Small vs Large (by avg position size tertiles), Winner vs Loser (by total PnL tertiles).
- Time-series visualization of PnL vs F&G Index to identify phase-level patterns.

### Predictive Model (Bonus)
- Gradient Boosting Classifier trained on per-(account, day) features to predict next-day profitability.
- Features: n_trades, total_pnl, win_rate, avg_size_usd, long_ratio, F&G value, sentiment encoding.
- Train/test split: 80/20, stratified. Evaluated with precision/recall/F1 and confusion matrix.

---

## Key Insights

| # | Insight |
|---|---------|
| 1 | **Fear days depress PnL** — median daily PnL is measurably lower under Fear sentiment, and the distribution shows a fatter left tail (larger losses). |
| 2 | **Traders remain long-biased even during Fear** — long ratio stays above 50% even when the market is fearful, creating a systematic mismatch that hurts returns. |
| 3 | **Frequent traders are more resilient** — high-frequency traders maintain better win rates across all sentiment phases compared to infrequent traders. |
| 4 | **Large-position traders amplify Fear-day drawdowns** — outsized positions generate the worst single-day losses during Fear periods. |
| 5 | **Neutral sentiment is the most stable alpha environment** — win rates and PnL are most consistent; Extreme Fear and Extreme Greed both introduce noise. |

---

## Strategy Recommendations (Part C)

### Strategy 1 — Sentiment-Calibrated Position Sizing
**Rule:** Scale position size inversely with the Fear/Greed index value.
- **Fear days (F&G < 40):** Reduce position size by 30–40% from baseline. Risk/reward skews negatively — drawdown risk is elevated.
- **Greed days (F&G > 60):** Allow full or slightly above-baseline sizing; win rates and median PnL improve.
- **Target:** Primarily large-position traders (top size tertile), who suffer the most on Fear days.

### Strategy 2 — Frequent Traders: Fade the Long Crowd on Extreme Fear
**Rule:** For high-frequency traders, tilt toward **short** bias during Extreme Fear (F&G < 30) rather than buying dips.
- The data shows traders collectively go long even when the index signals extreme fear — and lose. Contrarian short positioning during these periods captures the downside momentum.
- **Rule:** When F&G < 30, target short-bias >60% of trades and shorten average hold time to reduce drawdown exposure.

### Bonus Rule of Thumb
> *"Neutral days are the hidden edge."* Reserve highest-conviction trades for Neutral sentiment windows. Avoid initiating large positions during Extreme Fear or Extreme Greed spikes unless a clear edge exists.

---

*Submitted by:Aadya Rajesh*
