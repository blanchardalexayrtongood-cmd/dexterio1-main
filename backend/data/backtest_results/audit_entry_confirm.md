# Audit 2 — entry_confirm rejection funnel

- Corpora: 24 debug_counts files (fair + survivor_v1 + Option A/B + R.3 + calib_corpus_v1)
- Min trades attempted per playbook: **10**

## Funnel table

| Playbook | Matches | SetupsCreated | AfterRisk | Attempted | Opened | EntryConfirmRejEst | EC kill% | OpenRate% | Match→Open% | SA rej% | tp_reason breakdown |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|
| Aplus_04_HTF_15m_BOS_v1 | 988 | 934 | 934 | 245 | 55 | 190.0 | **77.6** | 22.4 | 5.567 | None |  |
| Aplus_03_v2 | 173 | 96 | 96 | 72 | 22 | 50.0 | **69.4** | 30.6 | 12.717 | 43.9 | liquidity_draw_swing_k3:41, fallback_rr_min_floor_binding:29, fallback_rr_no_pool:26 |
| Aplus_03_IFVG_Flip_5m | 346 | 342 | 171 | 105 | 35 | 70.0 | **66.7** | 33.3 | 10.116 | None |  |
| BOS_Scalp_1m | 13276 | 12900 | 7548 | 548 | 135 | 0 | **0.0** | 24.6 | 1.017 | None |  |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 89148 | 81708 | 13218 | 3924 | 2157 | 0 | **0.0** | 55.0 | 2.42 | None | fixed_rr:14018 |
| EMA_Cross_5m | 212 | 206 | 84 | 39 | 11 | 0 | **0.0** | 28.2 | 5.189 | None |  |
| Engulfing_Bar_V056 | 6005 | 5686 | 3342 | 351 | 86 | 0 | **0.0** | 24.5 | 1.432 | None |  |
| FVG_Fill_Scalp | 12283 | 11263 | 4350 | 608 | 40 | 0 | **0.0** | 6.6 | 0.326 | None |  |
| FVG_Fill_V065 | 370 | 370 | 148 | 58 | 1 | 0 | **0.0** | 1.7 | 0.27 | None |  |
| FVG_Scalp_1m | 365 | 365 | 146 | 77 | 3 | 0 | **0.0** | 3.9 | 0.822 | None |  |
| HTF_Bias_15m_BOS | 3213 | 3018 | 1168 | 327 | 5 | 0 | **0.0** | 1.5 | 0.156 | None |  |
| IFVG_5m_Sweep | 9673 | 9109 | 3542 | 50 | 33 | 0 | **0.0** | 66.0 | 0.341 | None |  |
| Liquidity_Sweep_Scalp | 9622 | 7609 | 5179 | 384 | 132 | 0 | **0.0** | 34.4 | 1.372 | None |  |
| Morning_Trap_Reversal | 11635 | 9162 | 6286 | 196 | 83 | 0 | **0.0** | 42.3 | 0.713 | None |  |
| NY_Open_Reversal | 14820 | 11660 | 2332 | 106 | 17 | 0 | **0.0** | 16.0 | 0.115 | None |  |
| News_Fade | 24752 | 19407 | 8574 | 373 | 20 | 0 | **0.0** | 5.4 | 0.081 | None |  |
| ORB_Breakout_5m | 1813 | 1803 | 365 | 35 | 16 | 0 | **0.0** | 45.7 | 0.883 | None |  |
| RSI_MeanRev_5m | 740 | 714 | 280 | 39 | 29 | 0 | **0.0** | 74.4 | 3.919 | None |  |
| Range_FVG_V054 | 390 | 390 | 156 | 156 | 0 | 0 | **0.0** | 0.0 | 0.0 | None |  |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 89148 | 81708 | 13218 | 4851 | 2868 | 0 | **0.0** | 59.1 | 3.217 | None | fixed_rr:14018 |
| Session_Open_Scalp | 510 | 459 | 174 | 44 | 24 | 0 | **0.0** | 54.5 | 4.706 | None |  |
| VWAP_Bounce_5m | 50 | 47 | 18 | 11 | 6 | 0 | **0.0** | 54.5 | 12.0 | None |  |

## Lecture

- **EC kill%** = estimation proportionnelle du % de trades attempts tués par `entry_confirm_no_commit` (l'engine agrège cette stat globalement, pas par playbook ; attribution proportionnelle aux attempts).
- **Match→Open%** = efficience bout-en-bout : combien de matches du détecteur deviennent des trades réels.
- **SA rej%** = structure_alignment gate rejection (Aplus_03_v2 uniquement pour l'instant).

## Implications

Si **EC kill% > 40%** : entry_confirm filtre la moitié des setups clean. Soit le gate est trop strict, soit le signal ne confirme jamais proprement en fin de 5m.
Si **Match→Open% < 0.5%** : compression énorme entre détection et exécution — normale si le playbook est spécifique (Aplus_03_v2 : 40 matches → 8 après SA → 2 opened = 5% → 0.05% end-to-end).
