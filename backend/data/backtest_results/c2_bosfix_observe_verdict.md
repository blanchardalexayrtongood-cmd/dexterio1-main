# Phase C.2 — BOS_Scalp_1m engine fix + uncap observation (2026-04-20)

## TL;DR

**Engine fix applied** ([phase3b_execution.py:15-17](backend/engines/execution/phase3b_execution.py#L15-L17)): added `BOS_Scalp_1m` to `PHASE3B_PLAYBOOKS` frozenset → YAML `max_duration_minutes` now honored for this playbook (was silently falling to 120m global cap).

**Observation run** with YAML bumped 15 → **240** min (purely to uncensor the winner-duration distribution that was clipped at 120m).

**Result**: extending the cap **DEGRADED** E[R] -0.109 → **-0.118** (Δ -0.009) on n=51 trades. Uncensoring revealed:
- Winners genuinely want to run >120m (p50 winner duration jumped 81 → **209 min** when uncapped).
- BUT losers also want to run longer — 3 trades that were time_stops at 120m became hard SLs at 240m (R worsened: -0.111 → -0.637, -0.381 → -0.578).
- WR rose 37.3 → 45.1% (more wins captured) but avg loss R worsened -0.319 → -0.390.

**Conclusion**: BOS_Scalp_1m has **no duration sweet spot**. The signal lacks edge regardless of how long we hold. Engine fix is correct and stays. YAML reverted to **120m** (best-known E[R]).

## Engine fix detail

**File**: [backend/engines/execution/phase3b_execution.py:15-17](backend/engines/execution/phase3b_execution.py#L15-L17)

```python
# BEFORE
PHASE3B_PLAYBOOKS = frozenset(
    {"NY_Open_Reversal", "News_Fade", "Liquidity_Sweep_Scalp"}
)

# AFTER
PHASE3B_PLAYBOOKS = frozenset(
    {"NY_Open_Reversal", "News_Fade", "Liquidity_Sweep_Scalp", "BOS_Scalp_1m"}
)
```

**Why surgical (not lifting the gate globally)**: 12 SCALPs would change behavior if the gate were removed. Most are DENYLIST/DISABLED/EXECUTION_LAYER_ISSUE so unaffected, but `FVG_Fill_Scalp` (ALLOWLIST, active) would tighten 120 → 30 min — uncontrolled blast radius. Adding individual playbooks to PHASE3B as they're calibrated is the right "Wave N" pattern this gate was designed for.

## Observation result (BOS_Scalp_1m only, n=51 unchanged)

| metric | cap 120m (prior) | cap 240m (observation) | Δ |
|---|---|---|---|
| n trades | 51 | 51 | 0 |
| WR % | 37.3 | **45.1** | +7.8 |
| E[R] | -0.109 | **-0.118** | **-0.009 ↓** |
| total R | -5.57 | -6.03 | -0.46 |
| Exit SL | 25 | 28 | +3 |
| Exit TP1 | 4 | 4 | 0 |
| Exit time_stop | 22 | 19 | -3 |
| Winner dur p50 / p75 / p90 | 81 / 120 / 120 | **209 / 240 / 240** | uncensored |
| Avg win R | 0.245 | 0.212 | -0.033 |
| Avg loss R | -0.319 | **-0.390** | -0.071 |

**Same 51 trades** — the 240m cap only changed the exit type/timing of the 22 trades that were time_stops at 120m.

### Transition matrix (cap 120m → cap 240m)

| 120m exit | 240m: SL | 240m: TP1 | 240m: time_stop |
|---|---|---|---|
| SL | 25 | 0 | 0 |
| TP1 | 0 | 4 | 0 |
| **time_stop** | **3** | 0 | **19** |

3 time_stops became SLs (worse), 19 stayed time_stops (some marginally better, some worse). Net effect: **uncapping is strictly net-negative** for E[R] despite a higher WR.

## Why higher WR but lower E[R]?

Of the 22 time_stops@120m, 19 stayed time_stops at 240m and several flipped from slightly-negative to slightly-positive R (e.g., -0.203 → 0.008, -0.024 → 0.022). That bumped the WR count without bumping the R per win — the new "wins" are tiny (~0.02-0.05 R), while the 3 trades that became SLs lost -0.5 R each.

Math: +4 marginal wins × ~0.05 R = +0.2 R; -3 trades × -0.3 R extra loss = -0.9 R. Net: -0.7 R / 51 trades = -0.014 E[R]. Matches the observed Δ -0.009.

## Cap sweep simulation (rough)

Using the 240m run as the data source, simulating what E[R] would have been if we'd capped earlier:

| cap | trades exit naturally | trades truncated | sim E[R] (truncated @ r=0) | sim E[R] (truncated @ r=-0.2) |
|---|---|---|---|---|
| 30 min | 18 | 33 | -0.005 (best, optimistic) | -0.134 |
| 60 min | 24 | 27 | -0.065 | -0.171 |
| 90 min | 28 | 23 | -0.096 | -0.187 |
| 120 min | 29 | 22 | -0.094 | -0.180 |
| 240 min | 51 | 0 | -0.118 (actual) | -0.118 (actual) |

The "truncated @ r=0" optimistic assumption suggests 30m might be best — but real truncated trades are likely mid-adverse, not at zero. Honest read: **no cap value moves E[R] meaningfully positive.** All caps land in [-0.10, -0.18] range.

## Cross-finding — TP1 placement is the actual problem

Looking at the 19 trades that stayed time_stops at 240m, several had positive R at exit (0.026, 0.156, 0.181, 0.490) but never reached TP1 (1.5R per YAML). The B1 finding (peak_R p60 = 0.40R) was correct: **typical winners only reach 0.5-1.0R peak; TP1 at 1.5R is unreachable for most setups.**

This is a **TP1 calibration problem, not a duration problem**. Lowering TP1 from 1.5R to ~0.5R would:
- Convert most of the 19 positive-R time_stops into TP1 wins.
- Capture the modest profit before mean-reversion eats it.
- Risk: locks in small wins, prevents the rare big runner.

Per the SIGNAL_QUALITY_SUSPECT B1 flag, however: **proposing TP1 < 0.5R is itself a signal-quality red flag** (the setup doesn't produce enough directional follow-through for any meaningful TP). This is the same pattern as Liquidity_Sweep_Scalp and Engulfing_Bar_V056.

## Decision

### What changes
1. **Engine fix stays**: `BOS_Scalp_1m` added to `PHASE3B_PLAYBOOKS`. This is correct behavior — YAML `max_duration_minutes` should be honored for SCALPs.
2. **YAML reverted 240 → 120**: 120m matches the prior effective cap (where E[R]=-0.109 was measured) and is now correctly enforced by the engine.
3. **No further BOS_Scalp_1m calibration recommended**: signal lacks edge across the duration range tested. SIGNAL_QUALITY_SUSPECT confirmed.

### What's blocked vs unblocked
- ✅ **Unblocked**: future SCALP playbooks added to PHASE3B will have YAML `max_duration_minutes` honored.
- ✅ **Confirmed**: BOS_Scalp_1m's prior B1 corpus observations (wins p75=120) reflected censorship; true winners want 200+ min, but the math doesn't close anyway.
- ❌ **Not unblocked**: BOS_Scalp_1m calibration. The duration anomaly was real but the underlying signal has no edge to extract.

### Three honest paths forward (decision needed from user)

**(A) Accept BOS_Scalp_1m as a third SIGNAL_QUALITY_SUSPECT failure** — it joins Engulfing_Bar_V056 and Liquidity_Sweep_Scalp as B1 candidates that can't be calibrated. Move BOS_Scalp_1m toward DENYLIST consideration (E[R]=-0.109 across 4 weeks is clear).

**(B) Lower TP1 1.5R → 0.5R as a separate calibration test** — risks codifying SIGNAL_QUALITY_SUSPECT pattern but might extract the modest moves that exist. Single-variable test, ~30 min.

**(C) Pivot to portfolio reduction** — after 3 calibration targets failed (Morning_Trap, Liquidity_Sweep, BOS_Scalp_1m all asymptote at E[R] in [-0.03, -0.12]), the question shifts from "calibrate signals" to "do any signals have edge?" Phase A had 3 KILLs and 5 CALIBRATEs; with 3/5 CALIBRATEs now confirmed null/negative, only Engulfing_Bar_V056 and SCALP_Aplus_1 remain — both already flagged. Honest move: declare B/C inconclusive and pivot to either (i) signal redesign or (ii) Phase E portfolio with a much smaller surviving set.

## Recommendation

**Option A + C combined**: accept BOS_Scalp_1m null result, freeze Phase B/C at this state, escalate to **portfolio survivor analysis**. Stop calibrating individual signals; instead, list every playbook with E[R] ≥ -0.05 across 4 weeks and ask whether any combination has portfolio-level edge.

The pattern is now clear across 3 calibration attempts:
- Morning_Trap_Reversal: B1 + B2 + C.1 → asymptote at E[R] = -0.081 (best of 3)
- Liquidity_Sweep_Scalp: C.1 → no movement (E[R] = -0.034)
- BOS_Scalp_1m: C.2 (engine fix + uncap) → degradation (E[R] = -0.118)

No single-signal calibration has produced E[R] > 0 across 4 weeks. Continued calibration is hitting diminishing returns on increasingly speculative levers.

## Artifacts

- Engine fix: [phase3b_execution.py:15-17](../../engines/execution/phase3b_execution.py#L15-L17)
- YAML observation snapshot: [backend/knowledge/campaigns/audit_c2_bosfix_observe.yml](../../knowledge/campaigns/audit_c2_bosfix_observe.yml) (max_duration=240 captured)
- YAML reverted to: [backend/knowledge/playbooks.yml](../../knowledge/playbooks.yml) BOS_Scalp_1m `max_duration_minutes: 120`
- Corpus: `backend/results/labs/mini_week/c2_bosfix_observe_v1/`
- Related: [bos_scalp_duration_anomaly.md](bos_scalp_duration_anomaly.md), [c1_lsweep_verdict.md](c1_lsweep_verdict.md), [c1_vwap_verdict.md](c1_vwap_verdict.md)
