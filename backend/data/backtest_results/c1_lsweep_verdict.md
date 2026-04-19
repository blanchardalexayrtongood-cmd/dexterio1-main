# Phase C.1 — Liquidity_Sweep_Scalp vwap_regime verdict (2026-04-20)

## TL;DR

**Patch applied**: `context_requirements.vwap_regime: true` on Liquidity_Sweep_Scalp (single-filter test, second target after Morning_Trap).

**Result on 4-week corpus** (`c1_lsweep_vwap_v1`, baseline = `c1_morningtrap_vwap_v1` so the delta isolates Liquidity_Sweep's vwap_regime):

- Liquidity_Sweep_Scalp: E[R] **-0.034 → -0.035** (Δ -0.001), n 51 → **50** (1 trade dropped), WR 39.2 → 40.0%, total_R -1.76 → -1.73.
- Engulfing_Bar_V056 (no patch): E[R] -0.089 → -0.103 (Δ -0.014). Cap-competition noise from Liquidity_Sweep selecting different pairs.
- Morning_Trap_Reversal (no further patch — already C.1): unchanged.
- BOS_Scalp_1m (no patch): unchanged.

**Verdict: vwap_regime had near-zero effect on Liquidity_Sweep_Scalp.** Filter is not a productive lever for this signal.

## Per-playbook delta

| playbook | n (B→A) | WR% (B→A) | E[R] (B→A, Δ) | total_R (B→A) | verdict |
|---|---|---|---|---|---|
| Liquidity_Sweep_Scalp | 51 → **50** | 39.2 → 40.0 | -0.034 → **-0.035** (-0.001 ↓) | -1.76 → -1.73 | **null effect — filter inert** |
| Engulfing_Bar_V056 | 32 → 32 | 37.5 → 37.5 | -0.089 → -0.103 (-0.014 ↓) | -2.86 → -3.31 | competition noise (no patch) |
| Morning_Trap_Reversal | 32 → 32 | 28.1 → 28.1 | -0.081 → -0.080 (+0.001 =) | -2.59 → -2.57 | unchanged (already C.1) |
| BOS_Scalp_1m | 51 → 51 | 37.3 → 37.3 | -0.109 → -0.109 (=) | -5.57 → -5.57 | no change (no patch) |

## Mechanic — same binding-cap pattern as Morning_Trap

The vwap_regime filter DID reduce Liquidity_Sweep matches substantially, but `max_setups_per_session: 3` is the binding constraint:

| week | matches B→A | setups B→A | trades B→A |
|---|---|---|---|
| jun_w3 | 512 → 432 (-15.6%) | 408 → 343 | 12 → 12 |
| aug_w3 | 648 → 419 (-35.3%) | 506 → 325 | 15 → 15 |
| oct_w2 | 650 → 362 (-44.3%) | 510 → 286 | 12 → 11 |
| nov_w4 | 617 → 469 (-24.0%) | 498 → 381 | 12 → 12 |

Same mechanic as Morning_Trap C.1: the filter shrinks the *pool* of eligible setups by 15-44% but the cap fires the first 3/session anyway. Net result: 1 trade dropped across 200+ session-slots — and that 1 trade was a marginal loser (total_R improved +0.03 R).

**Critical difference vs Morning_Trap**: on Morning_Trap, the vwap-aligned subset had marginally better WR (1 extra winner across 32 trades, +0.042 E[R] gain). On Liquidity_Sweep, the vwap-aligned subset is **statistically indistinguishable** from random — the filter swapped trades around but the new subset's WR (40%) is essentially the same as the old (39.2%).

## Interpretation

### What this confirms
- **vwap_regime is not a universal edge improver.** It helped Morning_Trap (+0.042 E[R]) and did nothing for Liquidity_Sweep. The Morning_Trap gain may have been signal-specific (trap reversals + VWAP context have a real interaction) rather than a general rule.
- **Liquidity_Sweep's E[R] = -0.034 sits near a hard floor.** Three filter/cap variations (B0.4 baseline, MT C.1 sideeffect, LSweep C.1) have all produced E[R] in [-0.029, -0.035] band. The signal asymptotes here.

### What this rules out
- Stacking another filter on Liquidity_Sweep is unlikely to produce >+0.02 E[R]. The `max_setups_per_session: 3` cap will keep absorbing pool reductions until the pool drops below ~3/session/symbol — at which point we'd lose trades without selecting *better* ones.
- The B1 `SIGNAL_QUALITY_SUSPECT` flag on Liquidity_Sweep (peak_R p60 = 0.57R) was correct: the signal doesn't produce enough directional follow-through, and post-filter selection can't fix that.

### Cumulative C.1 progression (E[R] across 2 targets)

| target | B0.4 baseline | C.1 result | Δ |
|---|---|---|---|
| Morning_Trap_Reversal | -0.147 | **-0.081** | +0.066 (after B2 + C.1) |
| Liquidity_Sweep_Scalp | -0.029 | **-0.035** | -0.006 (no real change) |

C.1 gave Morning_Trap a real (but insufficient) bump and gave Liquidity_Sweep nothing. The Phase C.1 hypothesis ("VWAP regime improves signal quality") is **partially supported** for Morning_Trap, **rejected** for Liquidity_Sweep.

## Decision point — back to the Option B branch

From [c1_vwap_verdict.md](c1_vwap_verdict.md):

> *"If Liquidity_Sweep also asymptotes, escalate to signal redesign / detector audit, not more filters."*

Liquidity_Sweep asymptoted. Three honest paths forward:

### (A) Try `volume_gate_ratio: 1.5` on Liquidity_Sweep
Different filter family (volume confirmation vs VWAP bias). Sweeps are by definition volume events, so a volume gate is theoretically more aligned with the signal mechanism than vwap_regime. **Cost**: 1 backtest. **Risk**: same binding-cap mechanic likely makes it inert. **Value**: 1 honest data point on whether *any* context filter can move this signal.

### (B) Accept C.1 inconclusive, escalate to detector/signal audit
Per the rule established in C.1 Morning_Trap verdict (when filters asymptote, the signal is the ceiling). Liquidity_Sweep at WR 40%, peak_R p60 = 0.57R has a math problem the engine can't solve. Either:
- Re-examine the detector ([what defines a "liquidity sweep" in code?](../../engines/detectors/)) — is it firing on real microstructure events or pattern-matching noise?
- Escalate to Phase C.3 HTF bias gate (different mechanism — restricts WHEN we trade, not WHICH setups within a session).
- Or drop Liquidity_Sweep from the calibration portfolio entirely and refocus.

### (C) Pivot to PHASE3B_PLAYBOOKS engine fix to unlock BOS_Scalp_1m
[bos_scalp_duration_anomaly.md](bos_scalp_duration_anomaly.md) documents that BOS_Scalp_1m has wins lasting 120m but `max_duration_minutes` from YAML is ignored because the playbook isn't in the `frozenset({"NY_Open_Reversal", "News_Fade", "Liquidity_Sweep_Scalp"})` allowlist. This is an engine-correctness gap, not a calibration question. Fixing it unlocks BOS_Scalp's calibration which has been blocked since B1.

## Recommendation

**Option C — fix the PHASE3B_PLAYBOOKS gate.**

Reasons:
- BOS_Scalp_1m has the largest sample (n=51, same as Liquidity_Sweep) and `DURATION_ANOMALY` flag — the most likely candidate for a real calibration win that hasn't been tested.
- Engine-correctness fix has predictable scope (frozenset → list, or remove the gate entirely) vs. the open-ended "audit the Liquidity_Sweep detector" branch.
- Phase C.1 has now produced 2 data points (Morning_Trap +0.042, Liquidity_Sweep 0.0). Adding a 3rd context filter test before fixing engine gaps is suboptimal sequencing.

**Do NOT** add another filter to Liquidity_Sweep (Option A) without first deciding whether to keep it in the portfolio at all. At E[R] = -0.034 the playbook is approximately neutral but consistently negative across 3 corpus variants — that's the signature of a low-edge signal, not a calibration problem.

## Artifacts

- Patched YAML: [backend/knowledge/playbooks.yml](../../knowledge/playbooks.yml) Liquidity_Sweep_Scalp `context_requirements.vwap_regime: true`
- Archive snapshot: [backend/knowledge/campaigns/audit_c1_lsweep_vwap.yml](../../knowledge/campaigns/audit_c1_lsweep_vwap.yml)
- Corpus: `backend/results/labs/mini_week/c1_lsweep_vwap_v1/`
- Related: [c1_vwap_verdict.md](c1_vwap_verdict.md), [b2_morningtrap_verdict.md](b2_morningtrap_verdict.md), [bos_scalp_duration_anomaly.md](bos_scalp_duration_anomaly.md)
