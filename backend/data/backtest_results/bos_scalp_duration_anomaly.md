# BOS_Scalp_1m duration anomaly — root cause (2026-04-19)

## TL;DR

**YAML `max_duration_minutes` is honored only for 3 Phase3B playbooks. All other SCALPs fall to a global 120-min cap.** BOS_Scalp_1m YAML says 15m but the engine enforces 120m.

## Evidence (from calib_corpus_v1)

- `BOS_Scalp_1m`: YAML `max_duration_minutes: 15` but observed wins duration p70/p75 = **120.0 / 120.0** min.
- `Liquidity_Sweep_Scalp`: YAML `max_duration_minutes: 30` and observed wins p75 = **30.0** min. ✓ respected.
- `Engulfing_Bar_V056`: YAML `max_duration_minutes: 120` coincides with the global fallback → no observable discrepancy.
- `Morning_Trap_Reversal`: category DAYTRADE, not subject to the SCALP time_stop path — 0% time_stop in corpus.

## Root cause in code

**[paper_trading.py:103-112](backend/engines/execution/paper_trading.py#L103-L112)** only assigns `max_hold_minutes` to the trade if the playbook is in `PHASE3B_PLAYBOOKS`:

```python
if is_phase3b_playbook(playbook_name):
    if setup.trade_type == "DAILY":
        ...
    elif setup.trade_type == "SCALP":
        if pb_def is not None and getattr(pb_def, "max_duration_minutes", None) is not None:
            max_hold_minutes = float(pb_def.max_duration_minutes)
```

**[phase3b_execution.py:15-17](backend/engines/execution/phase3b_execution.py#L15-L17)** — the gate:

```python
PHASE3B_PLAYBOOKS = frozenset(
    {"NY_Open_Reversal", "News_Fade", "Liquidity_Sweep_Scalp"}
)
```

**[paper_trading.py:378-401](backend/engines/execution/paper_trading.py#L378-L401)** — the fallback in the update loop:

```python
max_scalp_minutes = (
    float(trade.max_hold_minutes)
    if trade.max_hold_minutes is not None
    else float(getattr(self.risk_engine, '_max_scalp_minutes', 120.0))
)
```

For BOS_Scalp_1m: not Phase3B → `trade.max_hold_minutes = None` → falls to `risk_engine._max_scalp_minutes` (default **120.0**). The YAML value of 15 is never consulted.

This is **not a bug** in the strict sense — the comment at [paper_trading.py:378](backend/engines/execution/paper_trading.py#L378) says "Phase 3B: max_hold_minutes playbook, sinon cap global legacy". The design gates YAML enforcement behind Phase3B membership. But it means:

1. Any new SCALP playbook setting `max_duration_minutes` in YAML will be silently ignored.
2. B1 calibration's `max_duration_minutes: 15→120` proposal for BOS_Scalp_1m is a no-op — the engine already runs at 120m globally.
3. The B1 corpus observations for BOS_Scalp_1m reflect 120m reality, not the 15m the YAML advertises.

## Impact on Phase B

### B1 calibration validity

The BOS_Scalp_1m proposed YAML patch (`max_duration_minutes: 120`) is **vacuous** — it matches the hidden default. Applying it changes nothing at runtime. The corpus observations (wins duration p75=120) reflect the 120m cap, not a true winner-duration distribution at the YAML-declared 15m.

### What this means for B2

- **B2 Morning_Trap run is unaffected**: Morning_Trap is DAYTRADE (no SCALP time_stop logic path).
- **Any future BOS_Scalp_1m calibration requires fixing the gate first**. Otherwise we'd calibrate at 120m while deploying at 15m (or vice versa).

## Proposed fix (deferred, not applied)

Remove the `is_phase3b_playbook()` guard for `max_hold_minutes` — honor YAML for all SCALPs:

```python
# paper_trading.py, around line 103
if setup.trade_type == "SCALP":
    if pb_def is not None and getattr(pb_def, "max_duration_minutes", None) is not None:
        try:
            max_hold_minutes = float(pb_def.max_duration_minutes)
        except Exception:
            max_hold_minutes = None

# Phase3B-specific DAILY behavior stays gated behind is_phase3b_playbook
if is_phase3b_playbook(playbook_name):
    if setup.trade_type == "DAILY":
        if pb_def is not None and should_attach_session_window_end(playbook_name, setup.trade_type):
            session_window_end_utc = compute_session_window_end_utc(pb_def, now_ts)
```

Regression risk: any SCALP YAML with an incorrectly tight `max_duration_minutes` (e.g., copy-paste errors) would start enforcing it. Mitigation: before applying the fix, grep current YAMLs for SCALP + small max_duration and validate the values are intentional.

## Conclusion

- BOS_Scalp_1m can **not** be calibrated until this gate is fixed. Calibration under the current gate would produce YAML values that do not transfer.
- Phase B2 continues with Morning_Trap_Reversal only as planned.
- This fix belongs in a follow-up patch alongside Phase C (signal-quality filters) or as a standalone engine-correctness PR.
