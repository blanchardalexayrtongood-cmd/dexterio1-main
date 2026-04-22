# Audit 4 — filter splits

- Total trades: 5610
- Playbooks analyzed: 10
- Min bucket n for edge candidate: 15

## Edge candidates (avg_R > 0, n ≥ 15)

| Playbook | Dimension | Bucket | n | avg_R | WR% |
|---|---|---|---:|---:|---:|
| Engulfing_Bar_V056 | direction | SHORT | 18 | +0.0630 | 55.6 |
| Aplus_04_HTF_15m_BOS_v1 | mc_breakout_dir | NONE | 18 | +0.0485 | 61.1 |
| Engulfing_Bar_V056 | day_of_week | Monday | 16 | +0.0448 | 50.0 |
| Engulfing_Bar_V056 | symbol | QQQ | 32 | +0.0441 | 50.0 |
| BOS_Scalp_1m | direction | SHORT | 26 | +0.0399 | 57.7 |
| BOS_Scalp_1m | day_of_week | Monday | 24 | +0.0290 | 45.8 |
| Aplus_04_HTF_15m_BOS_v1 | sl_dist_quintile | 1 | 23 | +0.0238 | 69.6 |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | mc_breakout_dir | nan | 95 | +0.0182 | 54.7 |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | mc_breakout_dir | NONE | 151 | +0.0141 | 68.2 |
| Liquidity_Sweep_Scalp | sl_dist_quintile | 1 | 22 | +0.0074 | 40.9 |
| Aplus_03_IFVG_Flip_5m | direction | SHORT | 23 | +0.0038 | 60.9 |
| Liquidity_Sweep_Scalp | day_of_week | Wednesday | 15 | +0.0029 | 40.0 |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | session_bucket | first_30m | 183 | +0.0024 | 56.8 |

## Global splits (all playbooks merged)

### session_bucket

| Bucket | n | avg_R | WR% | Edge? |
|---|---:|---:|---:|---|
| first_30m | 468 | -0.0449 | 42.1 |  |
| afternoon_210_330 | 1696 | -0.0632 | 27.0 |  |
| last_30m | 770 | -0.0682 | 10.1 |  |
| morning_30_120 | 1176 | -0.1254 | 27.4 |  |
| midday_120_210 | 1498 | -0.1779 | 16.8 |  |

### session_label

| Bucket | n | avg_R | WR% | Edge? |
|---|---:|---:|---:|---|
| ny | 5608 | -0.1060 | 23.3 |  |

### killzone_label

| Bucket | n | avg_R | WR% | Edge? |
|---|---:|---:|---:|---|
| ny_pm | 823 | -0.0917 | 19.0 |  |
| none | 3425 | -0.1052 | 20.2 |  |
| ny_open | 1362 | -0.1174 | 33.6 |  |

### day_of_week

| Bucket | n | avg_R | WR% | Edge? |
|---|---:|---:|---:|---|
| Thursday | 475 | -0.0306 | 14.9 |  |
| Friday | 96 | -0.0511 | 29.2 |  |
| Wednesday | 1262 | -0.0614 | 29.2 |  |
| Tuesday | 1800 | -0.0813 | 22.7 |  |
| Monday | 1977 | -0.1783 | 21.8 |  |

### symbol

| Bucket | n | avg_R | WR% | Edge? |
|---|---:|---:|---:|---|
| QQQ | 2843 | -0.0922 | 22.6 |  |
| SPY | 2767 | -0.1206 | 24.0 |  |

### direction

| Bucket | n | avg_R | WR% | Edge? |
|---|---:|---:|---:|---|
| SHORT | 1737 | -0.0759 | 26.3 |  |
| LONG | 3873 | -0.1198 | 21.9 |  |

### mc_breakout_dir

| Bucket | n | avg_R | WR% | Edge? |
|---|---:|---:|---:|---|
| nan | 285 | -0.0336 | 39.3 |  |
| NONE | 296 | -0.0544 | 47.6 |  |
| LONG | 2636 | -0.0974 | 20.8 |  |
| SHORT | 2393 | -0.1309 | 21.1 |  |

### sl_dist_quintile

| Bucket | n | avg_R | WR% | Edge? |
|---|---:|---:|---:|---|
| 1 | 1122 | -0.0220 | 28.1 |  |
| 0 | 1122 | -0.0446 | 20.6 |  |
| 4 | 1122 | -0.1296 | 24.3 |  |
| 3 | 1122 | -0.1321 | 23.7 |  |
| 2 | 1122 | -0.2026 | 19.7 |  |

## Lecture

- **Edge candidate** = bucket où avg_R > 0 avec n ≥ 15. Signale une dimension qui isolerait potentiellement un subset trad.
- Plus il y a de candidats sur la même dimension pour différents playbooks → dimension robuste.
- **Attention** : les candidats découverts ici sont **in-sample** sur ce corpus 4-semaines. Appliquer tel quel sans test holdout = risque overfitting.
- **Caveat MASTER** : les playbooks Aplus_XX ont déjà du require_close_above_trigger + entry_buffer_bps + structure_alignment. Ajouter un filtre de plus augmente les contraintes, réduit n, pas toujours souhaitable.
