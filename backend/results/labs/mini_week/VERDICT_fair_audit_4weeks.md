# Phase A.3 — Fair Audit Verdict

**Inputs:** /home/dexter/dexterio1-main/backend/results/labs/mini_week/fair_jun_w3, /home/dexter/dexterio1-main/backend/results/labs/mini_week/fair_aug_w3, /home/dexter/dexterio1-main/backend/results/labs/mini_week/fair_oct_w2, /home/dexter/dexterio1-main/backend/results/labs/mini_week/fair_nov_w4

**Portfolio:** 5249 trades, total_R=-568.10, E[R]=-0.1082

**Verdict counts:** {'QUARANTINE': 9, 'CALIBRATE': 5, 'KILL': 3}


Classification rules (Phase A.3):
  - KILL        : trades >= 15 AND E[R] < -0.1  (proven destructor)
  - CALIBRATE   : trades >= 15 AND -0.1 <= E[R] <= 0.15 AND avg(|mae_r|) > 0.3
                  (signal triggers often, losses contained, TP/SL likely miscalibrated)
  - PROMOTE     : trades >= 15 AND E[R] > 0.15  (already edge-positive)
  - QUARANTINE  : otherwise  (too few trades OR inconclusive)


## Per-playbook results

| playbook | trades | wins | losses | winrate | total_r | expectancy_r | avg_mae_r | min_mae_r | avg_peak_r | p50_duration | exit_mix | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| VWAP_Bounce_5m | 3 | 2 | 1 | 66.7 | 0.242 | 0.0806 | -0.209 | -0.387 | 1.041 | 26.0 | TP1:2, time_stop:1 | QUARANTINE |
| HTF_Bias_15m_BOS | 3 | 2 | 1 | 66.7 | 0.233 | 0.0777 | -0.445 | -1.035 | 1.51 | 1382.0 | SL:3 | QUARANTINE |
| IFVG_5m_Sweep | 11 | 5 | 6 | 45.5 | 0.363 | 0.033 | -0.774 | -1.21 | 1.011 | 1376.0 | SL:10, eod:1 | QUARANTINE |
| Session_Open_Scalp | 12 | 6 | 6 | 50.0 | 0.044 | 0.0037 | -0.596 | -1.276 | 0.621 | 65.0 | SL:7, time_stop:5 | QUARANTINE |
| Engulfing_Bar_V056 | 26 | 10 | 16 | 38.5 | 0.055 | 0.0021 | -0.451 | -1.182 | 0.643 | 101.5 | time_stop:13, SL:11, TP1:2 | CALIBRATE |
| BOS_Scalp_1m | 42 | 14 | 28 | 33.3 | -0.234 | -0.0056 | -0.469 | -1.213 | 0.36 | 120.0 | time_stop:24, SL:17, TP1:1 | CALIBRATE |
| Liquidity_Sweep_Scalp | 39 | 8 | 31 | 20.5 | -1.674 | -0.0429 | -0.347 | -1.177 | 0.438 | 30.0 | time_stop:27, SL:11, TP1:1 | CALIBRATE |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 2868 | 745 | 2123 | 26.0 | -127.488 | -0.0445 | -0.305 | -1.655 | 0.376 | 120.0 | time_stop:2452, SL:321, TP1:95 | CALIBRATE |
| Morning_Trap_Reversal | 24 | 5 | 19 | 20.8 | -1.448 | -0.0603 | -0.727 | -1.123 | 1.391 | 373.0 | SL:18, TP1:5, eod:1 | CALIBRATE |
| FVG_Fill_Scalp | 6 | 0 | 6 | 0.0 | -0.367 | -0.0612 | -0.394 | -0.868 | 0.384 | 120.0 | time_stop:5, SL:1 | QUARANTINE |
| News_Fade | 10 | 4 | 6 | 40.0 | -0.649 | -0.0649 | -0.697 | -1.442 | 0.608 | 90.0 | session_end:7, SL:3 | QUARANTINE |
| RSI_MeanRev_5m | 10 | 1 | 9 | 10.0 | -1.012 | -0.1012 | -0.48 | -1.261 | 0.271 | 120.0 | time_stop:8, SL:2 | QUARANTINE |
| ORB_Breakout_5m | 16 | 4 | 12 | 25.0 | -1.64 | -0.1025 | -0.474 | -1.415 | 0.379 | 120.0 | time_stop:13, SL:3 | KILL |
| EMA_Cross_5m | 4 | 1 | 3 | 25.0 | -0.497 | -0.1243 | -0.531 | -0.931 | 0.526 | 120.0 | time_stop:3, SL:1 | QUARANTINE |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 2157 | 367 | 1790 | 17.0 | -429.935 | -0.1993 | -0.786 | -1.69 | 0.878 | 1604.0 | SL:1668, eod:338, TP1:151 | KILL |
| NY_Open_Reversal | 17 | 2 | 15 | 11.8 | -3.58 | -0.2106 | -0.762 | -1.233 | 0.84 | 1306.0 | SL:12, session_end:3, eod:1 | KILL |
| London_Fakeout_V066 | 1 | 0 | 1 | 0.0 | -0.511 | -0.5111 | -1.48 | -1.48 | 0.654 | 1.0 | SL:1 | QUARANTINE |



## CALIBRATE (5 playbooks)

| playbook | trades | wins | losses | winrate | total_r | expectancy_r | avg_mae_r | min_mae_r | avg_peak_r | p50_duration | exit_mix | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Engulfing_Bar_V056 | 26 | 10 | 16 | 38.5 | 0.055 | 0.0021 | -0.451 | -1.182 | 0.643 | 101.5 | time_stop:13, SL:11, TP1:2 | CALIBRATE |
| BOS_Scalp_1m | 42 | 14 | 28 | 33.3 | -0.234 | -0.0056 | -0.469 | -1.213 | 0.36 | 120.0 | time_stop:24, SL:17, TP1:1 | CALIBRATE |
| Liquidity_Sweep_Scalp | 39 | 8 | 31 | 20.5 | -1.674 | -0.0429 | -0.347 | -1.177 | 0.438 | 30.0 | time_stop:27, SL:11, TP1:1 | CALIBRATE |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 2868 | 745 | 2123 | 26.0 | -127.488 | -0.0445 | -0.305 | -1.655 | 0.376 | 120.0 | time_stop:2452, SL:321, TP1:95 | CALIBRATE |
| Morning_Trap_Reversal | 24 | 5 | 19 | 20.8 | -1.448 | -0.0603 | -0.727 | -1.123 | 1.391 | 373.0 | SL:18, TP1:5, eod:1 | CALIBRATE |



## KILL (3 playbooks)

| playbook | trades | wins | losses | winrate | total_r | expectancy_r | avg_mae_r | min_mae_r | avg_peak_r | p50_duration | exit_mix | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ORB_Breakout_5m | 16 | 4 | 12 | 25.0 | -1.64 | -0.1025 | -0.474 | -1.415 | 0.379 | 120.0 | time_stop:13, SL:3 | KILL |
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 2157 | 367 | 1790 | 17.0 | -429.935 | -0.1993 | -0.786 | -1.69 | 0.878 | 1604.0 | SL:1668, eod:338, TP1:151 | KILL |
| NY_Open_Reversal | 17 | 2 | 15 | 11.8 | -3.58 | -0.2106 | -0.762 | -1.233 | 0.84 | 1306.0 | SL:12, session_end:3, eod:1 | KILL |



## QUARANTINE (9 playbooks)

| playbook | trades | wins | losses | winrate | total_r | expectancy_r | avg_mae_r | min_mae_r | avg_peak_r | p50_duration | exit_mix | verdict |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| VWAP_Bounce_5m | 3 | 2 | 1 | 66.7 | 0.242 | 0.0806 | -0.209 | -0.387 | 1.041 | 26.0 | TP1:2, time_stop:1 | QUARANTINE |
| HTF_Bias_15m_BOS | 3 | 2 | 1 | 66.7 | 0.233 | 0.0777 | -0.445 | -1.035 | 1.51 | 1382.0 | SL:3 | QUARANTINE |
| IFVG_5m_Sweep | 11 | 5 | 6 | 45.5 | 0.363 | 0.033 | -0.774 | -1.21 | 1.011 | 1376.0 | SL:10, eod:1 | QUARANTINE |
| Session_Open_Scalp | 12 | 6 | 6 | 50.0 | 0.044 | 0.0037 | -0.596 | -1.276 | 0.621 | 65.0 | SL:7, time_stop:5 | QUARANTINE |
| FVG_Fill_Scalp | 6 | 0 | 6 | 0.0 | -0.367 | -0.0612 | -0.394 | -0.868 | 0.384 | 120.0 | time_stop:5, SL:1 | QUARANTINE |
| News_Fade | 10 | 4 | 6 | 40.0 | -0.649 | -0.0649 | -0.697 | -1.442 | 0.608 | 90.0 | session_end:7, SL:3 | QUARANTINE |
| RSI_MeanRev_5m | 10 | 1 | 9 | 10.0 | -1.012 | -0.1012 | -0.48 | -1.261 | 0.271 | 120.0 | time_stop:8, SL:2 | QUARANTINE |
| EMA_Cross_5m | 4 | 1 | 3 | 25.0 | -0.497 | -0.1243 | -0.531 | -0.931 | 0.526 | 120.0 | time_stop:3, SL:1 | QUARANTINE |
| London_Fakeout_V066 | 1 | 0 | 1 | 0.0 | -0.511 | -0.5111 | -1.48 | -1.48 | 0.654 | 1.0 | SL:1 | QUARANTINE |

