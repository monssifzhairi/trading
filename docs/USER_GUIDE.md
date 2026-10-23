# User Guide — MONSSIF QUANT EDGE V3

Analytical & paper-trading tool. Outputs are analysis, **not financial advice**;
a reference entry is the activation bar's close, **not a guaranteed fill price**.
The score is a configurable heuristic (default /100), **not a win probability**.

## Setup lifecycle
IDLE → **CANDIDATE** (zone tap on a confirmed bar; zone + ID frozen, no levels yet)
→ **ACTIVE** (a later confirmed bar closes beyond the zone: long above demand top,
short below supply bottom — entry/SL/TP1/TP2 freeze, rounded to the symbol tick)
→ **TP1_REACHED** → **COMPLETED** (TP2) · **INVALIDATED** (SL touch, gap through SL,
or opposing BOS/CHoCH — if SL and TP are both touched in one bar, SL is assumed
first because OHLC cannot reveal the order) · **EXPIRED** (candidate window or
`setupExpiry` elapsed). Reasons appear in the dashboard **State · Reason** row.

## Dashboard (28 rows)
Market: structure/internal/event/range/liquidity/zones/imbalances/VWAP/regime/
RVOL·RSI/HTF. Setup & Risk (toggleable): bias, setup, state/ID + age, entry,
SL, TP1/TP2, risk (price · ×ATR · R). Diagnostics (toggleable): score + per-factor
breakdown, hard-filter states, warnings (missing ATR/RVOL/VWAP, HTF≤chart TF),
candidate rejection reason, confirmation notes (pivot lag, closed-HTF-bar rule).

## Alerts (two methods — use one per event)
- **alertcondition()** dropdown: `Candidate Bullish/Bearish Detected`,
  `Confirmed Long/Short Setup (ACTIVE)`, `Risk Levels Available`, `Setup
  Invalidated`, `Setup TP1 Reached`, `Setup Completed (TP2)`, `Setup Expired`,
  plus all structure/sweep/VWAP conditions. Static text with {{ticker}}/{{interval}}.
- **alert() dynamic**: create the alert and choose **"alert() function calls only"**
  to receive symbol, TF, setup ID, direction, and all frozen levels in the message.
  Events fire once per state transition, on confirmed bars. Alerts have **not**
  been live-tested yet (see TEST_PLAN.md).

## Strategy version
Separate file; entries only when a setup becomes ACTIVE, sized by
`Risk % equity ÷ risk distance`, TP1 partial exit %, TP2/SL bracket, position
closed on invalidation/expiry. Commission 0.05% + 2 ticks slippage are defaults
in the `strategy()` line and can be overridden in Strategy Tester → Properties.
Backtests are hypothetical: intrabar order, gaps, liquidity and latency are
approximated by TradingView's broker emulator; validate out-of-sample and
paper-trade. Do not tune weights to maximise historical profit.

## Key settings
`candMaxBars` confirmation window · `setupExpiry` · `atrBuffer` SL buffer ·
`maxRiskAtr` rejection cap (0=off) · `tp1R/tp2R` targets · hard filters
(MTF/RVOL/ADX/VWAP, default OFF) · score weights (default max 100) ·
display toggles never change calculations.
