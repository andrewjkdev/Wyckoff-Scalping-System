# Wyckoff Liquidity-Sweep Scalping System — research notebook

A self-contained, reproducible study of whether a **Wyckoff-style shake-out (liquidity sweep and reclaim) inside a trend pullback** is a tradable scalping edge on **1-minute bars of liquid US ETFs**, and whether a **machine-learning meta-labeler** improves trade selection over the rules-only version.

Everything lives in **[`research.ipynb`](research.ipynb)**, which runs top to bottom on a machine with ≤ 4 GB of RAM (measured peak: 1698 MB).

## Verdict

**There is no tradable edge after costs.** Rules-only lost money out of sample in **6 of 6** IS/OOS splits at 1× costs, and in the stitched anchored walk-forward (-0.067R per trade, 95% CI -0.160 to +0.025). The setups carry some information (they beat random entries with the same geometry), but less than the round-trip cost of a 1-minute bracket. After correcting for the 2100 configurations searched, no in-sample winner is significant. The ML meta-labeler improved on rules-only in 5 of 6 walk-forward folds, but its stitched OOS expectancy (+0.105R, 95% CI -0.044 to +0.259) includes zero, its out-of-fold AUCs are at chance level, and it is negative at 2× costs. **Overfitting/integrity checks passed: 4 of 10.**

The full reasoning, with every number checked automatically against the notebook outputs, is in notebook §13.

### Out-of-sample results by split (net of costs; R = initial risk per trade)

| Split | OOS trades (rules) | Rules exp. R | Rules Sharpe | Rules max DD | ML exp. R | ML Sharpe | Rules exp. R @2× | ML exp. R @2× |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| IS1y | 4776 | -0.158 | -1.17 | -87.8% | — | — | -0.329 | — |
| IS2y | 5626 | -0.115 | -0.85 | -82.3% | — | — | -0.288 | — |
| IS3y | 3825 | -0.164 | -0.93 | -87.9% | -0.164 | -0.93 | -0.420 | -0.420 |
| IS4y | 1756 | -0.113 | -0.32 | -55.5% | — | — | -0.376 | — |
| IS5y | 1585 | -0.099 | -0.69 | -50.8% | — | — | -0.267 | — |
| HALF | 686 | -0.047 | -0.20 | -16.2% | +0.005 | -0.04 | -0.187 | -0.135 |
| WF stitched | 1319 | -0.067 | -0.30 | -33.4% | +0.105 | +0.28 | -0.244 | -0.080 |

*Splits: IS = the first 1–5 years from the data start, or the first half (HALF); OOS = everything after. "WF stitched" concatenates the six non-overlapping 3-year OOS segments of an anchored walk-forward (re-optimized every 3 years). ML is not fit when a split has fewer than 150 IS trades; when it keeps 100% of trades its row equals rules-only. Sharpe and Sortino are annualized from daily portfolio returns at 0.5% risk per trade.*

### Overfitting and integrity checks (primary split = HALF unless stated)

| Check | Verdict | Detail |
|---|---|---|
| Placebo: random entries | **PASS** | real OOS expectancy -0.047R, placebo 95th pct -0.226R, p = 0.004 |
| Placebo: time-shifted signals | **FAIL** | real -0.047R vs shifted 95th pct +0.194R, p = 0.223 |
| Placebo: shuffled ML labels | **FAIL** | real ML OOS +0.005R vs shuffled 95th pct +0.088R (p = 0.238); rules-only -0.047R |
| Deflated Sharpe Ratio | **FAIL** | HALF DSR = 0.000 (N=250); 0/12 splits/folds have DSR >= 0.95 |
| White's Reality Check / Hansen SPA | **FAIL** | HALF: Reality Check p = 1.000, SPA p = 0.962 |
| Probability of Backtest Overfitting (CSCV) | **PASS** | HALF PBO = 0.246; splits with PBO < 0.5: 9/12 |
| Parameter stability (plateau) | **PASS** | selected IS Sharpe 0.29; 78% of 23 neighbours keep >= 50% of it (median neighbour 0.23) |
| Monte Carlo (bootstrap Sharpe CI) | **FAIL** | rules-only bootstrap Sharpe 90% CI [-0.70, 0.30]; ML [-0.57, 0.45] |
| Cross-sectional consistency | **FAIL** | 3/8 ETFs profitable OOS; sign-test p = 0.855 |
| Lookahead audit | **PASS** | 32/32 audit checks passed |

### VIX regimes (primary split, rules-only, OOS)

| Prior-day VIX | Trades | Exp. R | 95% CI |
|---|---:|---:|---|
| <15 | 70 | -0.360 | [-0.659, -0.054] |
| 15-20 | 256 | -0.015 | [-0.195, +0.167] |
| 20-30 | 298 | -0.064 | [-0.238, +0.102] |
| >30 | 62 | +0.260 | [-0.164, +0.694] |

Fixed per-share costs are a larger fraction of a tight stop in calm markets, so losses concentrate when the prior-day VIX is low. No bucket's interval lies entirely above zero.

### Macro events, news uncertainty and geopolitical risk (primary split, rules-only, OOS)

FRED release calendars (CPI, NFP, PCE, GDP, retail sales, PPI), EPU and EMV were available; FOMC statement dates come from the Federal Reserve's calendar pages, because FRED's FOMC release is a near-daily data feed. Release days look less bad than other days, but every interval includes zero, and the FOMC ±30-minute window has too few trades to judge. Every IS-chosen event filter is treated as a hypothesis; 0 of 10 were confirmed out of sample.

| Dimension | Bucket | Trades | Exp. R | 95% CI | Reliable (≥30 trades) |
|---|---|---:|---:|---|---|
| event | non-event day | 467 | -0.129 | [-0.263, +0.014] | yes |
| event | any event day | 219 | +0.129 | [-0.074, +0.330] | yes |
| event | CPI | 27 | -0.040 | [-0.568, +0.529] | **no** |
| event | NFP | 38 | +0.030 | [-0.428, +0.551] | yes |
| event | FOMC | 26 | -0.169 | [-0.662, +0.404] | **no** |
| event | PCE | 43 | +0.246 | [-0.247, +0.724] | yes |
| event | GDP | 31 | +0.206 | [-0.387, +0.830] | yes |
| event | RETAIL | 32 | +0.150 | [-0.308, +0.647] | yes |
| event | PPI | 43 | +0.241 | [-0.215, +0.689] | yes |
| intraday | FOMC ±30min | 4 | -0.145 | [-1.215, +1.947] | **no** |
| intraday | FOMC day, outside ±30min | 22 | -0.174 | [-0.693, +0.449] | **no** |
| intraday | FOMC day, after release | 1 | -1.149 | [-1.149, -1.149] | **no** |
| intraday | 08:30 releases ±30min | 0 | — | — | structurally empty: window 08:00-09:00 is before the RTH open |
| EPU tercile | T1 low | 118 | +0.129 | [-0.165, +0.442] | yes |
| EPU tercile | T2 mid | 222 | -0.219 | [-0.397, -0.032] | yes |
| EPU tercile | T3 high | 346 | +0.004 | [-0.142, +0.170] | yes |
| EMV tercile | T1 low | 103 | -0.179 | [-0.430, +0.096] | yes |
| EMV tercile | T2 mid | 210 | -0.047 | [-0.257, +0.172] | yes |
| EMV tercile | T3 high | 373 | -0.010 | [-0.161, +0.145] | yes |
| GPR tercile | T1 low | 159 | -0.080 | [-0.297, +0.150] | yes |
| GPR tercile | T2 mid | 162 | +0.153 | [-0.092, +0.397] | yes |
| GPR tercile | T3 high | 365 | -0.121 | [-0.266, +0.035] | yes |
| EPU tercile (lag 2) | T1 low | 120 | -0.082 | [-0.334, +0.183] | yes |
| EPU tercile (lag 2) | T2 mid | 214 | -0.254 | [-0.435, -0.067] | yes |
| EPU tercile (lag 2) | T3 high | 352 | +0.091 | [-0.076, +0.267] | yes |
| GPR tercile (lag 2) | T1 low | 198 | -0.029 | [-0.246, +0.184] | yes |
| GPR tercile (lag 2) | T2 mid | 140 | +0.015 | [-0.229, +0.271] | yes |
| GPR tercile (lag 2) | T3 high | 348 | -0.081 | [-0.236, +0.072] | yes |

## What the notebook does

| § | Section | Highlights |
|---|---|---|
| 1 | Research summary | The 405-page Wyckoff course, read in full and summarized with page references. The book's own terms (**shake-out**, test on shrinking volume, normal ≤50% reaction, buying/selling waves, effort vs. result) are mapped to testable 1-minute rules, noting that the modern Phase A–E schematics are *not* in this text. MIT sources and other literature, every citation independently verified. |
| 2 | Config | Paths, universe rules, session, cost model, risk rules, splits, search budgets, seeds, memory instrumentation. |
| 3 | Data loading & cleaning | Per-symbol cleaning to parquet, re-gridding of missing minutes, a cost-adjustment factor for back-adjusted prices, **a correction for the 2022-03-07 price seam**, causal liquidity tiers. |
| 4 | Feature engineering | ATR, relative volume vs. same minute of day (within each volume-source segment), CLV/delta/effort-result/wick **proxies**, HTF trend from completed bars only, prior-day VIX. |
| 5 | Signal generation | A numba state machine: causal zigzag waves → pullback depth → sweep of a swing level → reclaim → volume-confirmation recipe → stop/target. |
| 6 | Backtest engine | Next-bar-open entries, stop-first on ambiguous bars, gap fills, time stop, flat by 15:55, cost in R, 4× per-position and gross leverage caps, latched −2R daily loss limit. 13 unit tests. |
| 7 | Parameter search | Seeded Optuna TPE on IS only, one search per split/fold, **every trial logged** (2100 total). |
| 8 | ML meta-labeling | Triple-barrier labels aligned with the strategy, purged and embargoed date-block CV, logistic regression / random forest / XGBoost, SHAP and permutation importance. |
| 9 | IS/OOS evaluation | OOS touched once per candidate; 1× and 2× costs; anchored walk-forward; equity and drawdown curves; annotated example trades. |
| 10 | Overfitting checks | Placebos (random entries, time shifts, shuffled labels), DSR and MinBTL, Reality Check and SPA, PBO by CSCV, parameter heatmaps, Monte Carlo, cross-section, automated lookahead audit (truncation tests). |
| 11 | VIX regimes | By level and percentile, with bootstrap CIs and a gross-vs-cost decomposition. |
| 12 | Macro events | FRED release calendars and Fed FOMC statement dates aligned to release times (ET), FOMC ±30 min, EPU/EMV/GPR terciles cut on IS only, event filters as IS-chosen / OOS-confirmed hypotheses. |
| 13 | Conclusions | Metrics and verdict tables, a narrative, and a **result-claim audit** that re-derives the headline metrics and checks every number in the prose against the run's outputs. |

## Data

* **1-minute OHLCV**: `/Users/andrew/main/trading/data/etfs/{SYMBOL}_clean_1min.csv`, 11 ETFs (SPY, QQQ, IWM, DIA, TLT, IEF, SHY, GLD, SLV, USO, CPER), 2002-12-30 → 2026-09-21. VXX is excluded as VIX-linked. None of the ETFs was delisted.
* **Known data issues handled in §3.** On 2022-03-07 the source switches from consolidated (pitrading) to IEX-only volume. Pre-switch prices are dividend-adjusted while post-switch prices are raw, which created a fake overnight gap that is corrected. Thin ETFs have sparse IEX prints and are gated by prior-day coverage.
* **Daily data**: ^VIX and split history from yfinance; FRED (release dates, `USEPUINDXD`, `EMVOVERALLEMV`) via `fredapi`/FRED API; FOMC statement dates from federalreserve.gov; the Caldara–Iacoviello GPR daily index from its official site.
* With no bid/ask, tape or footprint data, **all order-flow quantities are OHLCV proxies**.

## Running it

```bash
cd /Users/andrew/main/programming/projects/Wyckoff-Scalping-System
/opt/anaconda3/bin/jupyter nbconvert --to notebook --execute --inplace research.ipynb --ExecutePreprocessor.timeout=-1
```

* Kernel: the anaconda `python3` kernel with pandas, numpy, numba, pyarrow, scipy, scikit-learn, xgboost, optuna, shap, plotly, yfinance, fredapi, psutil, nbformat.
* `FRED_API_KEY` (optional) is read from the environment for §12 and never written to the notebook. Without it, §12 reports the FRED series as unavailable.
* `WYCKOFF_FAST=1` runs a quick smoke test (3 ETFs, tiny budgets). `WYCKOFF_REBUILD=1` rebuilds every cache.
* A full run from an empty cache takes about half an hour on an Apple-silicon laptop; with the search cached it takes under 10 minutes.

## Repository layout

| Path | Contents |
|---|---|
| `research.ipynb` | The notebook (executed, with outputs and Plotly charts). |
| `artifacts/wyckyoff_method_book.pdf` | The Wyckoff course (primary source for §1). |
| `artifacts/sources/` | Local copies of the openly available cited sources plus `MANIFEST.md` (paywalled works are linked only). Only the manifest is tracked by git, because many files carry publisher or MIT copyright. |
| `cache/external/` | Downloaded historical data (yfinance, FRED, GPR). |
| `cache/v6/` | Derived caches: cleaned bars, features, trial logs, trade logs, result tables. Versioned, so a code change can't silently reuse stale results. |

## Reproducibility and honesty

* Fixed seeds (recorded in the notebook) and a deterministic seeded TPE search; versions are printed at the start and end of the run.
* Optimization uses IS data only. Each split's single candidate touches OOS once, and nothing was re-tuned after OOS results were seen.
* Every PASS/FAIL threshold was fixed in code before the full run.
* An independent five-lens code review (leakage, engine, statistics, signal kernel, data/memory), with two adversarial verifiers per finding, confirmed 16 defects. All were fixed before the reported run, including the price seam at the source switch, a sweep that could carry across sessions, and loss-limit and leverage-cap logic.

*Research only, not investment advice.*
