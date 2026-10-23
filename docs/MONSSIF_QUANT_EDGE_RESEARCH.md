# MONSSIF QUANT EDGE — Source Research, Audit & Integration Design

> **Disclaimer:** This is a technical-analysis aid. Nothing here guarantees profitable
> trades. Backtest on historical data, paper-trade, and validate in TradingView's
> Pine Editor before any live use. No performance claims are made in this document.
>
> **Verified vs. assumed:** Facts obtained by inspecting the files in this repo or
> from the linked sources are marked *(verified)*. Everything else is an explicit
> assumption/recommendation. No source link, license, test result, or performance
> figure has been invented.

---

## 1. What is actually in this repository *(verified by inspection, 2026-10-23)*

| File | Pine | Lines | What it really contains | License header in file |
|---|---|---|---|---|
| `MarketStructure.pine` | v5 | 70 | Pivot BOS/CHoCH only (Persian input labels). **No inducements, no sweeps, no LuxAlgo attribution.** | None |
| `smart-money-concepts.pine` | v6 | 548 | "Smart Money Concepts Engine [Quantum Algo]": swing+internal BOS/CHoCH, order blocks, FVG+inversions, EQH/EQL, sweeps, premium/discount, confluence score (100 pt), dashboard, alerts. Header claims *"all code in this script are original work."* | **None (no LICENSE)** |
| `volume-profile.pine` | v6 | 185 | POC/VAH/VAL, 70% value area, optional up/down-candle split, optional intrabar mode | None |
| `pocket-pivot.pine` | v6 | 30 | Up-day volume > max down-day volume (10 bars), SMA-50 + rising-MA filters | None |

**Important correction to the brief:** the file described as
*"Market Structure with Inducements & Sweeps [LuxAlgo]"* is **not present** in
this repo. `MarketStructure.pine` is a 70-line v5 BOS/CHoCH script with no
LuxAlgo header and no inducement/sweep logic. Inducements/sweeps/OB/FVG are in
fact already covered — by `smart-money-concepts.pine`, not by a LuxAlgo file.
Do not rename or re-attribute these files without the authors' permission.

**License status of the repo itself:** none of the four files carries a license.
Under default copyright, "no license" ≠ free to redistribute. **Action required
from the repo owner:** add an explicit `LICENSE` for your own original code
(e.g. MIT or MPL-2.0) or keep it private/all-rights-reserved. This document
assumes *(assumption)* the Quantum Algo engine is the project owner's own work
and may be integrated; if it is a third party's, obtain permission first.

---

## 2. Missing-source research (verified links & licenses)

Conclusion up front: **none of the genuinely missing components needs any
third-party code.** All are one-line built-ins in Pine v6 (`ta.vwap`, `ta.dmi`,
`ta.rsi`, `ta.ema`, `ta.percentrank`, `request.security`). Implementing them
natively eliminates license risk entirely. The external sources below were
checked only as references / fallbacks.

| Candidate | Link | License *(verified from source)* | Verdict |
|---|---|---|---|
| LuxAlgo Smart Money Concepts (BOS/CHoCH, OB, FVG, EQH/EQL, premium/discount) | https://www.tradingview.com/script/CnB3fSph-Smart-Money-Concepts-SMC-LuxAlgo/ ; source header visible at https://gist.github.com/niquedegraaff/8c2f45dc73519458afeae14b0096d719 | **CC BY-NC-SA 4.0** (© LuxAlgo, in the script header) | **EXCLUDE from copying.** NonCommercial + ShareAlike would contaminate a commercial product. Already functionally duplicated by the in-repo engine anyway. Reimplementing SMC *concepts* independently (as the in-repo engine does) is the safe route. |
| TradingView built-in indicators / `ta.*` functions (VWAP, ADX, RSI, ATR…) | https://www.tradingview.com/pine-script-docs/welcome/ | TradingView's own built-ins are published under **MPL-2.0**; calling `ta.*` built-in *functions* in your own script copies no code at all | **USE (as API calls only).** This is how the integrated build implements VWAP/ADX/RSI/ATR/RVOL/MTF. |
| casoon/pine-scripts (v6 collection: VWAP bands, MTF framework, structure) | https://github.com/casoon/pine-scripts | **MIT** (repo states Pine v6 · MIT · WavesUnchained) | Optional reference only. Permissive, but unnecessary — native `ta.*` covers the need with less code to audit. Not copied. |
| amadeo987/tradingview-price-action (MSS/BOS, OB, FVG, VWAPs, 2-TF table) | https://github.com/amadeo987/tradingview-price-action | **MPL-2.0** (LICENSE in repo) | Optional reference. MPL-2.0 is file-level copyleft: reuse is legal with notices, but again unnecessary. Not copied. |
| Anchored VWAP variants (e.g. ak2k2/Custom-AAVWAP) | https://github.com/ak2k2/Custom-AAVWAP-Pinescript | **MPL-2.0** (GitHub license detection) | Exclude for now. Session VWAP is built-in; *swing-anchored* AVWAP is a possible future module (see §5). |
| Zeiierman/PineIndicators dynamic AVWAP | https://www.tradingview.com/script/gjFSGRCo-Dynamic-Swing-Anchored-VWAP-STRAT-Zeiierman-PineIndicators/ | **CC BY-NC-SA 4.0** (stated on page) | **EXCLUDE** — NonCommercial/ShareAlike, same problem as LuxAlgo. |

Licensing rules applied throughout: CC BY-NC-SA = attribution + **non-commercial
only** + derivatives under the same license → incompatible with a commercial
"MONSSIF QUANT EDGE" product. MPL-2.0 = reuse allowed, keep notices, modified
MPL files stay MPL. MIT = reuse allowed, keep copyright notice. No license =
treat as all-rights-reserved.

---

## 3. Feature-overlap & compatibility audit

### 3.1 Overlaps found

| Feature | Where it appears | Overlap / conflict |
|---|---|---|
| Swing BOS/CHoCH | `MarketStructure.pine` **and** `smart-money-concepts.pine` (swing tier) | **Duplicate.** The v5 file also lacks the v6 engine's already-broken flag: it re-derives direction from a single `trend` var and classifies every counter-break as CHoCH. Two scripts on one chart will draw conflicting BOS/CHoCH labels. → Keep only the v6 engine's version. |
| Internal structure | SMC engine only | No conflict; it is the context tier for the score. Keep. |
| Trend filter (MA) | `pocket-pivot.pine` (SMA-50 rising) **and** new EMA 50/200 module | Conceptual duplicate. Pocket Pivot keeps its own filter when used standalone; in the integrated build the EMA/ADX module is the single trend filter. |
| Relative volume | Pocket Pivot's "max down-day volume" **and** OB origin RVOL (`volume/volSma` in SMC) **and** new RVOL dashboard/filter | Three related but different measures. Integrated build standardises on `RVOL = volume / SMA(volume,20)`; the OB label keeps its per-origin RVOL. Documented, not merged, to avoid changing existing behaviour. |
| Volume Profile POC/VAH/VAL vs Premium/Discount vs VWAP | Three different "value" references | Complementary, not duplicates: VP = volume-at-price over lookback; P/D = position in swing range; VWAP = session volume-weighted mean. Risk is *chart clutter*, not math conflict → all default-off or dashboard-only in the integrated build. |
| Sweeps | SMC engine (swing + EQH/EQL sweeps) | The v5 MarketStructure file has none (despite the brief). No duplicate. |

### 3.2 Repainting & correctness findings *(verified by reading the code)*

- `MarketStructure.pine` (v5): break labels are drawn on the live bar with no
  `barstate.isconfirmed` gate — an intrabar close beyond the level that later
  retraces will still have printed. Lines/labels are never deleted → hits the
  500-object cap on long histories. **Recommendation: retire this file** (its
  logic is a subset of the v6 engine); do not port it to v6.
- `smart-money-concepts.pine` (v6): structure/sweep/FVG/OB events are correctly
  gated on `barstate.isconfirmed`; pivots are inherently lagged by `sLen`/`iLen`
  bars (confirmation delay, not repainting — pivots never change once printed).
  Object caps are implemented. ✅ Basis for the integrated build.
- `volume-profile.pine`: computes only on `barstate.islast`, so it draws the
  *current* lookback profile — historical profiles are not preserved (by design,
  but users must not read it as historical POC). "Buy/sell split" is an
  up/down-candle proxy, **not** true aggressor/delta data. Intrabar mode is
  bounded by TradingView's lower-TF history limits with a documented fallback.
  `volume` is `na` on some symbols (forex/CFD) → empty profile. Keep standalone;
  merging it into the integrated script would redraw 50 boxes/bar-group and
  compete for the same 500-box budget as OB/FVG. **Exclude from the merged file.**
- `pocket-pivot.pine`: signal can appear/disappear intrabar (today's volume and
  close are incomplete until close) — add `barstate.isconfirmed` if used for
  alerts. Its down-day test (`close[i] < close[i+1]`) and 10-bar loop are sound
  and non-repainting on confirmed bars. Keep standalone; a chart can only host
  so many overlays, and its signal is already expressible via the RVOL filter.

### 3.3 Pine-version compatibility

Target is **Pine v6** *(verified: 3 of 4 files already v6)*. `MarketStructure.pine`
is v5 syntax; it would compile as v5 but must not be pasted into a v6 script
without conversion (e.g. dynamic requests, `ta.*` signatures). The integrated
build stays v6 throughout and uses only v6-valid built-ins.

---

## 4. Final recommended architecture

**One integrated indicator + two standalone companions. Nothing copied from
CC BY-NC-SA sources.**

```
MONSSIF_QUANT_EDGE.pine        (v6, built from smart-money-concepts.pine)
├── Market structure (kept)    swing + internal BOS/CHoCH, confirmed bars only
├── Zones (kept)               Order Blocks (life-cycle), FVG + inversions
├── Liquidity (kept)           EQH/EQL, swing sweeps
├── Premium/Discount (kept)    swing-range position
├── Confluence score (kept)    original 100-pt `score` — semantics unchanged
├── NEW: Confirmation modules (original code, ta.* only, all toggleable)
│   ├── Session VWAP ±1σ/±2σ bands      (default: hidden, dashboard row on)
│   ├── Trend & momentum: EMA 50/200, ADX/DMI, RSI   (dashboard + filters)
│   ├── Volatility regime: ATR% percent-rank → Trending/Ranging/Volatile
│   ├── RVOL = volume / SMA(volume,20)  (dashboard + optional tap filter)
│   ├── MTF: HTF EMA trend from last CLOSED HTF bar ([1], lookahead off)
│   └── Extended quality: qScore = score + trend10 + MTF10 + RVOL10 + VWAP10 (/140)
├── NEW: Risk levels (on zone taps)  entry, SL = zone edge ∓ 0.25·ATR,
│                                    TP1 = 1.5R, TP2 = 2.5R (inputs), capped sets
├── NEW: Optional tap filters        MTF alignment / RVOL≥min / ADX≥thr /
│                                    VWAP side — ALL default OFF (no behaviour
│                                    change unless the user enables them)
└── Dashboard (extended 10 → 15 rows) + 4 new alerts (Q≥110 taps, VWAP crosses)

Standalone (unchanged):  volume-profile.pine, pocket-pivot.pine
Retired (recommended):   MarketStructure.pine  (duplicate of SMC swing tier)
Not integrated:          LuxAlgo SMC, any CC BY-NC-SA code, external VWAP libs
```

### Integrate / Optional / Exclude

| Component | Decision | Why |
|---|---|---|
| VWAP + σ bands | **Integrate, hidden by default** | Genuinely missing value reference; one built-in call; zero license risk |
| EMA/ADX/RSI trend-momentum | **Integrate (dashboard + optional filters)** | The engine had *no* classical trend/momentum confirmation |
| ATR% regime | **Integrate (dashboard)** | Cheap, prevents trading SMC taps blindly in dead/volatile regimes |
| RVOL | **Integrate (dashboard + optional filter)** | Standardises the three overlapping volume measures |
| MTF confirmation | **Integrate (dashboard + optional filter)** | Non-repainting via closed-HTF-bar `[1]` + `lookahead_off` |
| Risk levels SL/TP1/TP2 | **Integrate (on taps, capped)** | Was completely missing; derived from zone edge + ATR, in R multiples |
| Volume Profile | **Keep standalone, do NOT merge** | Box-budget contention (500), last-bar-only redraw cost |
| Pocket Pivot | **Keep standalone** | Distinct equity-style signal; merging adds clutter, no new info over RVOL |
| `MarketStructure.pine` v5 | **Exclude / retire** | Strict subset, repaint-prone labels, object leaks, v5 |
| LuxAlgo SMC / Zeiierman AVWAP | **Exclude** | CC BY-NC-SA 4.0 — NonCommercial + ShareAlike |
| Swing-anchored AVWAP, divergence engine, backtester/strategy conversion | **Future, optional** | Useful but each deserves its own audit; strategy conversion changes the product class |

---

## 5. Integration plan (as executed)

1. ✅ Inspected all four sources; identified target Pine **v6**.
2. ✅ Copied `smart-money-concepts.pine` → `MONSSIF_QUANT_EDGE.pine` as the base;
   originals untouched (no rewrite of working features).
3. ✅ Added confirmation modules with **original code calling only built-in
   `ta.*`/`request.security`** — no third-party source copied, so no new license
   obligations arise.
4. ✅ Preserved behaviour by default: every new *filter* defaults OFF, VWAP/EMA
   plots default hidden, original 100-pt `score` and tap conditions unchanged
   unless a filter is enabled. `qScore` (/140) is additive, shown in dashboard.
5. ✅ Non-repainting rules: HTF uses `[1]` closed bar + `lookahead_off`; taps
   already fire inside the engine's `barstate.isconfirmed` lifecycle; new VWAP
   cross alerts are gated on `barstate.isconfirmed`.
6. ⏳ **User step (cannot be done in this repo):** paste `MONSSIF_QUANT_EDGE.pine`
   into TradingView Pine Editor → "Add to chart". There is no Pine compiler in
   this environment, so **compilation on TradingView is unverified** — treat the
   first compile as a test, and check: (a) no v6 errors, (b) dashboard renders
   15 rows, (c) enabling each filter reduces (never increases) tap signals,
   (d) replay mode shows no HTF/VWAP history shifting.
7. ⏳ Owner step: add a `LICENSE` file clarifying rights to the original code
   before any public/commercial distribution.

### Known limitations of the integrated build

- Session VWAP anchors to the calendar day (`timeframe.change("D")`); for
  instruments with extended sessions this follows TradingView's day boundary,
  not a custom RTH session. On volume-less symbols VWAP degrades to an
  equal-weighted average (TradingView fallback) and RVOL is `na` — filters
  using RVOL then block taps only if enabled and RVOL is `na`… by design the
  RVOL filter treats `na` as *fail* (`not na(rvol) and rvol >= rvolMin`).
- HTF trend is a single-EMA state (bull/bear/neutral), not full HTF structure.
- Risk levels are *reference drawings* from the tap bar's close and zone edge;
  they do not account for gaps, slippage, or fees, and are not trade instructions.
- Pivot-based structure lags by the pivot length; scores are heuristic weights,
  not probabilities. **No component predicts or guarantees outcomes.**

---

## 6. Source list (all links checked 2026-10-23)

1. LuxAlgo SMC on TradingView — https://www.tradingview.com/script/CnB3fSph-Smart-Money-Concepts-SMC-LuxAlgo/ (CC BY-NC-SA 4.0 per script header, mirrored at https://gist.github.com/niquedegraaff/8c2f45dc73519458afeae14b0096d719)
2. Pine Script v6 docs — https://www.tradingview.com/pine-script-docs/welcome/
3. casoon/pine-scripts (MIT, v6) — https://github.com/casoon/pine-scripts
4. amadeo987/tradingview-price-action (MPL-2.0) — https://github.com/amadeo987/tradingview-price-action
5. ak2k2/Custom-AAVWAP-Pinescript (MPL-2.0) — https://github.com/ak2k2/Custom-AAVWAP-Pinescript
6. Zeiierman/PineIndicators AVWAP (CC BY-NC-SA 4.0) — https://www.tradingview.com/script/gjFSGRCo-Dynamic-Swing-Anchored-VWAP-STRAT-Zeiierman-PineIndicators/
