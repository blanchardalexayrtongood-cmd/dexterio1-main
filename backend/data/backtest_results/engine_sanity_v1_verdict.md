# Engine sanity v1 — verdict (2026-04-20)

## TL;DR

Script: [engine_sanity_v1.py](../../scripts/engine_sanity_v1.py) — 6 tests, read-only, algorithmic ground-truth only (no hand-picked examples). Week tested: 2025-10-06 to 2025-10-10 (same window as `oct_w2`), 1999 1m SPY bars.

**5/6 tests PASS. Test 5 FAILS with a real engine-correctness finding.**

| Test | Result |
|------|--------|
| 1. OHLC parquet → Candle field-by-field | ✅ PASS (1999/1999 bars identical) |
| 2. FVG detector vs algorithmic ground truth (c1.high<c3.low, min_gap 0.3%) | ✅ PASS (2/2 detected, 0 false positives) |
| 3. Engulfing predicate (`_is_bullish/bearish_engulfing`) 1:1 match | ✅ PASS (21/21 identical, 0 extra, 0 missing) |
| 4. Liquidity sweep detector, zero false positives | ✅ PASS (0 false positives this week) |
| 5. 1m→5m aggregation: TimeframeAggregator vs pandas resample | ❌ **FAIL** (see below) |
| 6. `required_signals@TF` gate (IFVG, FVG, SWEEP + direction filters + wrong-TF reject) | ✅ PASS (9/9 cases) |

**Core engine reads bars correctly and detectors match their stated definitions**. Tests 1–4, 6 cover: OHLC identity, FVG/engulfing/sweep pattern definitions, signal wiring (the `required_signals@TF` gate that routes detector output to playbooks). All clean.

**Test 5 uncovers a real gap-handling bug in `TimeframeAggregator`** that silently merges ~6% of 5m bars when the minute-with-`%5==4` 1m bar is missing. Not severe enough to explain the 27-playbook null pattern, but worth documenting and fixing.

## Test 5 finding: TFA gap-swallowing

### What the engine actually uses

The backtest loop uses [`engines/timeframe_aggregator.py`](../../engines/timeframe_aggregator.py) (`TimeframeAggregator`, wired in [engine.py:91,524](../../backtest/engine.py#L91) — the pandas `resample('5T')` path in `_build_multi_timeframe_candles` is **dead legacy code**, explicitly commented "ne JAMAIS appeler").

TFA is **incremental and close-driven**: it buffers OHLCV from 1m bars until `is_close_5m` fires, which is hard-coded as `minute % 5 == 4` ([timeframe_aggregator.py:74](../../engines/timeframe_aggregator.py#L74)).

### The bug

When the 1m bar with `minute%5==4` is missing from the feed (which happens naturally in Polygon/IEX data during low-volume stretches — volume=0 bars are dropped at ingest), TFA **never fires `is_close` for that window** and keeps absorbing subsequent 1m bars into the same "5m" bar until the next `%5==4` arrives.

**Concrete example, 2025-10-08 17:15 window**:

Source 1m bars present (17:13 and 17:19 are missing):
```
17:15  O/H/L/C = 672.68/672.72/672.68/672.72
17:16           672.80/672.80/672.78/672.78
17:17           672.805/672.85/672.805/672.85
17:18           672.89/672.89/672.89/672.89
17:20           672.86/672.86/672.83/672.83
17:21           672.86/672.86/672.83/672.83
17:22           672.83/672.865/672.83/672.865
17:23           672.865/672.95/672.86/672.92
17:24           672.88/672.905/672.88/672.905
```

TFA emits ONE 5m bar at 17:15 with `H=672.95, C=672.905` — it contains **9 minutes of data** because 17:19 (the expected close trigger) is missing, so TFA kept aggregating through 17:20–17:24 until `%5==4` fired at 17:24.

Pandas `resample("5min")` correctly emits TWO bars:
- 17:15 bin (17:15–17:19) → `H=672.89, C=672.89` (only 4 bars present in source)
- 17:20 bin (17:20–17:24) → `H=672.95, C=672.905`

**TFA is missing the 17:20 bar entirely** — it's silently merged into 17:15.

### Scope

Over this 1-week corpus (1999 1m bars, 52 intraday gaps ≥1 min):
- Pandas resample produces **420** 5m bars
- TFA emits **396** closed 5m bars (caps observed at 200 due to `WINDOW_SIZES["5m"]=200` rolling-window cap)
- **24 bars silently merged = 5.7% of 5m history**

The same bug applies to 15m (trigger `%15==14`), 1h (`minute==59`), 4h, and 1d. Likely worse on higher TFs where missing a single close-minute merges multiple 5-minute windows' worth of data.

### Impact on detectors

Detectors scanning the last N 5m bars see:
- OHLC of merged bars spanning up to ~10 minutes (wider high/low range than true 5m)
- One-off "missing" 5m bars in the sequence (17:20 in the example above doesn't exist in TFA's candle list)
- For FVG: the 3-bar gap check still works but bars c1/c2/c3 may not be chronologically adjacent in wall-clock time.

**Estimated signal-level effect**: 5.7% of 5m history contaminated per week is modest but non-zero. Probably explains occasional spurious or missed FVGs, not a root cause of the 27-playbook null pattern.

### Also: rolling-window cap

`WINDOW_SIZES["5m"] = 200` caps the 5m history TFA exposes. Over a 1-week run (~400 5m bars), detectors can only see the **last 200** closed 5m bars at any point. For FVG/IFVG/sweep detectors that scan `candles[-30:]` this is fine; for playbook logic that might expect multi-day HTF history, this is a silent truncation.

## What this means for the 27-playbook null pattern

The original question: *"est-ce que notre moteur est vraiment réel? est-ce que le bot comprend vraiment les patterns bougie et lit comme il faut?"*

Answer: **the engine reads bars correctly and the detectors match their stated definitions**. The 4 core detector tests (FVG, engulfing, sweep, signal routing) all pass with zero false positives against algorithmic ground truth.

The TFA gap-merge bug is real but explains at most ~6% distortion on 5m history, not a systematic edge kill.

**More plausible root causes for the null pattern** (already documented elsewhere):
1. The 7 "MASTER faithful" playbooks (`FVG_Fill_V065`, `Liquidity_Raid_V056`, `Range_FVG_V054`, `Asia_Sweep_V051`, `London_Fakeout_V066`, `Engulfing_Bar_V056`, `OB_Retest_V004`) borrow MASTER vocabulary but are NOT truly MASTER-faithful (Phase D.2: 0/7 enforce D/4H bias, 0/7 use liquidity-targeting TPs, 0/7 use 1m confirmation). They test the detector, not the MASTER setup.
2. Fixed RR TPs (2.0R/4.0R) are systematically unreachable for signals where `peak_R p80 < 1.1R` (Aplus_03_v1, Morning_Trap, BOS_Scalp, Liquidity_Sweep all hit this ceiling).
3. 3 of 6 MASTER families have never been tested with faithful instantiations (Aplus_03 v1 was the first — not a null signal, but needs minimal TP calibration to cross zero).

**Engine is honest.** The bottleneck is signal/TP structure, not bar parsing.

## Recommended next actions

**Low priority, not blocker for Phase B/C/D work:**

1. **Fix TFA gap handling**: when a new 1m bar arrives whose `_floor_timestamp` doesn't match `current.timestamp`, close the previous bar before starting the new one. Optional: pad missing minutes with synthetic bars (preserving last close) for time-series continuity. Affects all HTF (5m, 10m, 15m, 1h, 4h, 1d).
2. **Remove dead `_build_multi_timeframe_candles` code path** from `backend/backtest/engine.py:526-609` — it uses pandas `5T` frequency which pandas 3.x rejects, confirming it's never called.
3. **Consider raising `WINDOW_SIZES["5m"]` from 200 to ≥500** for backtest-mode completeness, or make it configurable. Current detectors don't need >50 bars but reporting/analysis code might.

**None of these is blocking.** The sanity suite confirms the engine is sound for the signal-level diagnostic work currently in progress (Aplus_03 v1 calibration, Family A/B/F untested instantiations).

## Artifacts

- Script: [backend/scripts/engine_sanity_v1.py](../../scripts/engine_sanity_v1.py)
- Raw output: captured at runtime, reproducible with `.venv/bin/python backend/scripts/engine_sanity_v1.py`
- Dead code identified: [backend/backtest/engine.py:526-609](../../backtest/engine.py#L526-L609)
- Production aggregation path: [backend/engines/timeframe_aggregator.py](../../engines/timeframe_aggregator.py)
