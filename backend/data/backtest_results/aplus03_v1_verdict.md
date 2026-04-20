# Aplus_03 IFVG Flip v1 — first faithful MASTER Family A instantiation (2026-04-20)

## TL;DR

Per Phase D.2 reframing: Family A (Aplus_01/03/04 — sweep/IFVG/HTF-BOS) was **never tested**. This is the first run.

**Correction 2026-04-20 (post-run)**: the v1 run initially wired two playbooks (`Aplus_03_IFVG_Flip_5m_LAB` + `Aplus_03_IFVG_Flip_5m_SAFE`) as "same signal, different gates". This misrepresents the LAB/SAFE pipeline — LAB and SAFE are a **promotion tier**, not two YAML variants. A playbook has ONE config and lives in LAB until it's proven winning, then gets promoted to SAFE (same config, added to paper/live allowlist). The `_SAFE` variant was deleted from YAML; the playbook is now `Aplus_03_IFVG_Flip_5m` (LAB-only until it earns a promotion). The 14-trade `_SAFE` subset below is kept for historical reference but **not representative of the signal going forward**.

**Result (Aplus_03_IFVG_Flip_5m, n=47)**: E[R]=**-0.074**, WR=36%, peak_R p60=**0.68R**, 0/47 TPs hit. 2/4 weeks positive (jun +0.025, aug +0.020), 2/4 negative (oct -0.068, nov -0.213).

**Product verdict**: the signal is not dead — 47 trades over 4 weeks with WR 36% and peak_R p60 = 0.68R clears the 0.6R signal-quality bar used to flag Morning_Trap/BOS_Scalp/Liquidity_Sweep as non-calibrable. But the TP structure (fixed 2.0/4.0R) is demonstrably unreachable: peak_R p80 = 1.06R means 80% of trades never see 1R of MFE. A **minimal B1-like calibration is warranted**: drop TP1 from 2.0R → 0.68R (peak_R p60), BE 1.5R → 0.40R, re-run 4 weeks. If it crosses E[R]>0 on ≥3/4 weeks → first playbook eligible for LAB → SAFE promotion. If it fails → Family A detector as-wired has no edge on SPY/QQQ 5m.

## Fidelity to sources

Source: `MASTER_FINAL.txt::How_To_Start_Day_Trading_As_A_Beginner_In_2025...` + [playbooks_Aplus_from_transcripts.yaml::Aplus_03_IFVG_Flip_from_FVG_Invalidation](../../knowledge/playbooks_Aplus_from_transcripts.yaml).

**Faithful**:
- Detector: [ifvg.py](../../engines/patterns/ifvg.py) emits on 5m close invalidating a same-direction FVG (bullish FVG fills-through-low → bearish IFVG fires, mirror).
- SL structural via `pattern.price_level` (opposite edge of the invalidated zone) — routed through `SWING` SL logic in [setup_engine_v2.py:382-396](../../engines/setup_engine_v2.py#L382-L396).
- Setup TF = 5m as per transcript.
- Single config: `enabled_in_modes: [AGGRESSIVE]` (LAB only). HTF bias gate OFF, ADX gate OFF, window 09:30-15:00 NY. Keeping it permissive so the signal gets a fair first measurement; if it proves winning after calibration, it gets promoted to SAFE mode via allowlist (no config change).

**Honest approximations** (documented in [manifest.json](../../results/labs/mini_week/aplus03_v1/manifest.json)):
1. **Trigger A only** — entry on invalidation close. Trigger B (retest into invalidated zone) not wired in v1. Per transcript, Trigger B is higher-probability, so this v1 excludes the better half of the setup.
2. **TP = fixed RR** (2.0/4.0 LAB, 2.5/5.0 SAFE). Transcript prescribes "next liquidity pool" — not implemented in engine (same gap as the 7 other "MASTER faithful" per Phase D.2 audit).
3. **HTF bias in SAFE** = intraday pivot 1H/4H via [market_state.py](../../engines/market_state.py), not D/4H draw-on-liquidity.
4. **`patterns_config.yml::ifvg.min_displacement_pct`** patched 0.05 → 0.0005. Previous 5% is unreachable on SPY/QQQ 5m (typical wick = 5-50 bps). Comment added in YAML explaining the rationale. This is a small engine fix, not a data hack.

## Results

### 4-week aggregate (Aplus_03_IFVG_Flip_5m — current YAML)

| | n | E[R] | WR | total_R | peak_R p60 | peak_R p80 | \|mae_R\| p75 | time_stop% | SL% | TP% |
|-|---|------|-----|---------|-------------|-------------|----------------|-------------|------|------|
| Aplus_03_IFVG_Flip_5m | **47** | **-0.074** | **36.2%** | -3.49 | **0.68** | 1.06 | 1.01 | 53% | 47% | **0%** |

### Orphaned SAFE-variant subset (deleted from YAML, kept for reference only)

| | n | E[R] | WR | total_R | peak_R p60 | TP% |
|-|---|------|-----|---------|-------------|------|
| Aplus_03_IFVG_Flip_5m_SAFE (deleted) | 14 | -0.142 | 21.4% | -1.99 | 0.52 | 0% |

The SAFE-variant YAML was removed after the run (LAB/SAFE is a promotion tier, not two config variants). These 14 trades ran with HTF bias gate + ADX ≥ 20 + 09:45-12:00 window — **not representative** of the current `Aplus_03_IFVG_Flip_5m` signal.

Exit mix: **0 TPs hit across 47 trades**. All wins are either `time_stop` with positive r_multiple (trailing/BE captured partial MFE) or `SL` labeled after breakeven was pushed (trailing SL hit in profit).

### Per-week stability (Aplus_03_IFVG_Flip_5m)

| week | n | E[R] | verdict |
|------|---|------|---------|
| jun_w3 | 7 | +0.025 | positive, small sample |
| aug_w3 | 14 | +0.020 | positive, meaningful sample |
| oct_w2 | 11 | -0.068 | mild negative |
| nov_w4 | 15 | **-0.213** | crash week — kills aggregate |

Direction balance: LAB 26 LONG / 21 SHORT; SAFE 8 LONG / 6 SHORT. No directional bias artifact.

### Orphaned SAFE per-week (deleted variant, reference only)

| week | n | E[R] |
|------|---|------|
| jun_w3 | 4 | -0.163 |
| aug_w3 | 3 | +0.001 |
| oct_w2 | 3 | -0.102 |
| nov_w4 | 4 | -0.260 |

Not representative of current config. Variant removed from YAML.

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

**B1-like TP/BE patch on the single Aplus_03_IFVG_Flip_5m config**:

```yaml
Aplus_03_IFVG_Flip_5m.take_profit_logic:
  min_rr: 0.70       # was 2.0
  tp1_rr: 0.70       # was 2.0 — peak_R p60 observed
  tp2_rr: 1.20       # was 4.0 — peak_R p80 observed
  breakeven_at_rr: 0.40  # was 1.5 — push to BE well before TP1
  trailing_trigger_rr: 0.50  # was 1.0
  trailing_offset_rr: 0.25  # was 0.5
```

This converts the playbook from "fixed RR optimism" to "empirical-MFE harvesting". **Does not touch the signal.** Re-run 4 weeks caps-active, gate = E[R] > 0 with n ≥ 40 AND per-week ≥ 3/4 non-negative.

If this calibration fails, the conclusion is "Family A detector as wired has no edge on SPY/QQQ 5m 4 weeks" — a stronger statement than the current "Family A untested".

If it succeeds, Aplus_03_IFVG_Flip_5m stays in LAB as a positive playbook, available for further polish (extended period, detector refinement, Trigger B wiring). **Promotion to SAFE is not automatic** — it is a manual user decision taken once, just before paper trading, when a curated set of polished positive playbooks is selected. A positive backtest does not trigger promotion.

## Sanity notes

- Trade timestamps span 09:30-15:00 US open hours (after TZ conversion) — no clock issues.
- Direction mix roughly 55/45 LONG/SHORT on LAB, no directional skew pathology.
- `match_grade` distribution across LAB trades: A/B/C/A+ all represented — scoring not binary.
- `peak_r` values observed up to 1.37R (jun_w3 QQQ LONG) confirming MFE tracking works.
- Exit labels "SL" on winning trades reflect **trailing stops hit after BE ratchet** (normal engine behavior, not bug).

## Artifacts

- [Run dir](../../results/labs/mini_week/aplus03_v1/)
- [manifest.json](../../results/labs/mini_week/aplus03_v1/manifest.json)
- YAML entry: [playbooks.yml:1816](../../knowledge/playbooks.yml#L1816) (single config, LAB-only)
- Detector: [ifvg.py](../../engines/patterns/ifvg.py)
- Threshold patch: [patterns_config.yml:15](../../knowledge/patterns_config.yml#L15)
- Transcript spec: [playbooks_Aplus_from_transcripts.yaml](../../knowledge/playbooks_Aplus_from_transcripts.yaml)
- Related: [tf_faithfulness_audit_v1_verdict.md](tf_faithfulness_audit_v1_verdict.md), [bias_audit_v1_verdict.md](bias_audit_v1_verdict.md)
