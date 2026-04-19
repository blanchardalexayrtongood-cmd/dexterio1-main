# Phase C.1 — Morning_Trap_Reversal vwap_regime verdict (2026-04-20)

## TL;DR

**Patch applied**: `context_requirements.vwap_regime: true` on Morning_Trap_Reversal only (single-filter test per plan C.1).

**Result on 4-week corpus** (`c1_morningtrap_vwap_v1`, same conditions as B2):

- Morning_Trap_Reversal: E[R] **-0.123 → -0.081** (Δ +0.042), WR 25.0% → **28.1%**, n 32 (unchanged), total_R -3.95 → **-2.59**.
- Other 3 targets: near-identical (Liquidity_Sweep noise -0.029 → -0.034, Engulfing unchanged, BOS_Scalp noise +0.001).

**Cumulative Morning_Trap improvement across 3 patches**: -0.147 (B1 baseline) → -0.123 (B2) → **-0.081** (C.1). Still negative.

## Per-playbook delta

| playbook | n | WR% (B2→C1) | E[R] (B2→C1, Δ) | total_R (B2→C1) | verdict |
|---|---|---|---|---|---|
| Morning_Trap_Reversal | 32→32 | 25.0 → 28.1 | -0.123 → **-0.081** (+0.042 ↑) | -3.95 → -2.59 | **partial improvement — still negative** |
| Engulfing_Bar_V056 | 32→32 | 37.5 → 37.5 | -0.089 → -0.089 (=) | -2.84 → -2.86 | no change (no patch) |
| Liquidity_Sweep_Scalp | 51→51 | 41.2 → 39.2 | -0.029 → -0.034 (-0.005 ↓) | -1.48 → -1.76 | noise (no patch) |
| BOS_Scalp_1m | 51→51 | 37.3 → 37.3 | -0.110 → -0.109 (+0.001 =) | -5.59 → -5.57 | noise (no patch) |

## Mechanic — why trade count is unchanged

vwap_regime filter DID reduce Morning_Trap matches substantially:

| week | matches B2 → C.1 | setups B2 → C.1 | trades B2 → C.1 |
|---|---|---|---|
| jun_w3 | 624 → 514 (-17.6%) | 493 → 407 | 8 → 8 |
| aug_w3 | 780 → 521 (-33.2%) | 613 → 406 | 10 → 10 |
| oct_w2 | 780 → 435 (-44.2%) | 609 → 341 | 6 → 6 |
| nov_w4 | 780 → 585 (-25.0%) | 617 → 468 | 8 → 8 |

**Binding constraint**: `max_setups_per_session: 2` on Morning_Trap_Reversal. The filter reduces the *pool* of eligible setups, but the first 2/session fire regardless. So C.1 effectively picked *different* 2/session pairs — vwap-aligned ones — without changing the daily volume.

This is the mechanism behind the +0.042 improvement: the vwap-aligned subset has marginally higher WR (1 extra winner across 32 trades), and the winners captured slightly more peak (avg peak_R 1.32 → 1.41).

## Interpretation

### What worked
- **vwap_regime does select marginally better setups**, consistent with the "trade with VWAP bias" intuition.
- Morning_Trap is closer to breakeven after 3 cumulative patches: -0.147 → -0.081.

### What didn't work
- **Still E[R]<0**. At WR 28%, average winner ≈1.41R, average loser ≈0.86R → the math still doesn't close. Breakeven at 28% WR requires winner/loser ratio ≈2.6, we observe ≈1.64.
- **Diminishing returns**: B2 +0.024, C.1 +0.042. Next filter would likely yield +0.01 to +0.02 at best — and may start reducing trade count enough to break the sample.
- **Signal is the ceiling**: 3 rounds of post-entry/post-filter tweaks moved E[R] by +0.066 total. The B1 "SIGNAL_QUALITY_SUSPECT" flag was correct — the underlying detector produces setups that don't have enough directional follow-through.

### Side-effects on non-patched playbooks
- Liquidity_Sweep_Scalp regressed -0.005 R. Reason: Morning_Trap's modified setup selection changed the trade queue at cap boundaries (cooldown shared at session level). Not a real signal change — expected noise from cap-interaction.

## Decision point

Per the earlier decision tree (Option A branch):

> *"If Morning_Trap still negative: pivot to Phase C.1 (VWAP/volume filters)."* — done.

Three possible next moves:

### (A) Stack a 2nd filter on Morning_Trap
Add `adx_min` or `volume_gate_ratio` alongside vwap_regime. Likely to reduce trades below the cap constraint and produce +0.01–0.02 more. **Value**: marginal. **Risk**: over-fitting a filter stack on 32 trades.

### (B) Accept Morning_Trap ceiling, move to other playbooks
Apply C.1 VWAP/volume filters to Liquidity_Sweep_Scalp (HIGH confidence B1 target) and re-run. Or attempt the C.0 re-test (4 ELI playbooks in isolation) to see if they trade normally when not overshadowed by DAY_Aplus_1 spam.

### (C) Declare Morning_Trap a soft failure, escalate signal redesign
The 3-patch progression (BE, max_dur, vwap_regime) hit a -0.08 asymptote. The next honest step is not more filters — it's asking whether the Morning_Trap *detector* itself (trap reversal on first hour NY) has any edge at current market conditions. This is a Phase C.3 (HTF bias gate) or Phase D (signal audit) question, not a Phase C.1 continuation.

## Recommendation

**Option B, restricted to Liquidity_Sweep_Scalp only.**

Reasons:
- Liquidity_Sweep was B1's HIGH-confidence target (not LOW like Morning_Trap) — more statistical meat.
- Time_stop 55% suggests it's sitting on unrealized moves; VWAP regime may help the long/short bias decision.
- C.0 re-test can wait — root cause is identified and documented (audit artifact, not a real bug).

**Do NOT** stack a 2nd filter on Morning_Trap yet. Accept the -0.08 floor as evidence of signal-quality ceiling and demand a redesign case before further iteration.

## Artifacts

- Patched YAML: [backend/knowledge/playbooks.yml](../../knowledge/playbooks.yml) Morning_Trap_Reversal `context_requirements.vwap_regime: true`
- Archive snapshot: [backend/knowledge/campaigns/audit_c1_morningtrap_vwap.yml](../../knowledge/campaigns/audit_c1_morningtrap_vwap.yml)
- Corpus: `backend/results/labs/mini_week/c1_morningtrap_vwap_v1/`
- Related: [b2_morningtrap_verdict.md](b2_morningtrap_verdict.md), [c0_execution_layer_root_cause.md](c0_execution_layer_root_cause.md)
