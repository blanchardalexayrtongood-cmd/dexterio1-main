# TF compression / MASTER faithfulness audit (Phase D.2, 2026-04-20)

## TL;DR

Read-only audit of the YAML configs of all "MASTER faithful" playbooks and the legacy 1m playbooks, cross-referenced against [MASTER_PLAYBOOK_MAP.md](../../../MASTER_PLAYBOOK_MAP.md).

**Headline finding**: the conceptual MASTER framework (D/4H bias → 15m/5m setup → 1m execution) is **barely implemented in the active engine.** Of 6 MASTER families:
- 0/6 fully implemented per spec
- 3/6 entirely missing or `research_only` (Family A Sweep+IFVG, Family B HTF+15m BOS, Family F Premarket Sweep 5m)
- 1/6 implemented but blocked (Family D ORB → Session_Open_Scalp LAB ONLY)
- 2/6 implemented partially (Family C FVG Fill, Family E News Fade)

The 7 "MASTER faithful" playbooks (V065/V056/V054/V051/V066/V004 + Engulfing) we've been calibrating in Phase B/C are **NOT direct instantiations of the MASTER families**. They use MASTER vocabulary (FVG, BOS, OB, sweep) at the YAML level but skip key parts of the MASTER pipeline:

1. **No D/4H bias check** in any of the 7. They use intraday `htf_bias` from market_state.py (pivot-based on 1H/4H intraday windows), not D/4H draw-on-liquidity.
2. **No 15m setup TF anywhere.** All 7 use `setup_tf: 5m`. MASTER families A/B explicitly require 15m setup; Family C accepts 5m but with HTF context.
3. **HTF bias gate inconsistently applied**: 4/7 have `htf_bias_allowed: [bullish, bearish, neutral]` (gate effectively OFF); 3/7 restrict to directional only (gate ON).
4. **Fixed-RR TPs everywhere**, no liquidity-targeting TPs (MASTER convention is TP at next draw on liquidity, not 2.0R).
5. **5m confirmation, no 1m confirmation step.** Per MASTER, you wait for 1m confirmation in the entry zone after 5m setup. None of the 7 do this — entries are MARKET or LIMIT off the 5m close.

**Important corollary**: the bear case "no edge in current detectors" is partially overstated. We've been calibrating playbooks that don't follow the MASTER pipeline. The playbooks the MASTER actually describes (Aplus_01/03/04 from transcripts) are `research_only` and **have never been tested**.

## Per-playbook YAML extraction

### 7 MASTER faithful

| Playbook | category | setup_tf | required_signals | bias gate | structure gate | TP1 | min_rr | description claim |
|---|---|---|---|---|---|---|---|---|
| FVG_Fill_V065 | SCALP | 5m | FVG@5m | OFF (all) | OFF (all) | 2.0R | 2.0 | "V065 faithful: 15m range → 5m FVG break → limit 50% → TP 2:1" |
| Liquidity_Raid_V056 | SCALP | 5m | SWEEP@5m | OFF (all) | OFF (all) | 2.0R | 2.0 | "V056 M2 faithful: equal H/L sweep → close confirmation → MARKET" |
| Range_FVG_V054 | SCALP | 5m | FVG@5m + engulfing | OFF (all) | OFF (all) | 3.0R | 3.0 | "V054 faithful: 5m range → FVG + engulfing → limit 50% → TP 3:1" |
| Asia_Sweep_V051 | DAYTRADE | 5m | SWEEP@5m + BOS@5m | **ON (bull/bear)** | **ON (up/down)** | 5.0R | 5.0 | "V051 faithful: daily bias + Asia sweep + BOS → London reversal → TP 5:1" |
| London_Fakeout_V066 | DAYTRADE | 5m | SWEEP@5m + BOS@5m | OFF (all) | OFF (all) | 3.0R | 3.0 | "V066 faithful: London fakeout → 5m BOS reversal → FVG entry → TP 3:1" |
| Engulfing_Bar_V056 | SCALP | 5m | engulfing | **ON (bull/bear)** | **ON (up/down)** | 2.0R | 2.0 | "V056 M1 faithful: engulfing bar on 5m → MARKET → TP 2:1" |
| OB_Retest_V004 | DAYTRADE | 5m | OB@5m + BOS@5m | **ON (bull/bear)** | **ON (up/down)** | 3.0R | 3.0 | "V004 faithful: OB before BOS → retest → MARKET → TP 3:1" |

### 1m playbooks (TF compression check)

| Playbook | setup_tf | confirmation_tf | required_signals | bias gate | TP1 | description |
|---|---|---|---|---|---|---|
| **FVG_Scalp_1m** | **1m** | **1m** | **FVG@1m** | OFF (all) | 1.5R | "1m FVG scalp: FVG on 1m → limit 50% → TP 1.5:1" |
| **BOS_Scalp_1m** | **1m** | **1m** | **BOS@1m** | OFF (all) | 1.5R | "1m BOS scalp: BOS + engulfing on 1m → MARKET → TP 1.5:1" |

Both are **TF-compressed** (BOS/FVG concepts compressed to 1m; both lack HTF gates). Both YAMLs are honest — the names contain "1m" so there's no faithfulness lie.

### Reference: existing FVG_Fill_Scalp (Family C MASTER claim)

`FVG_Fill_Scalp` actually has `setup_tf: 5m` and `confirmation_tf: 1m` in the current YAML. **MASTER_PLAYBOOK_MAP.md line 82 is stale** — it claims "branché mais en 1m, pas en 5m", but the current YAML is 5m setup + 1m confirm, which IS Family C–compliant. (Recommend updating MAP doc.)

## Faithfulness gaps vs MASTER framework

| MASTER requirement | Implementation status (across 7 faithful) |
|---|---|
| D/4H bias drives setup direction | **0/7** — none use D/4H bias; some use intraday `htf_bias` (pivot 1H/4H) |
| 15m or 5m setup TF | 7/7 use 5m (acceptable for Families C/D/E) |
| Setup must align with HTF bias | **3/7** enforce (Asia_Sweep, Engulfing, OB_Retest); 4/7 don't |
| Liquidity sweep precedes entry | 3/7 require sweep (Asia, London, Liq_Raid); 4/7 don't |
| TP at next liquidity draw | **0/7** — all use fixed RR (2.0–5.0R) |
| 1m confirmation in entry zone | **0/7** — all use 5m confirmation |
| Premarket / Asia session bias context | 1/7 (Asia_Sweep_V051 only) |

**The most consistent gap: TP placement.** Every "MASTER faithful" YAML uses fixed RR (2.0R/3.0R/5.0R) for TP1. MASTER doctrine targets the **next available liquidity** (session high, prior day high, equal highs). Per Phase B1 calibration findings, peak_R p60 was often 0.4–0.6R — meaning fixed 1.5R+ TPs are unreachable for typical setups. The MASTER would adapt TP to the nearest liquidity, often closer than 1.5R.

## Cross-reference with prior phase results

| Playbook | bias gate | survivor result | Phase B/C result |
|---|---|---|---|
| **Engulfing_Bar_V056** (gate ON) | n=26, E[R]=-0.010 | n=34 in B1 corpus, E[R]=-0.10 | **Best survivor with bias gate ON** |
| **Asia_Sweep_V051** (gate ON) | silent (B0.2: SESSION_WINDOW_MISMATCH) | — | Bias gate ON helps but session_window kills entry |
| **OB_Retest_V004** (gate ON) | silent (B0.2: PATTERN_PRECONDITION_BUG) | — | Bias gate ON but detector preconditions never met |
| **FVG_Fill_V065** (gate OFF) | silent (Phase C.0: position-size reject) | — | Cannot test bias-gate effect from data |
| **Liquidity_Raid_V056** (gate OFF) | silent (C.0: position-size reject) | — | — |
| **Range_FVG_V054** (gate OFF) | silent (C.0: position-size reject) | — | — |
| **London_Fakeout_V066** (gate OFF) | n=1, E[R]=-0.477 | — | Single-trade noise |

**Pattern**: the 3 playbooks with bias gate ON either survive (Engulfing) or are silenced for non-bias reasons (Asia_Sweep, OB_Retest). The 4 with bias gate OFF either get sized to zero (Phase C.0 artifact) or fire only marginally. **We can't conclude from this data alone that turning the bias gate ON would help the gate-OFF group**, but the survivor data hints at it.

## What this means for the "no edge" verdict

The 5-phase monotonic null pattern (fair audit → 3 calibrations → portfolio → bias gate) was on **the playbooks that exist**, not on **the MASTER framework as specified**. Three observations:

1. **The MASTER families A (Sweep+IFVG), B (HTF+15m BOS), F (Premarket Sweep 5m) are entirely untested.** The transcripts (Aplus_01/03/04) describe these but the YAMLs are `research_only`. Code primitives exist (ifvg.py, order_block.py, breaker_block.py) but no playbook chains them per spec.

2. **Family C (FVG Fill 5m)** is implemented in `FVG_Fill_Scalp` (5m setup + 1m confirm) and reasonably MASTER-aligned. Survivor result: n=34, E[R]=-0.076. So at least one MASTER-aligned playbook has been tested and it's negative.

3. **Family E (News Fade)** has been tested extensively; gate REOPEN_1R_VS_1P5R closed UNRESOLVED. News_Fade survives at n=10, E[R]=+0.001 in survivor.

**Conclusion: the bear case is well-supported for Family C and E (tested, negative).** The bear case is **NOT supported** for Families A, B, F (untested).

## Decision

### What changes
- Nothing in the engine, YAML, or active code paths.
- **Recommendation to update [MASTER_PLAYBOOK_MAP.md line 82](../../../MASTER_PLAYBOOK_MAP.md#L82)**: FVG_Fill_Scalp is now `setup_tf: 5m`, not 1m as the doc claims. Doc is stale.

### Three honest paths forward (decision needed from user)

**(A) Instantiate one of the 3 untested MASTER families and test it.**
Most-actionable candidate: **Aplus_03 — IFVG Flip from FVG Invalidation**. Per MAP line 170-172: "Plus simple : juste la logique d'invalidation d'un FVG existant. Potentiellement implémentable comme variante de FVG_Fill_Scalp avec confirmation_tf: 5m". Code primitives present (ifvg.py). Estimated 2-4 hours to draft YAML + playbook test on calib_corpus_v1 conditions.

**(B) Restructure the 4 "gate OFF" MASTER faithful YAMLs to enable the HTF bias gate, then re-test.**
Risk: this is a per-playbook calibration which Phase B/C says asymptotes. But Phase D.2 specifically suggests that the gate-OFF YAMLs are MASTER-unfaithful in a fixable way. The 3 gate-ON playbooks have suggestive results (Engulfing survives best); re-testing the other 4 with gate ON would either confirm or refute the hypothesis. Cheap (YAML edits only) — gates change, no code change.

**(C) Accept that the MASTER families that have been tested are negative + the untested ones aren't going to be tested.** Pivot to (A from earlier): paper-trade the survivor 3-pack as instrumented baseline.

### Recommendation

**(A) Instantiate Aplus_03 (IFVG Flip).** This is the highest-information move because:
1. It's an **untested MASTER family** — we don't yet know if MASTER-faithful detectors can produce edge on SPY/QQQ 4-week intraday data.
2. It's **scoped narrowly** — one playbook, code primitives exist, est. 2-4h to draft + test on the existing `calib_corpus_v1` infrastructure.
3. It generates a **clean negative or positive signal**: if Aplus_03 also asymptotes negative, we have stronger evidence for "MASTER doesn't transfer to 4-week SPY/QQQ intraday" and (C) becomes well-justified. If it produces edge, we've found a path that's been ignored for months.

(B) is incremental and likely null per Phase B/C asymptote. (C) is premature without testing the untested families.

## Artifacts

- Source: [MASTER_PLAYBOOK_MAP.md](../../../MASTER_PLAYBOOK_MAP.md), [DEXTERIO_CANONICAL_BRIEF.md](../../../DEXTERIO_CANONICAL_BRIEF.md)
- YAMLs reviewed: [playbooks.yml](../../knowledge/playbooks.yml) lines 477 (FVG_Fill_Scalp), 898–1394 (7 MASTER faithful), 1395–1539 (1m compressed)
- Related verdicts: [bias_audit_v1_verdict.md](bias_audit_v1_verdict.md), [survivor_v1_verdict.md](survivor_v1_verdict.md)
- Aplus transcripts: `backend/knowledge/playbooks_Aplus_from_transcripts.yaml` (research_only — would be the basis for path A)
