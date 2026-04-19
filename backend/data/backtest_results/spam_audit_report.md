# Phase B0.1a — Spam Audit (read-only)

**Params:** cooldown=5min, session_cap=10/playbook/session.
**Source:** fair_* parquets (RISK_EVAL_RELAX_CAPS=true).

## Verdict per playbook

| playbook | trades | cooldown_blocked | session_cap_blocked | combined_blocked % | re_entry_after_sl % | p50_gap_sec | p95_gap_sec | pct_under_5min | dedup | verdict |
|---|---|---|---|---|---|---|---|---|---|---|
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 2868 | 2783 (97.0%) | 2612 (91.1%) | 97.0% | 66.2% | 60 | 540 | 90.4% | 0 | **SPAM** |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 2157 | 2142 (99.3%) | 1926 (89.3%) | 99.3% | 88.2% | 60 | 1080 | 85.6% | 0 | **SPAM** |

## Classification
- `SPAM` : >50% trades blocked by prod caps → calibration on RELAX_CAPS data invalid
- `BORDERLINE` : 20-50% blocked → flag, re-run recommended with caps active
- `LEGITIMATE_VOLUME` : <=20% blocked → volume is real, calibration safe
