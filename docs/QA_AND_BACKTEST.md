# MONSSIF QUANT EDGE v2 — Changelog, QA Report & User Guide

> Analytical / educational system. Signals are analytical outputs, **not**
> guarantees, not personalized financial advice, and not execution prices.
> The quality score is a heuristic confluence count — **not a win probability**
> (it has never been statistically calibrated).

## 1. Audit of v1 (676 lines) — performed before any edit

| # | Finding | Severity | v2 action |
|---|---|---|---|
| A1 | FVG creation was gated by `showFvg` (a *display* toggle): hiding FVGs silently zeroed the FVG score points, dashboard count and inversion alerts | Logic bug | Logic always tracked; `showFvg` now only controls box/text transparency (incl. inversion recolouring) |
| A2 | Risk lines were drawn per-tap with no frozen-level contract, no invalidation/completion tracking; taps from different OBs could each draw overlapping level sets | Design bug | Full setup state machine (below); one active setup; levels frozen at creation |
| A3 | Premium/Discount map deleted + recreated 8 drawing objects on **every bar** | Performance | Redrawn only on `barstate.islast` |
| A4 | No setup identity/state; tap alerts could repeat across zones; `alertcondition` messages are static | Design | Setup IDs, one-shot state events, plus dynamic `alert()` messages with all levels |
| A5 | Dashboard had no Bias / Setup / Entry / SL / TP / RR rows and the score was unexplained | UX | Panel expanded 15 → 24 rows incl. `Score Parts` breakdown |
| A6 | Pivot confirmation lag (`sLen`/`iLen` bars) | Inherent | Unchanged — documented, this is confirmation delay, not repainting |
| A7 | EQH/EQL lines extended on every realtime tick | Minor perf | Left as-is (cheap `set_x2`); noted |
| A8 | `atrRank` is `na` during warm-up → regime label approximate on early bars | Minor | Dashboard-only impact; documented |
| A9 | HTF uses closed bar `[1]` + `lookahead_off`; VWAP-cross alerts gated on `barstate.isconfirmed` | OK | Preserved unchanged |
| A10 | OB creation is still gated by `showOB` (turning OB display off disables taps) | Known semantics | **Left unchanged deliberately** — changing it would alter existing user behaviour; documented here as a display/logic coupling to be aware of |

Preserved untouched: visual identity (colors, dark dashboard theme), all
original inputs & defaults, swing/internal structure, OB/FVG/liquidity
life-cycles, the original 100-pt core score weights, premium/discount map,
narrative, and all pre-existing alerts. New filters still default OFF.

## 2. Setup state machine (new)

States: `IDLE → ACTIVE (POTENTIAL LONG/SHORT, levels frozen) →` one-bar terminal
events `INVALIDATED / COMPLETED(TP2) / EXPIRED → IDLE`.

- **Creation:** only on a confirmed zone tap while IDLE (filters, if enabled,
  must pass). Validated before creation: `risk > 0`, `ATR > 0` and not `na`,
  and `risk ≤ Max Risk × ATR` if that cap is set (>0). Invalid candidates are
  rejected silently (no half-drawn levels).
- **Frozen levels:** Reference Entry = confirmed tap-bar close; SL = zone edge
  ∓ `SL Buffer × ATR` (long: demand bottom − buffer; short: supply top +
  buffer); TP1 = Entry ± `tp1R × Risk`; TP2 = Entry ± `tp2R × Risk`. By
  construction RR(TP1)=`tp1R`, RR(TP2)=`tp2R`; dashboard also shows risk in
  price and ATR units. Levels **never move** after creation.
- **Per-bar evaluation (bars after creation, confirmed bars only), in
  deliberate conservative order:** 1) SL touched (bar trades at/beyond SL) or
  opposing swing BOS/CHoCH → INVALIDATED; 2) TP2 reached → COMPLETED (TP1
  flagged if not already); 3) TP1 reached → stays ACTIVE, flagged once;
  4) age ≥ `Setup Expiry` → EXPIRED. Because OHLC cannot reveal the intrabar
  order of SL vs TP touches, **SL wins ties** — documented pessimism.
- **Deduplication:** while ACTIVE, further taps (any zone) create nothing —
  no duplicate setups, labels, level sets, or setup alerts. The pre-existing
  per-zone "Demand/Supply Tap" alerts still describe zone taps themselves.
- **Drawings:** one level set (4 lines + 4 labelled prices incl. setup ID)
  created per setup, extended only while ACTIVE, frozen in place when the
  setup ends, replaced by the next setup. Caps from v1 remain in force.

## 3. Signal Analysis Panel (dashboard, 24 rows)

Rows 0–14: original dashboard (structure, internal, event, range, liquidity,
zones, imbalances, core score, narrative, VWAP side, regime+ADX, RVOL/RSI,
HTF, quality /140). New rows 15–23: **Market Bias**
(BULLISH/BEARISH/NEUTRAL, "weak" = structure only, no EMA/HTF support),
**Setup** (POTENTIAL LONG/SHORT or last terminal status), **State/ID** (+ bar
age vs expiry), **Ref Entry**, **SL/Invalidate**, **TP1/TP2**, **Risk·RR**
(price, ×ATR, R multiples), **BOS/CHoCH + RSI + regime**, **Score Parts**
(e.g. `Struct 25 · Range 25 · Zone 20 · FVG 15 · Sweep 0 · Trend 10 · HTF 10 ·
RVOL 0 · VWAP 10`). Score maximums: core 100 + confirmations 40 = 140.

## 4. Alerts — how to configure in TradingView

Two mechanisms, use **one** per alert to avoid duplicates:

1. **Alert conditions (dropdown):** open the script's Alert dialog → choose
   e.g. `Potential Bullish Setup`, `Potential Bearish Setup`,
   `Setup Invalidated`, `Risk Levels Available`, `Setup TP1 Reached`,
   `Setup Completed (TP2)`, `Setup Expired` (plus all v1 conditions).
   Messages contain `{{ticker}}`/`{{interval}}` placeholders; full levels are
   on the chart/dashboard.
2. **Dynamic messages:** create the alert, select this script, then choose
   **"alert() function calls only"**. You will receive one message per event
   containing symbol, timeframe, direction, setup ID, frozen Entry/SL/TP1/
   TP2, risk and quality score. Because all events are raised on confirmed
   bars and once per state transition, they do not repeat for the same event.
   Set alert frequency/expiration in the dialog as you prefer; "Once Per Bar
   Close" semantics are already enforced by `alert.freq_once_per_bar_close`.

## 5. Strategy version (`MONSSIF_QUANT_EDGE_Strategy.pine`)

Separate `strategy()` script — never mixed with `indicator()`. It shares the
engine and state machine, and adds simulated execution only (no live orders):

- Entries on the setup-creation bar close (`process_orders_on_close = true`;
  the signal is only known at that close — no future data is used).
  `pyramiding = 0` plus a `strategy.position_size == 0` guard prevent
  duplicate entries. HTF data uses closed bars (`[1]`, `lookahead_off`).
- Exits use the same frozen levels: TP1 limit for `TP1 Exit %` of position,
  TP2 limit / SL stop for the remainder; structure-invalidation or expiry
  closes any open simulated position (`strategy.close_all`).
- Position size = `Risk % equity ÷ (risk distance × point value)`.
- Costs are modelled in the `strategy()` declaration: **0.05 % commission,
  2 ticks slippage** — edit these in the declaration to match your market;
  they are assumptions, not measurements.
- Statistics: TradingView's Strategy Tester report (total trades, win rate,
  net profit, profit factor, max drawdown) plus Data-Window plots of
  net profit, win/loss/total trades, profit factor
  (`grossprofit / |grossloss|`), max drawdown and historical win rate.

**Backtesting limitations (read before believing any number):** historical
results are hypothetical; OHLC bars cannot reveal intrabar SL/TP order (this
system assumes the pessimistic order in the indicator, while TradingView's
broker emulator makes its own intrabar assumptions — indicator and strategy
outcomes can therefore differ); fills ignore gaps beyond the modelled
slippage, partial fills, liquidity and latency; pivot confirmation lag is
real and retained; parameters were **not** optimised — validate out-of-sample
(walk-forward / unseen symbols & periods) and paper-trade before any use.

## 6. QA performed (static — no Pine compiler exists in this environment)

✅ Read the full v1 source (676 lines) before editing.
✅ Balanced-delimiter and duplicate top-level-name scan on both v2 files.
✅ Exactly one `indicator(` in the indicator file, exactly one `strategy(` in
the strategy file; no `strategy.*` calls in the indicator file.
✅ No `lookahead_on` anywhere; single `request.security` uses `[1]` +
`lookahead_off`. All setup events derive from `barstate.isconfirmed` paths.
✅ Long/short symmetry reviewed line-by-line for SL/TP/risk sign conventions;
validation guards (`risk > 0`, ATR valid, risk cap) precede any use; dashboard
divisions guard `na`/zero ATR.
✅ Drawing lifecycle: table is 2×24, rows written 0–23; risk set is a fixed
4 lines/4 labels; all v1 object caps retained; PD map now last-bar only.
✅ No new external dependencies, functions, or APIs were invented — only
Pine v6 built-ins already used in v1 plus `strategy.*`, `alert()`,
`str.contains`, `strategy.max_drawdown`.

## 7. NOT verified — requires TradingView (do not treat as tested)

⚠️ **Compilation in the Pine Editor has NOT been run** — there is no Pine
compiler in this environment. First compile is a test: fix any v6 type errors
it reports (most likely candidates if any: `strategy.max_drawdown`
availability on your account's Pine version, tuple/ternary typing in the new
dashboard rows).
⚠️ Visual layout of the 24-row table, label overlap, and box/line caps on
long histories. ⚠️ Bar-Replay check that setups/HTF never shift historically.
⚠️ Strategy Tester fills, TP1 partial-exit interaction with the TP2/SL exit,
and any broker-emulator differences. ⚠️ Alert delivery (both mechanisms).

## 8. Changelog v1 → v2.0

- Setup state machine (IDLE/ACTIVE/INVALIDATED/COMPLETED/EXPIRED) with IDs,
  frozen validated risk levels, expiry, structure-flip invalidation.
- Risk drawings unified into one labelled frozen set per setup (was per-tap).
- Dashboard: +9 professional rows (Bias, Setup, State/ID, Entry, SL, TP1/TP2,
  Risk·RR, BOS/CHoCH·RSI·regime, Score Parts); header renamed to product name.
- Score refactored into named components (identical weights/total).
- Fix A1 (FVG display/logic decoupling), Fix A3 (PD map performance).
- New inputs: Setup Expiry, Max Risk ×ATR, SL Buffer ×ATR (replaces the
  hard-coded 0.25 ATR), Show Setup Labels. All prior inputs preserved.
- Alerts: 7 new one-shot conditions + dynamic detailed `alert()` messages.
- New separate strategy file for backtesting/simulation (commission &
  slippage modelled, risk-based sizing, Data-Window performance plots).
