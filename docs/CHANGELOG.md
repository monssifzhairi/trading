# Changelog

## V3 (2026-10-23) — files: `MONSSIF_QUANT_EDGE_v3.pine`, `MONSSIF_QUANT_EDGE_v3_Strategy.pine`
- Explicit 7-state machine: IDLE → CANDIDATE → ACTIVE → TP1_REACHED → (INVALIDATED | COMPLETED | EXPIRED); terminal states held for display; monotonic setup IDs; one live setup max.
- Candidate confirmation: later confirmed close beyond the zone within `candMaxBars` (default 5); zone prices copied to state at candidate time so OB-box deletion can't corrupt a live candidate.
- Level engine: entry = activation-bar close; SL = frozen zone edge ∓ `atrBuffer`×ATR; TP1/TP2 = R multiples; all frozen levels rounded to `syminfo.mintick`; activation validates ATR>0, risk>0, ordering, `maxRiskAtr` cap; rejections recorded with reason.
- Gap rules: open through SL → INVALIDATED(gap); open beyond TP2 → COMPLETED(gap); same-bar SL+TP → SL assumed first (documented).
- Decoupled toggles: `showOB`, `showLiq`, `showSigs`, `showRisk`, `showSetupLbl`, `showFvg` affect visibility only (zones/lines tracked transparently when hidden). Strategy trades unaffected by any visual setting.
- Score: single bucket set, default max 100; EMA+HTF share one capped confirmation bucket; every weight is an input; dashboard shows per-factor `points/max`; hard filters listed separately and never award points.
- Dashboard: 24 → 28 rows (+ Filters, Warnings, State·Reason, Confirmation); section toggles for Setup/Risk and Diagnostics rows; score denominator is the configured max.
- Alerts: Candidate vs Confirmed are separate conditions; dynamic `alert()` for candidate (zone + window) and activation (all frozen levels + ID); invalidation/completion include ID + reason context via dashboard.
- Strategy: entries only at ACTIVE; exit IDs embed setup ID; INVALIDATED/EXPIRED `close_all`; defaults 0.05% commission / 2 ticks slippage (overridable in Strategy Tester Properties).
- Preserved: v2 engine (structure, OB, FVG, liquidity, PD map), colors/identity, all prior inputs (new ones appended), `strategy.losstrades` spelling, CW10002/CW10003 fixes, shorttitles ≤10 (`MQE V3` / `MQEV3 Sim`).

## v2.0.1 (041f8d7) — `strategy.losstrades` typo, `MQE Strat` shorttitle, hoisted `f_makeOB`/VWAP crosses.
## v2.0 (40b5eec) — state machine v1, frozen risk levels, 24-row dashboard, strategy twin.
## v1 (2fd453c) — integration of `smart-money-concepts.pine` engine + VWAP/EMA/ADX/RSI/RVOL/HTF modules.
