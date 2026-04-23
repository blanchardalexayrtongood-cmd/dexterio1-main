# Paper-vs-backtest reconcile — trades_miniweek_calib_corpus_v1_oct_w2_AGGRESSIVE_DAILY_SCALP.parquet

**Slippage model:** ConservativeFillModel, next-bar-open fills with 0.050% extra slippage + 1.00 bps spread (§0.7 G3, asymmetric buyer/seller).

**Interpretation:** negative delta_$ = conservative model would have been worse than the backtest's ideal fill.

## Aggregate

| metric | value |
|---|---|
| n_trades | 40 |
| total_delta_$ | -3507.81 |
| mean_delta_$ | -87.7 |
| p50_delta_$ | -86.06 |
| p95_delta_$ | -48.81 |
| min_delta_$ | -169.99 |
| max_delta_$ | -28.17 |
| total_delta_R | -3.146 |
| mean_delta_R | -0.0787 |
| pct_trades_worse | 100.0 |

## First 20 trades

| trade_id | symbol | direction | playbook | exit_reason | ideal_entry | conservative_entry | ideal_exit | conservative_exit | delta_$ | delta_R |
|---|---|---|---|---|---|---|---|---|---|---|
| d5143c64-0bc9-45d7-bcb0-319c8fecb922 | SPY | LONG | Liquidity_Sweep_Scalp | time_stop | 670.32 | 670.6672 | 670.3 | 669.9378 | -52.49 | -0.0262 |
| 872c7a72-a6c1-40a2-9db3-1588047ed08e | SPY | SHORT | Liquidity_Sweep_Scalp | time_stop | 670.515 | 670.0777 | 670.7601 | 671.1625 | -62.13 | -0.0621 |
| ef15001b-03c5-49e5-b43b-5936ff8e187f | SPY | SHORT | Morning_Trap_Reversal | SL | 671.0 | 670.5874 | 671.0 | 671.3876 | -88.82 | -0.0444 |
| cc52564f-bce6-419f-bc4e-cd7f02fefc06 | SPY | LONG | Liquidity_Sweep_Scalp | time_stop | 670.79 | 671.1324 | 671.09 | 670.6774 | -55.88 | -0.0559 |
| a3ee1761-0e62-4029-bfbc-86e63dfd57c1 | QQQ | LONG | Engulfing_Bar_V056 | time_stop | 607.19 | 607.5743 | 607.445 | 607.0855 | -121.98 | -0.061 |
| 6cfcb46b-0965-46db-b316-e11b43e30ca3 | SPY | LONG | BOS_Scalp_1m | time_stop | 670.8 | 671.1975 | 671.295 | 670.8922 | -119.24 | -0.0596 |
| 6b3dde01-0034-45df-9239-d700cc91c78a | QQQ | LONG | BOS_Scalp_1m | time_stop | 607.33 | 607.7094 | 607.45 | 607.0755 | -123.63 | -0.0618 |
| e48b9b2d-a591-4433-805f-9b965624a7c0 | QQQ | LONG | Engulfing_Bar_V056 | time_stop | 607.19 | 607.5593 | 607.45 | 607.0755 | -121.98 | -0.061 |
| af1c566f-ed58-480f-a757-14eb2e879163 | QQQ | LONG | BOS_Scalp_1m | time_stop | 607.41 | 607.8345 | 607.45 | 607.1055 | -94.58 | -0.0473 |
| 856ac9ee-d2c2-4b39-ab08-a4c8a7a6ea0a | SPY | SHORT | Morning_Trap_Reversal | SL | 670.51 | 670.1876 | 671.85102 | 672.1931 | -49.17 | -0.0492 |
| b074f0ca-2559-49c3-a1e6-c843e240fa03 | QQQ | LONG | Liquidity_Sweep_Scalp | time_stop | 609.16 | 609.5455 | 608.475 | 608.1149 | -120.04 | -0.12 |
| 3e022582-921b-4399-acfd-842d7fc1a141 | QQQ | LONG | Liquidity_Sweep_Scalp | time_stop | 608.63 | 608.8951 | 608.71 | 608.3048 | -107.92 | -0.1079 |
| 76f41dc7-ebb0-4c11-9bc9-65ca41a3bb44 | QQQ | LONG | Liquidity_Sweep_Scalp | time_stop | 609.2509 | 609.6206 | 608.1101 | 607.7501 | -117.47 | -0.1175 |
| 11fa7efe-faea-4983-ad13-ba6e45b9316e | QQQ | LONG | BOS_Scalp_1m | SL | 608.76 | 609.2153 | 607.39124 | 606.7907 | -169.99 | -0.17 |
| 6abe3de6-4541-48fe-8fcb-240e31f5b33e | QQQ | LONG | BOS_Scalp_1m | SL | 608.41 | 608.8851 | 607.19318 | 606.7907 | -141.29 | -0.1413 |
| 6d65a065-1421-47af-89b7-8f4d44ebca74 | QQQ | LONG | BOS_Scalp_1m | SL | 608.24 | 608.5249 | 605.1988 | 605.2216 | -41.93 | -0.0419 |
| 35496762-f228-4126-bb48-8a1ce9ba0ff4 | QQQ | LONG | Engulfing_Bar_V056 | time_stop | 604.14 | 604.5025 | 603.79 | 603.3977 | -117.74 | -0.1177 |
| 4fd7d1bc-d424-46a8-a816-1c74f92eb9c3 | SPY | LONG | Engulfing_Bar_V056 | time_stop | 668.25 | 668.661 | 668.33 | 667.909 | -117.3 | -0.1173 |
| 11fd9006-0efe-434d-9134-a2c5b47544fd | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 669.73 | 669.3082 | 671.24973 | 671.6377 | -85.03 | -0.085 |
| fa1172d3-f189-40df-9e6b-62e4936f49a3 | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 669.98 | 669.558 | 671.31996 | 671.6578 | -79.78 | -0.0798 |