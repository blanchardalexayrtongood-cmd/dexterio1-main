# Bias alignment audit v1 — D/4H bias filtering does NOT unlock edge (2026-04-20)

## TL;DR

First Phase D (signal redesign) workstream: does requiring trades to align with D and/or 4H HTF bias unlock E[R] > 0 on any subset? **No.** On the survivor_v1 corpus (249 trades, 4 weeks):

- **Daily bias**: aligned trades (n=142) E[R]=-0.068 vs counter (n=107) E[R]=-0.108 — Δ=+0.040 in favor of alignment, but neither cohort crosses zero.
- **4H bias**: aligned (n=133) E[R]=**-0.107** vs counter (n=116) E[R]=**-0.060** — **inverts** (counter is better), suggesting many playbooks are intrinsically reversal-style.
- **Combined D∧4H** (both must agree): aligned (n=103) E[R]=-0.099 vs counter (n=77) E[R]=-0.099 — **identical**, the most restrictive filter is also the most pointless.

**No subset with n ≥ 30 crosses E[R] > 0.** The largest positive subset is BOS_Scalp_1m counter on combined (n=17, E[R]=+0.029) — too small to act on.

**Conclusion**: HTF bias gating is **not the missing filter**. The signal-quality ceiling we hit in Phase B/C/Survivor is not relieved by adding D/4H regime alignment.

## Method

- **D bias at trade entry T**: sign(close_D[T-1] − SMA_D_5[T-1]) on resampled daily close from `SPY_1m.parquet` / `QQQ_1m.parquet` (per-symbol).
- **4H bias**: same formula on 4H close.
- **Aligned**: (bullish bias AND LONG) OR (bearish bias AND SHORT). Counter is the opposite.
- **Combined**: both D and 4H aligned in the same direction; otherwise mixed (or counter if both opposed).
- **Tool**: [bias_alignment_audit.py](../../scripts/bias_alignment_audit.py) (read-only, ~190 LOC).

**Caveat on the proxy**: SMA_5 against current close is a thin proxy for "structural HTF bias." The market_state.py engine uses pivot-based structure detection (uptrend/downtrend) which is richer. A more thorough audit would re-derive structural bias per the actual engine. But the proxy is directionally informative — if HTF alignment had a strong effect, we'd see it.

## Results

### Daily alignment (overall)

| cohort | n | E[R] | total_R |
|---|---|---|---|
| aligned | 142 | -0.068 | -9.60 |
| counter | 107 | -0.108 | -11.56 |
| **Δ (aligned − counter)** | | **+0.040** | |

Bias distribution: 191 trades fired into bear-bias periods, 58 into bull-bias periods (asymmetry reflects the 4-week corpus weighting toward declining/ranging weeks).

### 4H alignment (overall)

| cohort | n | E[R] | total_R |
|---|---|---|---|
| aligned | 133 | -0.107 | -14.17 |
| counter | 116 | -0.060 | -6.99 |
| **Δ (aligned − counter)** | | **-0.047** | |

**4H aligned is WORSE.** This signals two possibilities:
1. The SMA_5 proxy on 4H is too noisy (4H close vs 5-bar SMA flips often).
2. Many survivor playbooks are reversal-by-design (Morning_Trap_Reversal, BOS_Scalp_1m, News_Fade) — they fire AGAINST recent momentum on purpose.

### Combined D∧4H (both must agree)

| cohort | n | E[R] | total_R |
|---|---|---|---|
| aligned | 103 | -0.099 | -10.23 |
| counter | 77 | -0.099 | -7.62 |
| mixed | 69 | -0.048 | -3.31 |

**Aligned and counter are identical** (-0.099 each). Mixed is least bad. This is the cleanest evidence that **strict D∧4H gating is not the lever**.

### Per-playbook D-bias delta_ER (sorted)

| playbook | n_aligned | ER_aligned | n_counter | ER_counter | Δ ER |
|---|---|---|---|---|---|
| HTF_Bias_15m_BOS | 1 | +0.672 | 1 | -0.350 | +1.022 |
| EMA_Cross_5m | 2 | +0.072 | 5 | -0.199 | +0.271 |
| RSI_MeanRev_5m | 10 | -0.139 | 9 | -0.408 | +0.269 |
| IFVG_5m_Sweep | 15 | -0.087 | 7 | -0.352 | +0.265 |
| Session_Open_Scalp | 9 | +0.006 | 3 | -0.073 | +0.079 |
| Engulfing_Bar_V056 | 14 | +0.022 | 12 | -0.047 | +0.069 |
| Liquidity_Sweep_Scalp | 21 | -0.032 | 21 | -0.040 | +0.008 |
| FVG_Fill_Scalp | 27 | -0.076 | 7 | -0.073 | -0.003 |
| News_Fade | 7 | -0.023 | 3 | +0.058 | -0.081 |
| BOS_Scalp_1m | 22 | -0.111 | 20 | -0.009 | -0.102 |
| Morning_Trap_Reversal | 10 | -0.248 | 15 | -0.127 | -0.121 |

**Continuation/breakout/mean-rev playbooks** (RSI_MeanRev_5m, IFVG_5m_Sweep, EMA_Cross_5m, Engulfing_Bar_V056) **benefit** from D-bias alignment — but the absolute level stays negative.

**Reversal playbooks** (Morning_Trap_Reversal, BOS_Scalp_1m, News_Fade) are **hurt** by D-bias alignment — they're literally designed to fade, so requiring continuation alignment kills them. Confirms intent-of-design.

### Per-playbook 4H-bias notable inversions

| playbook | n_aligned | ER_aligned | n_counter | ER_counter | Δ ER |
|---|---|---|---|---|---|
| BOS_Scalp_1m | 18 | -0.190 | 24 | +0.033 | **-0.223** |
| Morning_Trap_Reversal | 15 | -0.240 | 10 | -0.078 | -0.162 |
| Liquidity_Sweep_Scalp | 20 | -0.065 | 22 | -0.010 | -0.055 |

BOS_Scalp_1m **counter-4H subset goes positive** (n=24, E[R]=+0.033). This is the most interesting single subset finding — but n is still small for a 4-week sample.

## Findings

1. **Bias gating doesn't relieve the signal-quality ceiling.** Best subset E[R] aligns with broad observation: aligned vs counter splits don't produce E[R] > 0 with meaningful n.

2. **D and 4H disagree about what "alignment" should mean.** D-aligned helps continuation/mean-rev playbooks; 4H-aligned hurts them on aggregate. This is consistent with **playbooks being reversal-style at the 4H scale, continuation-style at the D scale**.

3. **No universal HTF gate exists.** A gate that helps RSI_MeanRev_5m hurts Morning_Trap_Reversal. Per-playbook gates would be required, but per-playbook gates are exactly the per-signal calibration approach that asymptoted in Phase B/C.

4. **One genuine candidate**: BOS_Scalp_1m **counter-4H subset** (n=24, E[R]=+0.033). Worth tagging for further investigation but not actionable on n=24/4 weeks.

5. **The proxy (SMA_5) is a weak read on structural bias.** A re-derivation using market_state.py's `detect_structure` (pivot-based uptrend/downtrend) might give different numbers. Worth a second pass if this audit motivates further work.

## Decision

### What changes
- Nothing in the engine or YAML. This is a read-only audit.

### Implications for Phase D direction
HTF bias alignment was the **first hypothesis** for Phase D (signal redesign). It's now ruled out as the missing filter. The remaining Phase D workstreams are:

1. **TF compression audit**: which playbooks compress 5m/15m concepts to 1m bars and lose information? (Code reading — review playbook detectors vs MASTER specifications.)
2. **Detector preconditions sanity**: do the named detectors (FVG_Fill_V065, Liquidity_Raid_V056, OB_Retest_V004 — the 7 MASTER faithful) actually implement what their names suggest? Cross-reference against MASTER_FINAL.txt transcripts.
3. **Structural bias re-audit**: redo this audit using market_state.py's actual `detect_structure` instead of SMA_5 proxy. If the structural bias result is qualitatively different, the conclusion may need revision.

### Three honest paths forward (decision needed from user)

**(A) Continue Phase D — TF compression audit / detector code review.**
Read the 7 MASTER faithful detectors (FVG_Fill_V065, Range_FVG_V054, Liquidity_Raid_V056, Asia_Sweep_V051, London_Fakeout_V066, Engulfing_Bar_V056, OB_Retest_V004) and Aplus transcripts; check whether implementations match the conceptual MASTER framework (D/4H bias → 15m/5m setup → 1m execution). Cheap (read-only), high information value about WHY signals don't have edge.

**(B) Structural bias re-audit using market_state.py's `detect_structure`.**
This audit used SMA_5 as a bias proxy. The actual engine uses pivot-based structure. If structural bias gives meaningfully different results, today's verdict may be premature. Cheap to run if we can call the engine functions standalone on resampled HTF candles.

**(C) Accept Phase B/C/Survivor/Bias-D verdict as terminal: no edge in current detectors. Pivot to C — Polygon 18 months OR (A) paper as instrumented baseline.**
After 4 phases of monotonic null (fair audit → calibration → portfolio → bias gate), the bear case for the current detector set is well-supported. Either gather more data to detect a thinner edge (Polygon, deferred earlier) or accept current portfolio as near-breakeven.

### Recommendation

**(A) TF compression / detector code review** is the highest-EV next step. (B) is incrementally informative but unlikely to flip the verdict (HTF bias alignment producing zero/negative effect on combined alignment is hard to overturn with a different bias proxy). (C) prematurely closes the door without checking whether the detectors are even faithful to the MASTER specs they claim.

If (A) reveals that 3+ "MASTER faithful" detectors don't actually implement what they claim — that's a signal-edge investigation worth pursuing **before** giving up on detector-driven edge.

## Artifacts

- Script: [backend/scripts/bias_alignment_audit.py](../../scripts/bias_alignment_audit.py)
- Corpus: [backend/results/labs/mini_week/survivor_v1/](../../results/labs/mini_week/survivor_v1/)
- Related: [survivor_v1_verdict.md](survivor_v1_verdict.md), [c2_bosfix_observe_verdict.md](c2_bosfix_observe_verdict.md)
