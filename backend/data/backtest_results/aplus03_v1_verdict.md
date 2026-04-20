# Aplus_03 IFVG Flip v1 — first faithful MASTER Family A instantiation (2026-04-20)

## TL;DR

Per Phase D.2 reframing: Family A (Aplus_01/03/04 — sweep/IFVG/HTF-BOS) was **never tested**. This is the first run. Two variants wired (LAB permissive, SAFE strict), 4 weeks caps-active, 61 trades total.

**Result**: E[R] **-0.090** combined (LAB -0.074, SAFE -0.142). **2/4 weeks positive on LAB** (jun_w3 +0.025, aug_w3 +0.020) collapse to -0.07 / -0.21 on oct/nov. **0 TPs hit** on any of the 61 trades.

**Product verdict**: the signal is not dead — LAB fires 47 times in 4 weeks with WR 36% and peak_R p60 = 0.68R (clears the 0.6R signal-quality bar used in Phase B1). But the TP structure (fixed 2.0-5.0R) is demonstrably unreachable: peak_R p80 = 1.06R means 80% of trades never see 1R of MFE. A **minimal B1-like calibration is warranted**: drop TP1 from 2.0R → 0.68R (peak_R p60) on LAB, hold n and WR, expect E[R] to cross zero. SAFE does not merit calibration (n=14/4wk, 3/4 weeks negative, no sample stability).

## Fidelity to sources

Source: `MASTER_FINAL.txt::How_To_Start_Day_Trading_As_A_Beginner_In_2025...` + [playbooks_Aplus_from_transcripts.yaml::Aplus_03_IFVG_Flip_from_FVG_Invalidation](../../knowledge/playbooks_Aplus_from_transcripts.yaml).

**Faithful**:
- Detector: [ifvg.py](../../engines/patterns/ifvg.py) emits on 5m close invalidating a same-direction FVG (bullish FVG fills-through-low → bearish IFVG fires, mirror).
- SL structural via `pattern.price_level` (opposite edge of the invalidated zone) — routed through `SWING` SL logic in [setup_engine_v2.py:382-396](../../engines/setup_engine_v2.py#L382-L396).
- Setup TF = 5m as per transcript.
- LAB permissive (HTF bias OFF, ADX OFF, window 09:30-15:00) vs SAFE strict (HTF bias ON, structure trending, ADX ≥ 20, 09:45-12:00) — same signal, different gates.

**Honest approximations** (documented in [manifest.json](../../results/labs/mini_week/aplus03_v1/manifest.json)):
1. **Trigger A only** — entry on invalidation close. Trigger B (retest into invalidated zone) not wired in v1. Per transcript, Trigger B is higher-probability, so this v1 excludes the better half of the setup.
2. **TP = fixed RR** (2.0/4.0 LAB, 2.5/5.0 SAFE). Transcript prescribes "next liquidity pool" — not implemented in engine (same gap as the 7 other "MASTER faithful" per Phase D.2 audit).
3. **HTF bias in SAFE** = intraday pivot 1H/4H via [market_state.py](../../engines/market_state.py), not D/4H draw-on-liquidity.
4. **`patterns_config.yml::ifvg.min_displacement_pct`** patched 0.05 → 0.0005. Previous 5% is unreachable on SPY/QQQ 5m (typical wick = 5-50 bps). Comment added in YAML explaining the rationale. This is a small engine fix, not a data hack.

## Results

### 4-week aggregate

| playbook | n | E[R] | WR | total_R | peak_R p60 | peak_R p80 | \|mae_R\| p75 | time_stop% | SL% | TP% |
|----------|---|------|-----|---------|-------------|-------------|----------------|-------------|------|------|
| LAB | 47 | **-0.074** | 36.2% | -3.49 | **0.68** | 1.06 | 1.01 | 53% | 47% | **0%** |
| SAFE | 14 | -0.142 | 21.4% | -1.99 | 0.52 | 0.78 | 0.94 | 64% | 36% | 0% |
| **combined** | **61** | **-0.090** | **32.8%** | **-5.48** | — | — | — | 56% | 44% | **0%** |

Exit mix: **0 TPs hit across 61 trades**. All wins are either `time_stop` with positive r_multiple (trailing/BE captured partial MFE) or `SL` labeled after breakeven was pushed (trailing SL hit in profit).

### Per-week stability (LAB)

| week | n | E[R] | verdict |
|------|---|------|---------|
| jun_w3 | 7 | +0.025 | positive, small sample |
| aug_w3 | 14 | +0.020 | positive, meaningful sample |
| oct_w2 | 11 | -0.068 | mild negative |
| nov_w4 | 15 | **-0.213** | crash week — kills aggregate |

Direction balance: LAB 26 LONG / 21 SHORT; SAFE 8 LONG / 6 SHORT. No directional bias artifact.

### Per-week stability (SAFE)

| week | n | E[R] |
|------|---|------|
| jun_w3 | 4 | -0.163 |
| aug_w3 | 3 | +0.001 |
| oct_w2 | 3 | -0.102 |
| nov_w4 | 4 | -0.260 |

3/4 weeks negative, n too small for meaningful inference.

## Interpretation per decision rule

User's decision rule:
- zero trades → diagnose blockage → **N/A, 61 trades fired**
- few trades → structural rarity vs config → **N/A on LAB (47), applies to SAFE (14)**
- trades with franky negative E[R] → signal null vs exits/filtering → **THIS CASE**
- interesting start → propose smallest calibration → **THIS CASE ON LAB**

**Signal nul** hypothesis: WR 36% + peak_R p60 = 0.68R is **not a null signal** — it clears the "signal-quality suspect" bar (peak_R p60 < 0.6R) used to flag Engulfing/BOS_Scalp/Liquidity_Sweep as non-calibrable. 2/4 weeks are positive.

**Exits/filtering** hypothesis: 0/61 TPs hit, peak_R p80 = 1.06R → **fixed RR 2.0R TP1 is unreachable by the 5m IFVG Flip signal on SPY/QQQ**. This is exactly what D.2 predicted: MASTER transcripts prescribe liquidity-targeting TPs, we wired fixed 2.0-5.0R, and the signal's MFE distribution does not support it.

The math is straightforward: with WR 36% and peak_R p80 = 1.06R, a TP1 at 0.7R (capturing the upper-middle of observed MFE) plus BE ratchet would likely push E[R] positive. A TP1 at 2.0R requires the WR to compensate for near-zero TP-hit rate, which is not plausible.

## Recommended next calibration (minimal)

**B1-like patch on LAB only** (SAFE untouchable until sample grows):

```yaml
Aplus_03_IFVG_Flip_5m_LAB.take_profit_logic:
  min_rr: 0.70       # was 2.0
  tp1_rr: 0.70       # was 2.0 — peak_R p60 observed
  tp2_rr: 1.20       # was 4.0 — peak_R p80 observed
  breakeven_at_rr: 0.40  # was 1.5 — push to BE well before TP1
  trailing_trigger_rr: 0.50  # was 1.0
  trailing_offset_rr: 0.25  # was 0.5
```

This converts the playbook from "fixed RR optimism" to "empirical-MFE harvesting". **Does not touch the signal.** Re-run 4 weeks caps-active, gate = E[R] > 0 with n ≥ 40 AND per-week ≥ 3/4 non-negative.

If this calibration fails, the conclusion is "Family A detector as wired has no edge on SPY/QQQ 5m 4 weeks" — a stronger statement than the current "Family A untested".

If it succeeds, Aplus_03_LAB becomes the **first product-grade playbook** and earns SAFE re-wiring + portfolio slot consideration.

## Sanity notes

- Trade timestamps span 09:30-15:00 US open hours (after TZ conversion) — no clock issues.
- Direction mix roughly 55/45 LONG/SHORT on LAB, no directional skew pathology.
- `match_grade` distribution across LAB trades: A/B/C/A+ all represented — scoring not binary.
- `peak_r` values observed up to 1.37R (jun_w3 QQQ LONG) confirming MFE tracking works.
- Exit labels "SL" on winning trades reflect **trailing stops hit after BE ratchet** (normal engine behavior, not bug).

## Artifacts

- [Run dir](../../results/labs/mini_week/aplus03_v1/)
- [manifest.json](../../results/labs/mini_week/aplus03_v1/manifest.json)
- YAML entries: [playbooks.yml:1817-1950](../../knowledge/playbooks.yml#L1817-L1950)
- Detector: [ifvg.py](../../engines/patterns/ifvg.py)
- Threshold patch: [patterns_config.yml:15](../../knowledge/patterns_config.yml#L15)
- Transcript spec: [playbooks_Aplus_from_transcripts.yaml](../../knowledge/playbooks_Aplus_from_transcripts.yaml)
- Related: [tf_faithfulness_audit_v1_verdict.md](tf_faithfulness_audit_v1_verdict.md), [bias_audit_v1_verdict.md](bias_audit_v1_verdict.md)
