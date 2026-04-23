# Paper-vs-backtest reconcile — trades_miniweek_calib_corpus_v1_aug_w3_AGGRESSIVE_DAILY_SCALP.parquet

**Slippage model:** ConservativeFillModel, next-bar-open fills with 0.050% extra slippage + 1.00 bps spread (§0.7 G3, asymmetric buyer/seller).

**Interpretation:** negative delta_$ = conservative model would have been worse than the backtest's ideal fill.

## Aggregate

| metric | value |
|---|---|
| n_trades | 50 |
| total_delta_$ | -5495.76 |
| mean_delta_$ | -109.92 |
| p50_delta_$ | -95.29 |
| p95_delta_$ | -31.73 |
| min_delta_$ | -559.13 |
| max_delta_$ | 112.24 |
| total_delta_R | -5.157 |
| mean_delta_R | -0.1031 |
| pct_trades_worse | 98.0 |

## First 20 trades

| trade_id | symbol | direction | playbook | exit_reason | ideal_entry | conservative_entry | ideal_exit | conservative_exit | delta_$ | delta_R |
|---|---|---|---|---|---|---|---|---|---|---|
| 082fd649-a06a-42bd-9420-caddf22b43fd | QQQ | LONG | Morning_Trap_Reversal | SL | 577.555 | 577.9466 | 576.312445 | 575.7044 | -85.97 | -0.043 |
| daf23902-ebe9-4a80-aab8-a4246ffb1a08 | QQQ | LONG | BOS_Scalp_1m | SL | 577.35 | 577.7264 | 576.1953 | 575.7044 | -111.89 | -0.0559 |
| 7d454001-0789-4033-8758-d591b442421a | QQQ | LONG | Engulfing_Bar_V056 | SL | 577.335 | 577.6464 | 576.18033 | 575.7044 | -136.21 | -0.0681 |
| caa68e5a-ec80-4149-8c9e-ef7eff111d07 | QQQ | LONG | BOS_Scalp_1m | SL | 577.13 | 577.4963 | 575.97574 | 575.7044 | -110.32 | -0.0552 |
| 23956057-55d1-4da3-bb0a-0f87e6af5b6c | SPY | LONG | Engulfing_Bar_V056 | time_stop | 643.3999 | 643.726 | 642.7293 | 642.2794 | -120.28 | -0.0601 |
| 92d9748c-8480-494a-ae36-97741fcd136f | SPY | LONG | BOS_Scalp_1m | time_stop | 643.38 | 643.711 | 642.56 | 642.1645 | -112.61 | -0.0563 |
| a0fcf795-e848-48ee-837f-7d522c61e6ed | SPY | LONG | Liquidity_Sweep_Scalp | time_stop | 642.7301 | 643.1557 | 643.12 | 642.7991 | -84.35 | -0.0844 |
| 425a1fbe-de11-4ba2-a0c0-f3421ca56be7 | QQQ | LONG | Liquidity_Sweep_Scalp | time_stop | 575.92 | 576.2555 | 576.45 | 576.1141 | -84.6 | -0.0846 |
| 8676382d-fdba-4bc9-a2b0-1bb37ef08e34 | SPY | LONG | Liquidity_Sweep_Scalp | time_stop | 642.73 | 643.1156 | 643.02 | 642.6342 | -59.4 | -0.0594 |
| 410fbfce-d56d-458f-a5bd-613be3483fdd | QQQ | LONG | Morning_Trap_Reversal | SL | 577.595 | 577.9666 | 575.067405 | 574.2902 | -96.5 | -0.0965 |
| ac223001-faac-4216-bd04-41a52d4fe1d8 | QQQ | SHORT | Liquidity_Sweep_Scalp | time_stop | 574.09 | 573.5856 | 575.36 | 575.7102 | -106.82 | -0.1068 |
| 00c38514-dfea-425d-8511-46ea763c8df3 | QQQ | LONG | BOS_Scalp_1m | SL | 575.14 | 575.4901 | 573.3448599999999 | 572.921 | -129.24 | -0.1292 |
| c81d5aa0-fdbb-4189-af8f-1b73b9c95fb3 | QQQ | SHORT | Liquidity_Sweep_Scalp | TP1 | 575.06 | 574.695 | 573.2824099999999 | 573.609 | -85.76 | -0.0858 |
| 427b19d5-b81e-4f1f-953e-4824c837676d | QQQ | SHORT | Liquidity_Sweep_Scalp | SL | 574.84 | 574.4951 | 573.4314519999999 | 573.3288 | -30.04 | -0.03 |
| 609cc79e-8768-49ab-b373-d46d1aa300a7 | QQQ | SHORT | BOS_Scalp_1m | SL | 574.07 | 573.6456 | 572.396206 | 572.6334 | -110.48 | -0.1105 |
| 5e5e5592-1aa6-4e1f-a4ef-18cac2a08508 | SPY | LONG | Engulfing_Bar_V056 | time_stop | 642.16 | 642.4953 | 641.85 | 641.4949 | -53.16 | -0.0532 |
| 6f63fe3c-193c-4d2d-85b0-e3fbb8dc223e | SPY | LONG | BOS_Scalp_1m | time_stop | 642.13 | 642.4953 | 641.615 | 641.225 | -58.15 | -0.0582 |
| ac1a97d3-5113-46f3-905d-f7cbc7e972c4 | SPY | LONG | Engulfing_Bar_V056 | time_stop | 642.61 | 642.9906 | 641.2925 | 640.9202 | -57.97 | -0.058 |
| 97fc88da-f9e6-42f9-b36f-046fcb9a5961 | QQQ | SHORT | Morning_Trap_Reversal | TP1 | 574.68 | 574.3852 | 569.7759599999999 | 570.1369 | -81.97 | -0.082 |
| fb9f8263-5095-4d26-895f-c66bb92b5263 | QQQ | SHORT | BOS_Scalp_1m | SL | 567.16 | 566.8697 | 566.460296 | 567.2051 | -174.94 | -0.1749 |