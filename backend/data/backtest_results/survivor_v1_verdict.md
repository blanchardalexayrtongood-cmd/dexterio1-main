# Survivor portfolio analysis v1 (2026-04-20)

## TL;DR

Ran an isolated 4-week corpus (`survivor_v1`) with **23 candidate playbooks** under production caps (cooldown + session_cap active, kill_switch off, RELAX_CAPS off) to answer the post-Phase B/C question: **does any combination of playbooks have portfolio-level edge, even when no single signal does?**

**Result**: 4 survivors clear the `E[R] ≥ -0.05 AND n ≥ 10` bar. **No combination crosses E[R] > 0 across 4 weeks.** Best 3-pack lands at E[R] = -0.009 (n=48), best pair at E[R] = -0.007 (n=22). Full 4-survivor portfolio = E[R] = -0.022, n=90, total_R = -1.93, max DD = -2.54R.

The portfolio is **near breakeven, not edge-positive**. Diversification is real (News_Fade has negative daily-R correlation with Engulfing_Bar_V056 and Liquidity_Sweep_Scalp at ~-0.34) but not sufficient to flip the sign.

**Cross-contamination finding**: per-playbook E[R] is **environment-dependent** — Morning_Trap_Reversal degraded -0.081 (focused C.1 4-target) → **-0.175** (survivor 23-target run); BOS_Scalp_1m improved -0.118 (C.2 focused) → **-0.062** (survivor run). Slot competition alters which setups fire, which alters per-playbook outcomes. **Single-playbook E[R] is not a portable scalar.**

## Corpus configuration

- **Run**: [backend/results/labs/mini_week/survivor_v1/](../../results/labs/mini_week/survivor_v1/)
- **Weeks**: jun_w3, aug_w3, oct_w2, nov_w4 (same 4 as Phase A fair audit)
- **Caps**: production (cooldown active, `max_setups_per_session` active)
- **Env flags**: `ALLOW_ALL_PLAYBOOKS=true`, `RELAX_CAPS=false`, `DISABLE_KILL_SWITCH=true`
- **Allowlist**: 23 playbooks (excludes confirmed-DENYLIST DAY_Aplus_1, SCALP_Aplus_1, NY_Open_Reversal, ORB_Breakout_5m, plus YAML-disabled Lunch_Range_Scalp)
- **Total trades**: 249 across 15 playbooks that fired (8 silent again)

## Per-playbook table (15 playbooks that fired)

| playbook | n | E[R] | total_R | survivor? |
|---|---|---|---|---|
| HTF_Bias_15m_BOS | 2 | +0.161 | +0.32 | n<10 |
| VWAP_Bounce_5m | 3 | +0.075 | +0.22 | n<10 |
| **News_Fade** | **10** | **+0.001** | **+0.01** | ✅ |
| FVG_Scalp_1m | 3 | -0.009 | -0.03 | n<10 |
| **Engulfing_Bar_V056** | **26** | **-0.010** | **-0.26** | ✅ |
| **Session_Open_Scalp** | **12** | **-0.014** | **-0.17** | ✅ |
| FVG_Fill_V065 | 1 | -0.015 | -0.02 | n<10 |
| **Liquidity_Sweep_Scalp** | **42** | **-0.036** | **-1.52** | ✅ |
| BOS_Scalp_1m | 42 | -0.062 | -2.62 | E[R]<-0.05 |
| FVG_Fill_Scalp | 34 | -0.076 | -2.57 | E[R]<-0.05 |
| EMA_Cross_5m | 7 | -0.121 | -0.85 | E[R]<-0.05 |
| IFVG_5m_Sweep | 22 | -0.171 | -3.77 | E[R]<-0.05 |
| Morning_Trap_Reversal | 25 | -0.175 | -4.38 | E[R]<-0.05 |
| RSI_MeanRev_5m | 19 | -0.267 | -5.07 | E[R]<-0.05 |
| London_Fakeout_V066 | 1 | -0.477 | -0.48 | n<10 |

**4 survivors**: News_Fade, Engulfing_Bar_V056, Session_Open_Scalp, Liquidity_Sweep_Scalp.

## Sub-portfolio combinations (every subset of survivors)

| combo | n | E[R] | total_R | PF |
|---|---|---|---|---|
| News_Fade + Session_Open_Scalp | 22 | **-0.0070** | -0.15 | 0.91 |
| News_Fade + Engulfing | 36 | -0.0070 | -0.25 | 0.94 |
| News_Fade + Engulfing + Session_Open | **48** | **-0.0087** | **-0.42** | **0.92** |
| Engulfing + Session_Open | 38 | -0.0113 | -0.43 | 0.89 |
| All 4 survivors | 90 | -0.0215 | -1.93 | 0.77 |
| News_Fade + Engulfing + Liquidity_Sweep | 78 | -0.0227 | -1.77 | 0.77 |
| Engulfing + Session + Liq_Sweep | 80 | -0.0243 | -1.95 | 0.74 |
| News_Fade + Session + Liq_Sweep | 64 | -0.0261 | -1.67 | 0.68 |
| Engulfing + Liquidity_Sweep | 68 | -0.0262 | -1.78 | 0.73 |
| News_Fade + Liquidity_Sweep | 52 | -0.0289 | -1.51 | 0.67 |
| Session + Liquidity_Sweep | 54 | -0.0312 | -1.68 | 0.59 |

**Best combination = News_Fade + Engulfing + Session_Open_Scalp** (E[R] = -0.009, n=48, PF=0.92). Adding Liquidity_Sweep degrades all combos. Liquidity_Sweep is a survivor by E[R] but its volume drags portfolio math.

## Daily-R correlation (genuine diversification check)

|  | News_Fade | Engulfing | Session_Open | Liq_Sweep |
|---|---|---|---|---|
| News_Fade | 1.00 | **-0.34** | -0.12 | **-0.34** |
| Engulfing | -0.34 | 1.00 | +0.42 | +0.10 |
| Session_Open | -0.12 | +0.42 | 1.00 | +0.20 |
| Liq_Sweep | -0.34 | +0.10 | +0.20 | 1.00 |

**News_Fade has the only meaningful negative correlations** (~-0.34) with Engulfing and Liquidity_Sweep — it's a contra-cycle playbook. Engulfing + Session_Open are positively correlated (+0.42), so they're not great diversifiers of each other.

This explains why best pair = News_Fade + Session_Open (-0.12 corr) and best 3-pack adds Engulfing (negative-corr to News_Fade): the math captures real diversification, but the underlying signal pool is too weak for diversification alone to flip the sign.

## Per-week stability (4-survivor portfolio)

| week | n | total_R | E[R] |
|---|---|---|---|
| 2025-06 (jun_w3) | 22 | -0.57 | -0.026 |
| **2025-08 (aug_w3)** | **35** | **+0.50** | **+0.014** |
| 2025-10 (oct_w2) | 11 | -0.90 | -0.082 |
| 2025-11 (nov_w4) | 22 | -0.96 | -0.044 |

**Only 1/4 weeks positive.** Aug carries the portfolio briefly into the green; the other 3 drag it back. Equity peak +0.60R, final -1.93R, max DD -2.54R.

This is the canonical "no edge" pattern: positive-week spikes that don't compound into a trend.

## Cross-contamination finding (important)

Same playbook, different cohort = different per-playbook E[R]:

| playbook | focused run | survivor run | Δ |
|---|---|---|---|
| Morning_Trap_Reversal | -0.081 (C.1, 4-target) | **-0.175** | -0.094 ↓ |
| BOS_Scalp_1m | -0.118 (C.2, focused) | **-0.062** | +0.056 ↑ |
| Liquidity_Sweep_Scalp | -0.035 (C.1, 4-target) | -0.036 | ~0 |
| Engulfing_Bar_V056 | -0.10 (B1 corpus) | -0.010 | +0.09 ↑ |

**Mechanism**: under production caps (cooldown 5m + `max_setups_per_session` per playbook), running 23 playbooks vs 4 changes the slot allocation order in the risk engine. Some playbooks get a worse subset of their setups (Morning_Trap), others get a better subset (BOS_Scalp, Engulfing) when slot scarcity forces the engine to skip marginal candidates.

**Implication for calibration**: a playbook's "calibrated" E[R] measured in isolation is **not its portfolio E[R]**. Phase B1/B2/C results were optimistic for Morning_Trap and pessimistic for BOS_Scalp/Engulfing. A re-test of Morning_Trap calibration in the survivor cohort would likely show different numbers.

**Implication for next steps**: any portfolio decision must be measured **in the cohort it will run in**, not aggregated from isolated single-playbook tests.

## Decision

### What the data says
1. **No single playbook has 4-week edge** at n ≥ 10 with E[R] > 0 (best is News_Fade at +0.001, n=10).
2. **No combination has 4-week edge.** Best 3-pack reaches -0.009 — essentially breakeven, not edge.
3. **Diversification is real** (News_Fade neg-corr with Engulfing/Liq_Sweep) but quantitatively small.
4. **Per-playbook E[R] is environment-dependent.** Calibration done in isolation does not predict portfolio behavior.

### Three honest paths forward (decision needed from user)

**(A) Accept survivor portfolio as a paper-trading baseline.**
News_Fade + Engulfing + Session_Open_Scalp (E[R]=-0.009, n=48 over 4 weeks). Frame paper as "instrument the engine and measure live slippage" rather than "expect P&L". Risk: paper-trading a near-zero-E[R] portfolio burns time and convinces nothing.

**(B) Pivot to fundamental signal redesign / detector audit (Phase D).**
After 3 single-signal calibration failures + 1 portfolio failure under production caps, the working hypothesis is **the detectors themselves don't capture edge in current SPY/QQQ 1m intraday flow**. Review: which patterns require HTF context (D/4H bias) we're not gating on? Which playbooks compress 5m/15m concepts to 1m and lose information? This is the MASTER alignment work flagged in the canonical brief.

**(C) Expand the data window.**
P2 (Polygon 18 months) was deferred post-Phase A. With 4 weeks giving null results consistently, the question of "is 4 weeks enough sample to detect a thin edge?" becomes worth asking. Monte Carlo on 90 trades → CI for portfolio E[R] is wide; we may not have power to distinguish E[R]=0 from E[R]=+0.05.

### Recommendation

**(B) signal redesign** is the highest-EV path. (A) and (C) both bet that the current detectors are correct and we just need more sample / live data to confirm — but the consistent pattern across Phase A (5249 trades) → Phase B/C (single-signal calibration) → survivor (portfolio) is **monotonic null**. More of the same data on the same detectors won't change the answer.

Concrete (B) workstreams to consider:
- **D/4H bias gate audit**: how many trades fired against HTF bias? Do filtered subsets show edge?
- **TF mismatch audit**: how many playbooks compress 5m concepts into 1m bars? Is the signal degraded?
- **Detector preconditions sanity**: re-read the 4 EXECUTION_LAYER playbooks (now known to be sizing rejects, not bugs) — do their preconditions even reflect the MASTER patterns they're named after?

## Artifacts

- Corpus: [backend/results/labs/mini_week/survivor_v1/](../../results/labs/mini_week/survivor_v1/)
- Manifest: [survivor_v1/manifest.json](../../results/labs/mini_week/survivor_v1/manifest.json)
- Related: [c1_lsweep_verdict.md](c1_lsweep_verdict.md), [c2_bosfix_observe_verdict.md](c2_bosfix_observe_verdict.md), [b2_morningtrap_verdict.md](b2_morningtrap_verdict.md)
