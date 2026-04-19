# Phase B0.2 — Silent Playbook Funnel Diagnosis

**Source:** debug_counts_*.json + trades_*.parquet from 4 fair audits (jun_w3 + aug_w3 + oct_w2 + nov_w4).

## Silent playbooks (0 trades across 4 weeks)

| playbook | matches_4w | setups_4w | after_risk_4w | trades_4w | level_1 | level_2 | reason |
|---|---|---|---|---|---|---|---|
| Asia_Sweep_V051 | 0 | 0 | 0 | 0 | **DETECTOR_NEVER_FIRES** | SESSION_WINDOW_MISMATCH | session=LONDON time_windows=[['02:00', '05:00']] |
| BOS_Momentum_Scalp | 0 | 0 | 0 | 0 | **DETECTOR_NEVER_FIRES** | DISABLED_OR_WRONG_MODE | enabled_in_modes=['LAB'] |
| FVG_Fill_V065 | 74 | 74 | 74 | 0 | **EXECUTION_LAYER_ISSUE** | - | - |
| FVG_Scalp_1m | 73 | 73 | 73 | 0 | **EXECUTION_LAYER_ISSUE** | - | - |
| Liquidity_Raid_V056 | 4 | 4 | 4 | 0 | **EXECUTION_LAYER_ISSUE** | - | - |
| London_Sweep_NY_Continuation | 0 | 0 | 0 | 0 | **DETECTOR_NEVER_FIRES** | DISABLED_OR_WRONG_MODE | enabled_in_modes=['LAB'] |
| Lunch_Range_Scalp | 0 | 0 | 0 | 0 | **DETECTOR_NEVER_FIRES** | DISABLED_OR_WRONG_MODE | enabled_in_modes=['DISABLED', 'LAB'] |
| OB_Retest_V004 | 0 | 0 | 0 | 0 | **DETECTOR_NEVER_FIRES** | PATTERN_PRECONDITION_BUG_OR_STRUCTURAL_RARITY | passes config checks — detector code review needed |
| Power_Hour_Expansion | 0 | 0 | 0 | 0 | **DETECTOR_NEVER_FIRES** | DISABLED_OR_WRONG_MODE | enabled_in_modes=['LAB'] |
| Range_FVG_V054 | 78 | 78 | 78 | 0 | **EXECUTION_LAYER_ISSUE** | - | - |
| Trend_Continuation_FVG_Retest | 0 | 0 | 0 | 0 | **DETECTOR_NEVER_FIRES** | DISABLED_OR_WRONG_MODE | enabled_in_modes=['LAB'] |

## Non-silent playbooks (context)

| playbook | matches_4w | setups_4w | after_risk_4w | trades_4w |
|---|---|---|---|---|
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 14858 | 13218 | 13218 | 2868 |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 14858 | 13218 | 13218 | 2157 |
| BOS_Scalp_1m | 2626 | 2516 | 2516 | 42 |
| Liquidity_Sweep_Scalp | 2427 | 1922 | 1922 | 39 |
| Engulfing_Bar_V056 | 1203 | 1114 | 1114 | 26 |
| Morning_Trap_Reversal | 2964 | 2332 | 2332 | 24 |
| NY_Open_Reversal | 2964 | 2332 | 2332 | 17 |
| ORB_Breakout_5m | 367 | 365 | 365 | 16 |
| Session_Open_Scalp | 102 | 87 | 87 | 12 |
| IFVG_5m_Sweep | 1923 | 1771 | 1771 | 11 |
| News_Fade | 5460 | 4287 | 4287 | 10 |
| RSI_MeanRev_5m | 148 | 140 | 140 | 10 |
| FVG_Fill_Scalp | 2427 | 2175 | 2175 | 6 |
| EMA_Cross_5m | 44 | 42 | 42 | 4 |
| HTF_Bias_15m_BOS | 639 | 584 | 584 | 3 |
| VWAP_Bounce_5m | 10 | 9 | 9 | 3 |
| London_Fakeout_V066 | 1 | 1 | 1 | 1 |

## Taxonomy
- **Level 1** (funnel): DETECTOR_NEVER_FIRES / SCORING_FILTERS_ALL / RISK_FILTER_KILLS_ALL / EXECUTION_LAYER_ISSUE
- **Level 2** (YAML config parse for DETECTOR_NEVER_FIRES):
  - DISABLED_OR_WRONG_MODE — trivial fix (enable in AGGRESSIVE)
  - SESSION_WINDOW_MISMATCH — config fix (time_windows don't overlap US hours)
  - HTF_BIAS_GATE_REQUIRED — config fix (bias too strict)
  - TF_CONFIG_MISMATCH — config fix (TF not loaded)
  - PATTERN_PRECONDITION_BUG_OR_STRUCTURAL_RARITY — detector code review required (Phase C.0)