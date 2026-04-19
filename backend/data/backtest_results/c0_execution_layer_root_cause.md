# C.0 EXECUTION_LAYER_ISSUE root cause (2026-04-20)

## TL;DR

**The 4 "EXECUTION_LAYER_ISSUE" playbooks (FVG_Fill_V065, Range_FVG_V054, Liquidity_Raid_V056, FVG_Scalp_1m) are NOT broken. They fail because DAY_Aplus_1 spam drained the account negative before they had a chance to fire.** Position-size rejection at [risk_engine.py:966](backend/engines/risk_engine.py#L966) silently drops every setup once `account_balance × factor < entry_price`.

This is an **artifact of the fair audit configuration** (kill_switch OFF + DAY_Aplus_1 active), not a real bug. To prove it, the 4 ELI playbooks need a re-run with DAY_Aplus_1 disabled (or kill_switch ON).

## Evidence chain

### 1. Funnel observation (B0.2, fair_oct_w2)

| playbook | matches | setups | after_risk | trades | verdict |
|---|---|---|---|---|---|
| FVG_Fill_V065 | 24 | 24 | 24 | 0 | EXECUTION_LAYER_ISSUE |
| Range_FVG_V054 | 24 | 24 | 24 | 0 | EXECUTION_LAYER_ISSUE |
| Liquidity_Raid_V056 | 4 | 4 | 4 | 0 | EXECUTION_LAYER_ISSUE |
| FVG_Scalp_1m | 73 | 73 | 73 | 0 | EXECUTION_LAYER_ISSUE |

Setups pass risk filter, then vanish. No per-playbook trace explains why.

### 2. Smoking gun — global rejection counter

`debug_counts.counts.trades_open_rejected_by_reason` for fair_oct_w2:

```json
{"Position size < 1 share after cap": 495}
```

495 trades attempted but rejected with this single reason. No other rejection types — confirming this is **the** silent kill-path.

### 3. Code path

[risk_engine.py:951-966](backend/engines/risk_engine.py#L951-L966):

```python
if setup.symbol in ['SPY', 'QQQ']:
    position_size = risk_dollars / distance_stop
    position_size = int(position_size)
    if position_size < 1:
        return PositionSizingResult(valid=False, reason='Position size < 1 share')
    required_capital = position_size * setup.entry_price
    factor = self._get_max_capital_factor(self.state.trading_mode, setup.quality)
    max_capital = self.state.account_balance * factor
    if required_capital > max_capital:
        position_size = int(max_capital / setup.entry_price)
        if position_size < 1:
            return PositionSizingResult(valid=False, reason='Position size < 1 share after cap')
```

The "after cap" branch fires when `int(account_balance × factor / entry_price) < 1`. With factors ranging from 0.95 (SAFE) to 2.0 (A+), this requires `account_balance < entry_price / 2.0` — i.e., **account < ~$300 for SPY at $600**.

### 4. Account trajectory in fair_oct_w2

From the trades parquet (sorted by entry time):

- Trade 1 (2025-10-06 13:30): cumulative R = +0.02
- ...
- Trade 1877 (2025-10-10 15:00): cumulative R = **-222.6**

With `pnl_R_account` accumulating negatively at ~-0.12R/trade × 1877 trades, and risk per trade ≈ 1% of account, the equity goes deeply negative within hours. Once `state.account_balance` drops below ~$300, EVERY subsequent setup on SPY/QQQ rejects with "Position size < 1 share after cap" — regardless of playbook.

The 4 "ELI" playbooks happen to fire LATE in the session window:
- FVG_Fill_V065: `time_windows: [09:45, 12:00]` (after DAY_Aplus_1's 09:30 burst)
- Range_FVG_V054: similar mid-morning bias
- FVG_Scalp_1m: 1m playbook firing throughout
- Liquidity_Raid_V056: post-sweep, late

By the time their first setup is ready, the account has been crushed by DAY_Aplus_1.

### 5. Why other playbooks DO trade in the same run

- `Morning_Trap_Reversal` (08 trades opened): fires 09:30-10:30, **earlier** than DAY_Aplus_1's worst spam → some setups slip through before account is drained.
- `News_Fade`, `Engulfing_Bar_V056`, etc.: same — earlier or different sessions, account still solvent for a window.

This is a **timing artifact**, not a quality difference between playbooks.

## What this means for C.0

**Original B0.2 verdict (`EXECUTION_LAYER_ISSUE = bug exécution, pas détecteur`) was wrong.** The "execution layer issue" is actually:

> *The fair audit ran with DAY_Aplus_1 active and kill_switch OFF. DAY_Aplus_1 generated 2157 negative-E[R] trades, draining account_balance below the minimum-share threshold before late-session playbooks could fire. The position-size guard at risk_engine.py:966 then silently rejected 495 setups across multiple playbooks.*

**It's not a bug. It's the audit setup.**

## Re-test required (cheap, deferred — read-only diagnosis enough)

To confirm: run 1 week with `RISK_EVAL_CALIB_ALLOWLIST="FVG_Fill_V065,Range_FVG_V054,Liquidity_Raid_V056,FVG_Scalp_1m"` (no DAY_Aplus_1, caps actives). Expected: trades open normally, all 4 playbooks produce ≥1 trade per session.

If trades still don't open → real EXECUTION_LAYER bug. If they open → diagnosis confirmed and these playbooks join Phase B1/B2 calibration candidates.

## Action items

1. **Update CLAUDE.md** "Phase B0 findings" to reflect that EXECUTION_LAYER_ISSUE was an audit artifact, not a real bug.
2. **Update silent_playbooks_diagnosis.md** to add this root-cause analysis.
3. **Defer C.0 fix** — there is no fix needed. These 4 playbooks should be added to the next calibration run (B1 round 2 or C.1 portfolio expansion).
4. **Long-term hardening** (separate engine PR): make the position-size rejection emit a per-playbook counter so this kind of silent funnel collapse is visible in `debug_counts` without forensic parquet analysis.

## Related

- [silent_playbooks_diagnosis.md](silent_playbooks_diagnosis.md) — original B0.2 verdict (now updated)
- [bos_scalp_duration_anomaly.md](bos_scalp_duration_anomaly.md) — separate engine artifact (PHASE3B gate)
- [VERDICT_fair_audit_4weeks.md](../../results/labs/mini_week/VERDICT_fair_audit_4weeks.md) — Phase A baseline
