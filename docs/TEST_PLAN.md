# Test Plan & Real Results

## A. Static validation — EXECUTED 2026-10-23 (this repo)
Script: ad-hoc Python scan (balanced delimiters, single declaration per file,
no `strategy.*` in indicator, `strategy.losstrades` spelling, shorttitle ≤10,
no duplicate top-level names, `lookahead_off` only, ≥15 alertconditions,
state-machine tokens present, `showOB/showSigs` absent from logic conditions,
table 2×28). **Result: ALL CHECKS PASS on both V3 files (23/23 per file).**
This proves text-level consistency only — **it is not compilation.**

## B. TradingView validation — PENDING (not executed; no TV access here)
Do not mark any item passed until run in TradingView:
1. ☐ Compile `MONSSIF_QUANT_EDGE_v3.pine` — record every error/warning verbatim.
2. ☐ Compile `MONSSIF_QUANT_EDGE_v3_Strategy.pine` independently.
3. ☐ Symbols/TFs: an index, FX pair (no-volume check), crypto; 5m/1H/4H; HTF both above and ≤ chart TF (warning row must appear).
4. ☐ Bar Replay: candidate appears on tap close only; activation only on a later confirming close; levels never move after activation; terminal states fire once.
5. ☐ Toggle every visual switch mid-setup: logic/alerts unchanged; risk drawings appear/disappear without duplicates.
6. ☐ Alerts: create one `alertcondition` alert and one "alert() function calls only" alert; trigger in Replay; confirm no duplicates per event and candidate ≠ confirmed wording.
7. ☐ Strategy Tester: trades only from ACTIVE setups; TP1 partial % + TP2/SL remainder; invalidation closes position; record trades/win-rate/net/PF/maxDD from the report (do not copy Data-Window plots as results).
8. ☐ Regression vs `archive/` v2 on the same chart: structure labels, OB/FVG, sweeps, PD map unchanged except documented state-machine behaviour.

## C. Acceptance status
Criteria 2,3,5 (risk validation, transitions, toggle decoupling): implemented + statically checked, **runtime unverified**.
Criteria 1,4,7 (compile, no duplicate alerts, Strategy Tester): **PENDING TradingView**.
No backtest metrics exist yet; none are claimed.
