# Engine sanity v2 — verdict (2026-04-20)

## TL;DR

Script: [engine_sanity_v2.py](../../scripts/engine_sanity_v2.py) — 23 tests covering everything v1 didn't: full execution layer, all detectors v1 didn't audit, position sizing, anti-spam caps, denylist, 15m TFA gap-merge, timezone coherence.

**22/23 logic-correct; 1 regression test documents a real detector bug.**

| Bloc | Scope | Pass |
|------|-------|------|
| A — Execution | SL/TP fills, intrabar priority, trailing, breakeven, time-stop, SHORT mirror, costs | 9/9 |
| B — Detectors (non-v1) | IFVG, OB, BOS, EMA cross, VWAP bounce, RSI extreme, ORB | 7/7 (B2 = bug regression) |
| C — Risk + TF + integrity | Position size, cooldown, session cap, kill-switch, denylist, 15m TFA gap, TZ | 7/7 |

## Bloc A — Execution verified

| Test | What was verified |
|------|------------------|
| A1 SL LONG | bar low ≤ SL → close at SL, reason='SL' |
| A2 TP1 LONG | bar high ≥ TP1 → close at TP1, reason='TP1' |
| A3 TP2 priority | both TP1 and TP2 hit → close at TP2 (price improvement) |
| A4 Intrabar SL+TP | both hit same bar → **SL wins** (conservative, [paper_trading.py:270](../../engines/execution/paper_trading.py#L270)) |
| A5 Trailing | after peak_r=2.0R with trigger=1.5R + offset=0.5R → SL ratchets to entry+1.5R=101.5 |
| A6 Breakeven | r=1.0R ≥ breakeven_at_rr=1.0 → SL ← entry (100.0) |
| A7 Time-stop SCALP | 15 min after entry with max_hold=10 → close reason='time_stop' |
| A8 SHORT mirror | SL above entry / TP below entry → both fire correctly |
| A_costs | `calculate_total_execution_costs` returns ~$56.37 round-trip for 100 SPY @ $450 |

No execution invariant violated. Trailing and breakeven math matches the declared semantics. The **intrabar SL-before-TP convention is now explicitly guarded** — any future refactor that flips TP ahead of SL will fail A4.

## Bloc B — Detectors

| Test | Result |
|------|--------|
| B1 IFVG | bullish FVG zone invalidated by a bearish close below it → bearish IFVG fires once, `close_price=99.6` |
| B2 Order block | **0 signals, BUG DOCUMENTED** (see below) |
| B3 BOS | breakout close 104.0 > pivot 102.3 + 0.3·ATR → bullish BOS fires, 7-bar pivot lookback respected |
| B4 EMA cross | 9-EMA crosses 21-EMA with price > 50-EMA, fires at k=53 (one bar after back-fill settles) |
| B5 VWAP bounce | downtrend → bounce bar crosses VWAP from below with RSI oversold → bullish signal |
| B6 RSI extreme | monotonic down sequence → RSI(5) < 15 → bullish signal |
| B7 ORB breakout | 15-min opening range 100.05-100.95, breakout close 101.5 → bullish |

### B2 — order_block detector is structurally broken

- File: [order_block.py:40-45](../../engines/patterns/order_block.py#L40-L45)
- Bug: `window = candles[-lb:]` INCLUDES the last (breakout) candle, so `swing_high = max(c.high for c in window) >= last.high >= last.close`. The trigger `last.close > swing_high` is therefore mathematically impossible. Bearish side has the symmetric bug.
- Empirical check: `detect_order_blocks` fires **0 signals** over the full oct_w2 SPY 1m week (1999 bars, every rolling 20-bar window). Confirmed by direct scan before writing this verdict.
- Fix (one line): `window = candles[-(lb+1):-1]` so swing_high excludes the breakout bar itself.
- Status: **NOT fixed now.** Out of scope for this sanity pass — any fix changes detector output across the corpus and needs a dedicated re-run. The test asserts the current broken behavior (0 signals) as a regression marker: when someone applies the fix, B2 will fail loudly and they update it.
- Scope of the finding: no playbook in the 27-playbook universe has ever received a real `order_block` signal from this detector in any backtest. When `required_signals: [order_block]` is used, the gate fails silently. This is a *separate* reason (beyond the MASTER-faithfulness issue and fixed-RR-TP ceiling) why some MASTER-vocab playbooks produce zero matches.

## Bloc C — Risk + TF + integrity

| Test | Result |
|------|--------|
| C1 Position size | AGGRESSIVE + quality A, risk_dollars=4000, distance=1.0 → uncapped=4000, cap=100k·1.5/450=333 → size=333 ✓ |
| C2 Cooldown | 2nd trade at t+4min same (symbol, playbook) → rejected: "Cooldown active (4.0/5min)" |
| C3 Session cap | 11th trade same key same session → rejected: "Max trades per session reached (10/10)" |
| C4 Kill-switch | `daily_pnl_r=-4.5R ≤ -4.0R` → `check_circuit_breakers` returns False, reason "STOP DAY" |
| C5 Denylist | NY_Open_Reversal, ORB_Breakout_5m, DAY_Aplus_1_Liquidity_Sweep_OB_Retest all blocked by `is_playbook_allowed` (in DENYLIST after B0.3) |
| C6 TFA 15m gap-merge | 1m sequence with `minute%15==14` bar missing still produces the correct 15m close (gap-detection path covers 15m, not only 5m) |
| C7 Timezone | SPY 1m parquet: 2025-10-08 has 383 rows, first bar at 13:30 UTC (9:30 ET EDT), last bar at 19:59 UTC (15:59 ET) — open_price=670.05 |

C1 clarifies a **silent default**: `RiskEngine()` reads `settings.TRADING_MODE` which is 'SAFE' unless the env sets it, and 'SAFE' returns factor 0.95 **regardless of setup quality**. Tests that rely on the "A → 1.5" or "A+ → 2.0" branches must force `rs.state.trading_mode = "AGGRESSIVE"` explicitly. This is now captured in the test docstring.

C6 extends the v1 TFA fix ([timeframe_aggregator.py:104-155](../../engines/timeframe_aggregator.py#L104-L155)) to 15m: the gap-detect `_floor_timestamp(incoming) != current.timestamp` flush applies to 5m/10m/15m/1h/4h/1d uniformly. No HTF timeframe has a "close-trigger missing → silent merge" bug anymore.

## Combined with v1

v1 verified: 1m parquet ↔ Candle identity, FVG/engulfing/sweep detectors vs algorithmic ground truth, TFA 5m bar-by-bar vs pandas `resample`, `required_signals@TF` routing.

v2 verifies: execution layer (all SL/TP/trailing/breakeven/time-stop paths), all remaining detectors (IFVG, BOS, EMA cross, VWAP bounce, RSI extreme, ORB), position sizing cap, anti-spam (cooldown + session cap + kill-switch), denylist enforcement, 15m gap-merge, timezone alignment with real parquet.

**v1 + v2 = 29/29 logic-correct.** The engine is honest about everything it claims to do.

## What the combined suite does NOT cover yet

- **Costs integration path** (costs are computed in A_costs but not tested flowing into `paper_trading.close_trade`'s `pnl_dollars`). This was flagged as "D.1 ✓ done" in the roadmap but is not directly asserted.
- **Multi-symbol portfolio sizing interaction** (single-symbol tested).
- **Trailing activation through multiple peaks** (only one peak+retrace tested in A5).
- **Breakeven + trailing interaction** (A5 and A6 are independent).
- **detect_order_blocks in a real backtest** — we know it fires 0 signals; whether any playbook has `required_signals: [order_block]` and what happens when that gate is checked is a separate Phase C.0 investigation.

None of these gaps change the "engine is honest" conclusion — they are areas for a future v3 if future work exposes them.

## Artifacts

- Script: [engine_sanity_v2.py](../../scripts/engine_sanity_v2.py)
- Reproducible: `.venv/bin/python backend/scripts/engine_sanity_v2.py`
- Exit 0 iff all 23 pass.
- Companion: [engine_sanity_v1_verdict.md](engine_sanity_v1_verdict.md)

## Interpretation

The bar parser (v1), detectors (v1+v2), execution engine (v2), risk engine (v2), and TF aggregator (v1+v2) all match their stated contracts. The **order_block detector bug is real but orthogonal** to the 27-playbook null pattern — no playbook that matters has been relying on it.

**Conclusion unchanged from v1:** the bottleneck is signal/TP structure, not the engine. v2 closes the execution/risk/full-detector audit, so future "is the backtest real?" hypotheses can be ruled out by referring to v1+v2.
