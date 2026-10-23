# MONSSIF QUANT EDGE — Root-Cause Audit (v2 → V3)

Scope: `MONSSIF_QUANT_EDGE.pine` v2 (834 lines) + strategy twin, read in full.
v2 is preserved in `archive/` and git history (fb1ce40 → 041f8d7). **No TradingView
compiler output, screenshots or user failure logs were provided for v2**, so defects
below are from code inspection, prioritised by impact. Only confirmed compiler
feedback on record: CE10272 `strategy.lostrades`, SHORT_TITLE_TOO_LONG, CW10003,
CW10002 (all fixed in 041f8d7 and re-verified in V3).

| # | Defect (v2) | Root cause | V3 fix | Test | Remaining limitation |
|---|---|---|---|---|---|
| R1 | Hiding Order Blocks silently disabled zone taps, setups and (in strategy) entries | `f_makeOB` gated zone *creation* on `showOB` | Logic always runs; hidden zones are fully transparent boxes | Static scan: no `showOB` in logic conditions | Hidden zones still consume box objects |
| R2 | Hiding signal labels disabled setup creation entirely | Candidate/activation was inside `if showSigs and …` | `showSigs` gates only the label push; filters+structure gate logic | Static scan PASS | — |
| R3 | Hiding EQH/EQL removed liquidity data used by score/dashboard | EQ creation gated on `showLiq` | Always tracked; transparent when hidden | Static scan | Consumes line objects |
| R4 | No CANDIDATE state: a tap instantly became an "active" setup with levels | Single-step creation in the OB loop | Tap → CANDIDATE; activation requires a later confirmed close beyond the zone within `candMaxBars`; else INVALIDATED/EXPIRED with reason | State-transition table in TEST_PLAN (TV verification pending) | Confirmation rule is a design choice, not backtest-optimised |
| R5 | Risk levels not tick-rounded; no gap handling; rejection was silent | Raw float math; silent `if valid` | `f_tick()` rounds frozen Entry/SL/TP to `syminfo.mintick`; gap-through-SL / gap-beyond-TP2 handled explicitly; rejections recorded in `lastReject` + dashboard State·Reason row | Static: `f_tick` present, ordering validated at activation | Display rounding may make tiny risks 0 on coarse-tick symbols → rejected (by design) |
| R6 | Score double-counted correlated evidence (EMA trend + HTF stacked; structure counted twice across core+quality, max 140 with arbitrary 110 threshold) | Additive buckets added in two phases | Single score (default max 100), EMA+HTF share one capped bucket, all weights are inputs, dashboard shows `points/max` per factor, high-score alert is relative (≥80% of configured max) | Static + doc review | Weights remain heuristic — configurable, explicitly not probabilities |
| R7 | `showRisk`/`showSetupLbl` toggling mid-setup left stale drawings or none | Drawings created once on the tap event, toggle read only then | Drawing manager runs every bar: creates from frozen levels if enabled & missing (incl. after re-enabling), deletes if disabled, extends only while live | Code review | Tested only statically |
| R8 | Strategy could enter at CANDIDATE-equivalent instant and exits weren't bound to a setup | Entries on tap events; generic exit IDs | Entries only on ACTIVE activation (`evAct*`); entry/exit IDs embed setup ID; INVALIDATED/EXPIRED closes open position | Static scan of strategy block | Broker-emulator intrabar behaviour unverifiable here |
| R9 | Dashboard could mislead: regime during ATR warm-up, HTF ≤ chart TF, `/140` denominator | No missing-data surface | Diagnostics rows: Filters, Warnings (ATR/RVOL/VWAP missing, HTF≤TF), State·Reason, Confirmation notes | Code review | Early-bar indicators still warm up (inherent) |
| R10 | Same-bar SL+TP ambiguity undocumented in behaviour text (code did SL-first) | OHLC ordering unknowable | Kept SL-first (conservative), now stated in code, reason string and user guide | Doc | Fundamental OHLC limitation |
| R11 | FVG display/logic coupling | Fixed in v2 (A1) | Retained; inversion recolouring still respects visibility | — | — |
| R12 | PD map recreated every bar | Fixed in v2 (A3, islast) | Retained | — | — |

**Not defects / retained deliberately:** pivot lag (`sLen`/`iLen` — confirmation
delay, not repainting); HTF `[1]`+`lookahead_off`; original score factors (re-weighted,
configurable); all v1/v2 features remain present in V3.
