# Audit 1 — peak_R vs TP per-playbook

- Min n per playbook: **15**
- Playbooks analyzed: **14**
- Verdict distribution: {'GEOMETRY_CONDEMNED': 10, 'GEOMETRY_TIGHT': 4}

## Table

| Playbook | n | WR% | E[R] | peak_R p50/p60/p80 | mae_R p20/p50 | TP_RR | ratio p80/TP | Verdict | SL%/TP1%/TS% | Corpus |
|---|---:|---:|---:|---:|---:|---:|---:|---|---:|---|
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 2868 | 26.0 | -0.044 | 0.164/0.253/0.578 | -0.546/-0.15 | 2.0 | 0.289 | **GEOMETRY_CONDEMNED** | 11.2/3.3/0 | fair |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 2157 | 17.0 | -0.199 | 0.456/0.752/1.615 | -1.037/-1.002 | 3.0 | 0.538 | **GEOMETRY_CONDEMNED** | 77.3/7.0/0 | fair |
| BOS_Scalp_1m | 42 | 35.7 | -0.062 | 0.239/0.406/0.862 | -1.021/-0.395 | 1.5 | 0.575 | **GEOMETRY_CONDEMNED** | 45.2/7.1/0 | survivor_v1 |
| Liquidity_Sweep_Scalp | 42 | 33.3 | -0.036 | 0.276/0.386/0.905 | -0.521/-0.235 | 1.5 | 0.603 | **GEOMETRY_CONDEMNED** | 21.4/4.8/0 | survivor_v1 |
| FVG_Fill_Scalp | 34 | 17.6 | -0.076 | 0.3/0.436/0.856 | -0.705/-0.258 | 1.5 | 0.571 | **GEOMETRY_CONDEMNED** | 38.2/0/0 | survivor_v1 |
| Engulfing_Bar_V056 | 26 | 38.5 | -0.010 | 0.276/0.663/1.07 | -0.814/-0.278 | 2.0 | 0.535 | **GEOMETRY_CONDEMNED** | 34.6/7.7/0 | survivor_v1 |
| IFVG_5m_Sweep | 22 | 45.5 | -0.172 | 0.621/0.913/1.332 | -1.018/-0.904 | 3.0 | 0.444 | **GEOMETRY_CONDEMNED** | 81.8/0/0 | survivor_v1 |
| RSI_MeanRev_5m | 19 | 21.1 | -0.267 | 0.226/0.258/0.633 | -0.89/-0.374 | 1.5 | 0.422 | **GEOMETRY_CONDEMNED** | 15.8/5.3/0 | survivor_v1 |
| NY_Open_Reversal | 17 | 11.8 | -0.211 | 0.57/0.636/1.214 | -1.03/-1.0 | 3.0 | 0.405 | **GEOMETRY_CONDEMNED** | 70.6/5.9/0 | fair |
| ORB_Breakout_5m | 16 | 25.0 | -0.102 | 0.268/0.373/0.47 | -0.701/-0.49 | 2.0 | 0.235 | **GEOMETRY_CONDEMNED** | 18.8/0/0 | fair |
| Aplus_04_HTF_15m_BOS_v1 | 55 | 47.3 | -0.073 | 0.707/0.772/1.015 | -1.04/-0.521 | 1.0 | 1.015 | **GEOMETRY_TIGHT** | 74.5/23.6/0 | b_aplus04_v1 |
| Aplus_03_IFVG_Flip_5m | 35 | 42.9 | -0.055 | 0.542/0.638/0.734 | -0.798/-0.331 | 0.7 | 1.048 | **GEOMETRY_TIGHT** | 48.6/28.6/0 | r3_aplus03 |
| Morning_Trap_Reversal | 25 | 16.0 | -0.175 | 0.448/0.827/2.432 | -1.054/-1.01 | 3.0 | 0.811 | **GEOMETRY_TIGHT** | 72.0/8.0/0 | survivor_v1 |
| Aplus_03_v2 | 22 | 45.5 | -0.019 | 0.522/0.584/0.669 | -0.533/-0.216 | 0.698 | 0.958 | **GEOMETRY_TIGHT** | 50.0/27.3/0 | aplus03_v2 |

## Lecture

- **GEOMETRY_CONDEMNED** (ratio p80/TP < 0.80) : TP fixe > peak_R p80, le marché offre rarement assez de MFE pour atteindre le TP. Pathologie Aplus_03 R.3 / Aplus_04 Option B.
- **GEOMETRY_TIGHT** (ratio 0.80–1.10) : TP atteignable mais rare. Winners plafonnent juste au-dessus.
- **GEOMETRY_OK** (ratio > 1.10) : TP réaliste vs MFE observée.

## Implication

Pour tout playbook **GEOMETRY_CONDEMNED** :
- fixed RR est structurellement mauvais → refaire le TP via `tp_logic: liquidity_draw` (Option A v2), OU abaisser TP_RR vers peak_R p60, OU KILL.
- calibration incrémentale (bouger BE, trailing, max_duration) ne résoudra PAS le problème — on parle de géométrie TP, pas d'exit logic.
