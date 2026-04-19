# Phase A.3 — Fair Audit Verdict

**Inputs:** /home/dexter/dexterio1-main/backend/results/labs/mini_week/fair_oct_w2

**Portfolio:** 1877 trades, total_R=-222.63, E[R]=-0.1186

**Verdict counts:** {'QUARANTINE': 12, 'CALIBRATE': 2, 'KILL': 1}


Classification rules (Phase A.3):
  - KILL        : trades >= 15 AND E[R] < -0.1  (proven destructor)
  - CALIBRATE   : trades >= 15 AND -0.1 <= E[R] <= 0.15 AND avg(|mae_r|) > 0.3
                  (signal triggers often, losses contained, TP/SL likely miscalibrated)
  - PROMOTE     : trades >= 15 AND E[R] > 0.15  (already edge-positive)
  - QUARANTINE  : otherwise  (too few trades OR inconclusive)


## Per-playbook results

| playbook | trades | wins | losses | winrate | total_r | expectancy_r | avg_mae_r | min_mae_r | avg_peak_r | p50_duration | exit_mix | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| HTF_Bias_15m_BOS | 1 | 1 | 0 | 100.0 | 0.03 | 0.0304 | -0.156 | -0.156 | 1.711 | 2632.0 | SL:1 | QUARANTINE |
| VWAP_Bounce_5m | 1 | 1 | 0 | 100.0 | 0.014 | 0.0136 | -0.231 | -0.231 | 1.519 | 26.0 | TP1:1 | QUARANTINE |
| ORB_Breakout_5m | 4 | 3 | 1 | 75.0 | 0.019 | 0.0048 | -0.448 | -1.415 | 0.716 | 82.5 | time_stop:2, SL:2 | QUARANTINE |
| News_Fade | 5 | 3 | 2 | 60.0 | 0.01 | 0.002 | -0.692 | -1.211 | 0.738 | 90.0 | session_end:3, SL:2 | QUARANTINE |
| FVG_Fill_Scalp | 1 | 0 | 1 | 0.0 | -0.008 | -0.0077 | -0.239 | -0.239 | 0.243 | 120.0 | time_stop:1 | QUARANTINE |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 927 | 255 | 672 | 27.5 | -11.378 | -0.0123 | -0.282 | -1.655 | 0.285 | 120.0 | time_stop:796, SL:128, TP1:3 | QUARANTINE |
| Engulfing_Bar_V056 | 10 | 2 | 8 | 20.0 | -0.215 | -0.0215 | -0.594 | -1.097 | 0.58 | 101.5 | time_stop:5, SL:5 | QUARANTINE |
| IFVG_5m_Sweep | 6 | 2 | 4 | 33.3 | -0.131 | -0.0218 | -0.98 | -1.117 | 0.865 | 1179.5 | SL:6 | QUARANTINE |
| Session_Open_Scalp | 5 | 2 | 3 | 40.0 | -0.117 | -0.0233 | -0.479 | -1.039 | 0.38 | 120.0 | time_stop:4, SL:1 | QUARANTINE |
| Liquidity_Sweep_Scalp | 15 | 0 | 15 | 0.0 | -0.615 | -0.041 | -0.412 | -1.0 | 0.305 | 30.0 | time_stop:13, SL:2 | CALIBRATE |
| RSI_MeanRev_5m | 8 | 1 | 7 | 12.5 | -0.414 | -0.0518 | -0.501 | -1.261 | 0.293 | 120.0 | time_stop:6, SL:2 | QUARANTINE |
| BOS_Scalp_1m | 15 | 3 | 12 | 20.0 | -1.035 | -0.069 | -0.667 | -1.213 | 0.259 | 120.0 | time_stop:9, SL:6 | CALIBRATE |
| NY_Open_Reversal | 5 | 0 | 5 | 0.0 | -0.426 | -0.0853 | -0.903 | -1.029 | 0.453 | 337.0 | SL:4, session_end:1 | QUARANTINE |
| Morning_Trap_Reversal | 8 | 1 | 7 | 12.5 | -0.938 | -0.1173 | -0.736 | -1.029 | 0.922 | 791.5 | SL:7, TP1:1 | QUARANTINE |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 866 | 38 | 828 | 4.4 | -207.425 | -0.2395 | -0.91 | -1.57 | 0.626 | 1634.0 | SL:828, TP1:38 | KILL |



## CALIBRATE (2 playbooks)

| playbook | trades | wins | losses | winrate | total_r | expectancy_r | avg_mae_r | min_mae_r | avg_peak_r | p50_duration | exit_mix | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Liquidity_Sweep_Scalp | 15 | 0 | 15 | 0.0 | -0.615 | -0.041 | -0.412 | -1.0 | 0.305 | 30.0 | time_stop:13, SL:2 | CALIBRATE |
| BOS_Scalp_1m | 15 | 3 | 12 | 20.0 | -1.035 | -0.069 | -0.667 | -1.213 | 0.259 | 120.0 | time_stop:9, SL:6 | CALIBRATE |



## KILL (1 playbooks)

| playbook | trades | wins | losses | winrate | total_r | expectancy_r | avg_mae_r | min_mae_r | avg_peak_r | p50_duration | exit_mix | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 866 | 38 | 828 | 4.4 | -207.425 | -0.2395 | -0.91 | -1.57 | 0.626 | 1634.0 | SL:828, TP1:38 | KILL |



## QUARANTINE (12 playbooks)

| playbook | trades | wins | losses | winrate | total_r | expectancy_r | avg_mae_r | min_mae_r | avg_peak_r | p50_duration | exit_mix | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| HTF_Bias_15m_BOS | 1 | 1 | 0 | 100.0 | 0.03 | 0.0304 | -0.156 | -0.156 | 1.711 | 2632.0 | SL:1 | QUARANTINE |
| VWAP_Bounce_5m | 1 | 1 | 0 | 100.0 | 0.014 | 0.0136 | -0.231 | -0.231 | 1.519 | 26.0 | TP1:1 | QUARANTINE |
| ORB_Breakout_5m | 4 | 3 | 1 | 75.0 | 0.019 | 0.0048 | -0.448 | -1.415 | 0.716 | 82.5 | time_stop:2, SL:2 | QUARANTINE |
| News_Fade | 5 | 3 | 2 | 60.0 | 0.01 | 0.002 | -0.692 | -1.211 | 0.738 | 90.0 | session_end:3, SL:2 | QUARANTINE |
| FVG_Fill_Scalp | 1 | 0 | 1 | 0.0 | -0.008 | -0.0077 | -0.239 | -0.239 | 0.243 | 120.0 | time_stop:1 | QUARANTINE |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 927 | 255 | 672 | 27.5 | -11.378 | -0.0123 | -0.282 | -1.655 | 0.285 | 120.0 | time_stop:796, SL:128, TP1:3 | QUARANTINE |
| Engulfing_Bar_V056 | 10 | 2 | 8 | 20.0 | -0.215 | -0.0215 | -0.594 | -1.097 | 0.58 | 101.5 | time_stop:5, SL:5 | QUARANTINE |
| IFVG_5m_Sweep | 6 | 2 | 4 | 33.3 | -0.131 | -0.0218 | -0.98 | -1.117 | 0.865 | 1179.5 | SL:6 | QUARANTINE |
| Session_Open_Scalp | 5 | 2 | 3 | 40.0 | -0.117 | -0.0233 | -0.479 | -1.039 | 0.38 | 120.0 | time_stop:4, SL:1 | QUARANTINE |
| RSI_MeanRev_5m | 8 | 1 | 7 | 12.5 | -0.414 | -0.0518 | -0.501 | -1.261 | 0.293 | 120.0 | time_stop:6, SL:2 | QUARANTINE |
| NY_Open_Reversal | 5 | 0 | 5 | 0.0 | -0.426 | -0.0853 | -0.903 | -1.029 | 0.453 | 337.0 | SL:4, session_end:1 | QUARANTINE |
| Morning_Trap_Reversal | 8 | 1 | 7 | 12.5 | -0.938 | -0.1173 | -0.736 | -1.029 | 0.922 | 791.5 | SL:7, TP1:1 | QUARANTINE |

