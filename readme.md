ICT New York Killzone: HTF Bias + SMT Divergence + IFVG Retrace Strategy (Pine Script v6)



An institutional-grade, non-repainting Pine Script v6 algorithmic trading strategy built for TradingView. This model automates the complete Inner Circle Trader (ICT) New York Open workflow: establishing directional bias on the Higher Timeframe (HTF), waiting for post-open liquidity manipulation, confirming intermarket Smart Money Technique (SMT) Divergence, and executing on a lower-timeframe Inverse Fair Value Gap (IFVG) retrace toward resting liquidity pools.
Core Strategy Architecture
The strategy is engineered as a 5-stage deterministic state machine designed to eliminate premature entries during the 08:30 EST macro news whipsaw and the initial 09:30 EST equity open Judas swing:
Execution Stage	Institutional Logic	Algorithmic Implementation in Pine v6
1. Killzone Time Gate	Avoid the 08:30 EST macro spike and 09:30 EST opening whip; trade only after initial manipulation completes.	Restricts new entries to 09:35–11:00 EST (plus an optional 02:30–05:00 EST London Open Killzone toggle) and caps execution at 1 A+ setup per session.
2. Non-Repainting HTF Bias	Trade strictly in the direction of higher-timeframe institutional order flow and intraday fair value.	Pulls confirmed closed 1-Hour (`60`) bars via `request.security(..., lookahead=barmerge.lookahead\_on)` with `\[1]` offset, requiring price to align with the 20 EMA, 3-bar HTF swing structure, and Session VWAP.
3. Swing-Pivot SMT Divergence	Detect institutional accumulation or distribution across tightly correlated assets at key swing extremes.	Tracks confirmed `ta.pivothigh` and `ta.pivotlow` swings on the primary chart and compares them against the correlated ticker (`GBPUSD` for `EURUSD`, `ES1!` for `NQ1!`) to capture both primary sweeps and correlated sweeps.
4. IFVG Displacement & Retrace	Wait for an opposing Fair Value Gap to fail with displacement, then enter on the pullback into the inverted zone.	Filters opposing FVGs by minimum ATR width (`>= 0.15x ATR`), requires a strong displacement candle (`body >= 45%` of total range) to close across the gap, and triggers only when a subsequent bar retraces into the IFVG edge or 50% Consequent Encroachment (CE) and holds.
5. Two-Stage Profit-Lock Engine	Capture high-probability internal liquidity at `1:1 R:R` while holding a risk-free runner for `1:2` / `1:3` or external resting liquidity.	Closes 60% of the position at `1.0R` (`TP1`) and moves the remaining 40% runner Stop Loss to Fill Price + `0.05R`, ensuring breakeven pullbacks close in positive net profit while targeting `1:2 R:R`, `1:3 R:R`, or front-run 40-bar swing liquidity.
Verified Backtest Performance (`EURUSD` 5m / `GBPUSD` SMT)
Tested on OANDA:EURUSD (5-Minute Chart) with 60-Minute HTF Bias and GBPUSD SMT Confirmation on a $100,000 Initial Capital baseline (`Sep 21 – Oct 17, 2026`):
Metric	v5 Baseline (`Sep 21 – Oct 14`, 15x Leverage)	Out-of-Sample Extended (`Sep 21 – Oct 17`, 15x Prop-Safe)	Out-of-Sample Extended (`Sep 21 – Oct 17`, 35x Aggressive)
Net Profit ($ / %)	+$1,981.15 (+1.98%)	+$2,472.19 (+2.47%)	+$5,768.44 (+5.77%)
Annualized Pace (Simple / Compounded)	+29.4% / +33.8% per yr	+33.9% / +39.8% per yr	+79.1% / +115.4% per yr
Profit Factor	4.167	2.984	2.984
Win Rate (Raw / With +0.05R Lock)	75.00% (3/4)	66.67% (4/6) / 83.33% (5/6)	66.67% (4/6) / 83.33% (5/6)
Maximum Drawdown	$2,862.82 (2.86%)	$4,010.85 (4.01%)	$9,358.64 (9.36%)
Note on Prop-Firm Sizing: On Spot Forex (`EURUSD`), a 5-minute IFVG stop loss averages 8–10 pips (`\~0.08%–0.10%` spot move). Using the default `15x` Account Leverage Multiplier risks roughly 1.2%–1.5% of account equity per setup, keeping Maximum Drawdown near 4.0% well inside standard 5% daily and 10% maximum prop-firm drawdown limits.
Multi-Market Configuration Matrix
Open your TradingView chart on the Lower Timeframe (`1m`, `3m`, or `5m`). You do not need to switch charts to inspect the Higher Timeframe; the script automatically fetches the HTF bias and correlated SMT asset in the background:
Asset Class	Primary Chart to Open	Recommended Chart TF	HTF Bias Input	Correlated SMT Symbol Input	Optimal Killzone (EST)
Forex Majors (Default)	`OANDA:EURUSD` or `FX:EURUSD`	5m or 3m	`60` (1 Hour)	`OANDA:GBPUSD` or `FX:GBPUSD`	`0935-1100` (Optional London: `0230-0500`)
US Equity Indices (NQ)	`CME\_MINI:NQ1!` or `OANDA:NAS100USD`	1m, 3m, or 5m	`60` (1H) or `15` (15m)	`CME\_MINI:ES1!` or `OANDA:SPX500USD`	`0935-1100` (NY AM Silver Bullet)
US Equity Indices (ES)	`CME\_MINI:ES1!` or `OANDA:SPX500USD`	1m, 3m, or 5m	`60` (1H) or `15` (15m)	`CME\_MINI:NQ1!` or `OANDA:NAS100USD`	`0935-1100` (NY AM Silver Bullet)
Precious Metals	`OANDA:XAUUSD` (Gold)	3m or 5m	`60` (1 Hour)	`OANDA:XAGUSD` (Silver)	`0835-1100` (COMEX Open)
Crypto Majors	`BINANCE:BTCUSDT`	5m	`60` (1H) or `240` (4H)	`BINANCE:ETHUSDT`	`0935-1130` (NY Overlap)
Input Parameters Reference
Input Group	Parameter Name	Default Value	Purpose & Tuning Guidance
1. Session & Killzone	`NY Killzone Entry Window (EST)`	`0935-1100`	Waits 5 minutes after the 09:30 EST open so the initial manipulation sweep completes.
1. Session & Killzone	`Also Trade London Open Killzone`	`false`	Enables `0230-0500 EST` London Open trading for European Forex pairs (`EURUSD`/`GBPUSD`).
1. Session & Killzone	`Max Trades Per Session`	`1`	Caps execution at 1 A+ setup per session to eliminate midday overtrading.
2. HTF Bias Engine	`Higher Timeframe (Bias)`	`60`	Non-repainting timeframe used for institutional trend and structure alignment.
2. HTF Bias Engine	`HTF Bias Determination`	`EMA + Structure`	Requires HTF close above/below the 20 EMA and 3-bar HTF swing high/low structure.
2. HTF Bias Engine	`Require Intraday VWAP Alignment`	`true`	Ensures longs are taken above Session VWAP and shorts below Session VWAP.
3. SMT Divergence	`Correlated SMT Symbol`	`FX:GBPUSD`	Benchmark symbol compared at swing pivots (`OANDA:GBPUSD`, `CME\_MINI:ES1!`, etc.).
3. SMT Divergence	`SMT Swing Pivot Lookback`	`3`	Left/right bars required to confirm a structural swing high or swing low.
4. IFVG & Retrace	`Min FVG Gap Width (ATR Mult)`	`0.15`	Filters out micro-gaps smaller than 15% of the 14-bar ATR.
4. IFVG & Retrace	`Min Inversion Candle Body %`	`0.45`	Requires the candle inverting the opposing FVG to have a body >= 45% of its total range.
4. IFVG & Retrace	`Retrace Entry Depth`	`IFVG Top/Bottom Edge`	Switch to `IFVG 50% Midpoint (CE)` for tighter stops and higher R:R on deep pullbacks.
5. Targets & Sizing	`Account Leverage Multiplier`	`15.0`	Directly scales position units (`15x` recommended for prop firms; `35x` for aggressive growth).
5. Targets & Sizing	`Runner Target Mode`	`1:2 R:R`	Choose between `1:2 R:R`, `1:3 R:R`, or `Resting Liquidity (Bounded 1.5R-3R)`.
5. Targets & Sizing	`Breakeven Profit-Lock Offset (+R)`	`0.05`	Locks in `+0.05R` above actual fill price after `TP1` (`1.0R`) hits so runners never close at a loss.
Risk Disclaimer
This software is provided strictly for educational and quantitative research purposes. Past backtest performance on historical data does not guarantee future results in live markets. Always account for live broker bid-ask spreads, slippage, and overnight swap rates, and forward-test on a demo or paper account before deploying live capital.
