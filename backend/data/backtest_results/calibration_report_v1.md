# Phase B1 — calibration_report_v1

**Source corpus:** `calib_corpus_v1` (4 weeks, caps actives, allowlist 4 targets, 170 trades total).
**Method:** adaptive percentiles by win count (HIGH/MEDIUM/LOW confidence). SL from p95/p85/p80 of `|mae_r|` on wins +10% buffer. TP1 from p60/p55/p50 of `peak_r` on non-SL exits. Max duration from p75/p70/p65 of wins. Breakeven at 0.7×TP1.
**Deferred to B2:** trailing grid (needs per-bar MFE series — not in trades parquet). Current trailing values kept as-is.

## ⚠️ Reviewer caveats (READ BEFORE APPLYING)

### Morning_Trap_Reversal
- **LOW_WIN_SAMPLE** — only 7 wins. Proposed SL (0.59R) sits on thin statistics; expect instability in B2 validation.

### Engulfing_Bar_V056
- **LARGE_TP1_CUT** — current TP1=2.0R → 0.68R (drop of 66%). Validate against B2 re-run before accepting — extreme cuts have overfit risk.
- **HIGH_TIME_STOP** — 53% of trades exit on time_stop. Either max_duration was too short or TP1 unreachable. Monitor in B2.

### BOS_Scalp_1m
- **SIGNAL_QUALITY_SUSPECT** — proposed TP1=0.22R is below 0.5R. With full-SL losses at ~1R and winners capped at 0.22R, break-even WR would need 82% (currently 37%). Calibrating TP down to catch MFE may not fix negative E[R] — the signal itself is weak.
- **LARGE_TP1_CUT** — current TP1=1.5R → 0.22R (drop of 85%). Validate against B2 re-run before accepting — extreme cuts have overfit risk.
- **DURATION_ANOMALY** — current YAML max_duration=15m but observed winner durations reach 120m. Either YAML is ignored by engine (bug) or `time_stop` triggers on a different limit. Investigate before applying.

### Liquidity_Sweep_Scalp
- **SIGNAL_QUALITY_SUSPECT** — proposed TP1=0.28R is below 0.5R. With full-SL losses at ~1R and winners capped at 0.28R, break-even WR would need 78% (currently 41%). Calibrating TP down to catch MFE may not fix negative E[R] — the signal itself is weak.
- **LARGE_TP1_CUT** — current TP1=1.5R → 0.28R (drop of 81%). Validate against B2 re-run before accepting — extreme cuts have overfit risk.
- **HIGH_TIME_STOP** — 55% of trades exit on time_stop. Either max_duration was too short or TP1 unreachable. Monitor in B2.

### Overall verdict

- **Safest apply**: Morning_Trap_Reversal only (small TP/BE/duration tweaks, TP1≥3R preserved).
- **Conditional apply**: Liquidity_Sweep_Scalp (HIGH confidence on percentiles) — but proposed TP1=0.28R is a red flag for signal weakness, not TP misplacement. Consider A/B in B2 (current YAML vs patch).
- **Hold / investigate first**: BOS_Scalp_1m (duration anomaly) + Engulfing_Bar_V056 (large TP1 cut, signal-quality flag).
- **Root cause likely upstream**: the cluster of low proposed TP1s (<0.7R) across 3/4 playbooks suggests the detectors fire on weak setups, not that TP1 is mis-placed. Phase C (regime/VWAP filters) may fix more than Phase B1 calibration can.

## Summary table

| playbook | n | wins | WR | E[R] | avg_peak_R | time_stop% | conf | SL curr→new | TP1 curr→new | BE curr→new | MaxDur curr→new |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Morning_Trap_Reversal | 34 | 7 | 20.6% | -0.147 | +1.25 | 0% | LOW | SWING→0.59R | 3.00→3.07 | 1.00→2.15 | —→155 |
| Engulfing_Bar_V056 | 34 | 12 | 35.3% | -0.101 | +0.62 | 53% | MEDIUM | —→0.46R | 2.00→0.68 | 1.00→0.48 | 120→120 |
| BOS_Scalp_1m | 51 | 19 | 37.3% | -0.110 | +0.45 | 43% | MEDIUM | —→0.39R | 1.50→0.22 | 999.00→— | 15→120 |
| Liquidity_Sweep_Scalp | 51 | 21 | 41.2% | -0.029 | +0.56 | 55% | HIGH | —→0.45R | 1.50→0.28 | 1.00→— | 30→30 |

## Morning_Trap_Reversal

- **Corpus**: n=34, wins=7, WR=20.6%, E[R]=-0.147, avg peak_R=+1.25, time_stop=0%
- **Confidence**: `LOW` (percentiles used: SL=p80 MAE, TP=p50 peak, dur=p65)
- **Wins by exit**: {'TP1': 7}

**Current vs proposed:**
- SL: `SWING/trap_extreme` → proposed R-cap ≤ **0.59R** (kept structural if SWING — new cap just protects downside)
- TP1: `3.00R` → **3.07R**
- TP2: `5.00R` (unchanged)
- Breakeven: `1.00R` → **2.15R**
- Max duration: `—m` → **155m**
- Trailing: `trigger=—R, offset=—R` (unchanged — B2 grid search)

**YAML delta (`calibration_patch_v1.yml` entry):**
```yaml
- playbook_name: Morning_Trap_Reversal
take_profit_logic:
  breakeven_at_rr: 2.15
max_duration_minutes: 155
```

**Reasoning:**

## Engulfing_Bar_V056

- **Corpus**: n=34, wins=12, WR=35.3%, E[R]=-0.101, avg peak_R=+0.62, time_stop=53%
- **Confidence**: `MEDIUM` (percentiles used: SL=p85 MAE, TP=p55 peak, dur=p70)
- **Wins by exit**: {'time_stop': 6, 'SL': 4, 'TP1': 2}
- **Time-stop wins peak_R**: p50=0.80, p75=1.16

**Current vs proposed:**
- SL: `FIXED/pattern_extreme` → proposed R-cap ≤ **0.46R** (kept structural if SWING — new cap just protects downside)
- TP1: `2.00R` → **0.68R**
- TP2: `4.00R` (unchanged)
- Breakeven: `1.00R` → **0.48R**
- Max duration: `120m` → **120m**
- Trailing: `trigger=1.00R, offset=0.50R` (unchanged — B2 grid search)

**YAML delta (`calibration_patch_v1.yml` entry):**
```yaml
- playbook_name: Engulfing_Bar_V056
take_profit_logic:
  tp1_rr: 0.68
  min_rr: 0.68
  breakeven_at_rr: 0.48
```

**Reasoning:**
- Current TP1=2.0R rarely hit. Corpus p55 of peak_R on non-SL exits = 0.68R → lowering TP1 captures MFE that currently expires at time_stop.

## BOS_Scalp_1m

- **Corpus**: n=51, wins=19, WR=37.3%, E[R]=-0.110, avg peak_R=+0.45, time_stop=43%
- **Confidence**: `MEDIUM` (percentiles used: SL=p85 MAE, TP=p55 peak, dur=p70)
- **Wins by exit**: {'time_stop': 8, 'SL': 7, 'TP1': 4}
- **Time-stop wins peak_R**: p50=0.38, p75=0.59

**Current vs proposed:**
- SL: `FIXED/pattern_extreme` → proposed R-cap ≤ **0.39R** (kept structural if SWING — new cap just protects downside)
- TP1: `1.50R` → **0.22R**
- TP2: `—R` (unchanged)
- Breakeven: `999.00R` → **—R**
- Max duration: `15m` → **120m**
- Trailing: `trigger=0.80R, offset=0.30R` (unchanged — B2 grid search)

**YAML delta (`calibration_patch_v1.yml` entry):**
```yaml
- playbook_name: BOS_Scalp_1m
take_profit_logic:
  tp1_rr: 0.22
  min_rr: 0.22
max_duration_minutes: 120
```

**Reasoning:**
- Current TP1=1.5R rarely hit. Corpus p55 of peak_R on non-SL exits = 0.22R → lowering TP1 captures MFE that currently expires at time_stop.
- Max duration p70 of wins = 120m (current 15m) → increase aligns cutoff with actual winner duration.

## Liquidity_Sweep_Scalp

- **Corpus**: n=51, wins=21, WR=41.2%, E[R]=-0.029, avg peak_R=+0.56, time_stop=55%
- **Confidence**: `HIGH` (percentiles used: SL=p95 MAE, TP=p60 peak, dur=p75)
- **Wins by exit**: {'SL': 12, 'time_stop': 6, 'TP1': 3}
- **Time-stop wins peak_R**: p50=0.44, p75=0.56

**Current vs proposed:**
- SL: `FIXED/local_extreme` → proposed R-cap ≤ **0.45R** (kept structural if SWING — new cap just protects downside)
- TP1: `1.50R` → **0.28R**
- TP2: `2.50R` (unchanged)
- Breakeven: `1.00R` → **—R**
- Max duration: `30m` → **30m**
- Trailing: `trigger=0.80R, offset=0.30R` (unchanged — B2 grid search)

**YAML delta (`calibration_patch_v1.yml` entry):**
```yaml
- playbook_name: Liquidity_Sweep_Scalp
take_profit_logic:
  tp1_rr: 0.28
  min_rr: 0.28
```

**Reasoning:**
- Current TP1=1.5R rarely hit. Corpus p60 of peak_R on non-SL exits = 0.28R → lowering TP1 captures MFE that currently expires at time_stop.
