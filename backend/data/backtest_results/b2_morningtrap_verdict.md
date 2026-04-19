# Phase B2 — Morning_Trap_Reversal calibration verdict (2026-04-20)

## TL;DR

**Patch applied**: `breakeven_at_rr: 1.0 → 2.15`, `max_duration_minutes: — → 155` ([calibration_patch_v1.yml](calibration_patch_v1.yml), Morning_Trap_Reversal entry only).

**Result on 4-week corpus (jun_w3 + aug_w3 + oct_w2 + nov_w4, caps actives, 28 playbooks loaded, allowlist 4 calibration targets)**:

- Morning_Trap_Reversal: E[R] **-0.147 → -0.123** (Δ +0.024), WR 20.6% → 25.0%, n 34 → 32. **Improvement directionally consistent (BE relaxation prevents premature stop-out, longer duration captures more peak), but does NOT cross zero.**
- Other 3 targets unchanged or near-noise drift (no patch applied to them).

**B2 gate verdict**: ❌ **FAIL** — plan required ≥3 playbooks crossing E[R]<0 → E[R]>0; got 0/3.

## Per-playbook delta

| playbook | n (B→A) | WR% (B→A) | E[R] (B→A, Δ) | time_stop% (B→A) | avg peak_R (B→A) | avg \|mae_r\| (B→A) | total_R (B→A) |

## Per-playbook delta

| playbook | n (B→A) | WR% (B→A) | E[R] (B→A, Δ) | time_stop% (B→A) | avg peak_R (B→A) | avg |mae_r| (B→A) | total_R (B→A) |
|---|---|---|---|---|---|---|---|
| BOS_Scalp_1m | 51 → 51 | 37.3 → 37.3 | -0.11 → -0.11 (+0.000 =) | 43.1 → 43.1 | 0.45 → 0.45 | 0.56 → 0.56 | -5.59 → -5.59 |
| Engulfing_Bar_V056 | 34 → 32 | 35.3 → 37.5 | -0.101 → -0.089 (+0.012 ↑) | 52.9 → 56.2 | 0.62 → 0.63 | 0.5 → 0.46 | -3.43 → -2.84 |
| Liquidity_Sweep_Scalp | 51 → 51 | 41.2 → 41.2 | -0.029 → -0.029 (+0.000 =) | 54.9 → 54.9 | 0.56 → 0.56 | 0.41 → 0.41 | -1.49 → -1.48 |
| Morning_Trap_Reversal | 34 → 32 | 20.6 → 25.0 | -0.147 → -0.123 (+0.024 ↑) | 0.0 → 0.0 | 1.25 → 1.32 | 0.83 → 0.89 | -5.0 → -3.95 |

## Exit mix delta

### BOS_Scalp_1m
- Before: SL=49.0%, TP1=7.8%, time_stop=43.1%
- After:  SL=49.0%, TP1=7.8%, time_stop=43.1%

### Engulfing_Bar_V056
- Before: SL=41.2%, TP1=5.9%, time_stop=52.9%
- After:  SL=37.5%, TP1=6.2%, time_stop=56.2%

### Liquidity_Sweep_Scalp
- Before: SL=39.2%, TP1=5.9%, time_stop=54.9%
- After:  SL=39.2%, TP1=5.9%, time_stop=54.9%

### Morning_Trap_Reversal
- Before: SL=76.5%, TP1=20.6%, eod=2.9%
- After:  SL=71.9%, TP1=21.9%, eod=6.2%

## Verdict (auto)

- **BOS_Scalp_1m**: no change (n=51→51, E[R] -0.11→-0.11)
- **Engulfing_Bar_V056**: partial improvement (E[R] up, still negative) (n=34→32, E[R] -0.101→-0.089)
- **Liquidity_Sweep_Scalp**: no change (n=51→51, E[R] -0.029→-0.029)
- **Morning_Trap_Reversal**: partial improvement (E[R] up, still negative) (n=34→32, E[R] -0.147→-0.123)

## Interpretation

**What the patch did right** (mechanical, expected):
- BE moved 1.0R → 2.15R: fewer winners stopped at break-even before they hit TP1. Visible in WR climb (20.6% → 25.0%), TP1 share up (20.6% → 21.9%), SL share down (76.5% → 71.9%).
- max_duration extended (now honored — Morning_Trap is DAYTRADE, not subject to PHASE3B SCALP cap): some trades that previously eod'd or exited prematurely now have room. eod% 2.9% → 6.2% — patch transferred a few SL-bound trades into longer-duration ones.

**Why it didn't cross zero**:
- The losers still dominate. avg |mae_r| went **up** (0.83 → 0.89) — keeping trades alive longer with a wider BE means losers absorb more risk before exit.
- The gain on winners (avg peak_R 1.25 → 1.32) does not offset losses at this WR level. To breakeven at 25% WR, average winner needs ~3R net; we observe ~1.3R peak with patch.
- **Conclusion: the BE/duration patch was the right calibration but does not fix a 25% WR signal**. Morning_Trap_Reversal at its current detector quality cannot be edge-positive via TP/SL tuning alone.

**Why other targets didn't move**:
- Liquidity_Sweep_Scalp + BOS_Scalp_1m unchanged: no patch applied, identical trades.
- Engulfing_Bar_V056 small drift (-0.101 → -0.089, n 34 → 32) is competition/cap effects from Morning_Trap's longer hold time changing other allowlists — not signal.

## Cross-check vs B1 reviewer caveats

The B1 report flagged Morning_Trap as `LOW_WIN_SAMPLE` (only 7 wins). B2 result confirms instability: percentile-derived BE/max_dur improved metrics on the same corpus but does not generalize to a different result. Sample size remains the limiting factor for Morning_Trap — and the larger lesson is that **TP/SL calibration only helps when the underlying signal already produces positive expectancy on average MFE vs MAE**. Morning_Trap's MAE distribution is wide enough that wider BE just gives losers more rope.

## Decision (per validated Option A)

User-validated branch (2026-04-19): "If Morning_Trap E[R]>0 → try Liquidity_Sweep next. If still negative → pivot to Phase C.1 (VWAP/volume filters)".

Morning_Trap remains negative → **pivot to Phase C.1**.

Calibration alone has hit diminishing returns. Next bet is signal filtering (VWAP regime gate, volume confirmation) to **reject** the worst Morning_Trap setups before they enter, not to widen post-entry tolerances. This addresses the WR floor directly — a smaller number of higher-WR trades changes the breakeven math more than wider BE on the existing distribution.

## Anti-overfit (B2.3, partial)

Plan required E[R]_test ≥ 0.5 × E[R]_train split (jun+aug train, oct+nov test). With Morning_Trap at -0.123 over 32 trades total, splitting yields ~16 trades per fold — too few for a reliable train/test split. Skipping this gate: the calibration didn't pass the absolute gate (E[R]>0), so overfit verification on a failing patch is moot.

## Artifacts

- Patched YAML: [backend/knowledge/playbooks.yml](../../knowledge/playbooks.yml) Morning_Trap_Reversal entry
- Archived snapshot: [backend/knowledge/campaigns/audit_fair_v2_morningtrap.yml](../../knowledge/campaigns/audit_fair_v2_morningtrap.yml)
- B2 corpus: `backend/results/labs/mini_week/b2_morningtrap_v1/` (4 weeks, manifest + parquets)
- Comparison script: [backend/scripts/compare_calibration.py](../../scripts/compare_calibration.py)
- Parallel investigation: [bos_scalp_duration_anomaly.md](bos_scalp_duration_anomaly.md) (BOS_Scalp YAML max_duration silently ignored, blocks BOS_Scalp calibration)

## Next steps (Phase C.1, queued)

1. Activate `vwap_regime: near` on mean-reversion playbooks (incl. Morning_Trap_Reversal).
2. Activate `volume_gate_ratio` on breakouts (BOS_Scalp_1m if duration anomaly fixed first).
3. Re-run 4 weeks, single-filter-at-a-time, measure E[R] delta.
4. Defer BOS_Scalp_1m calibration until PHASE3B_PLAYBOOKS gate is fixed (engine-correctness PR).
