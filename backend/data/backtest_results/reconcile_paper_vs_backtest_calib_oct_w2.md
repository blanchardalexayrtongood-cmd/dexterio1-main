# Paper-vs-backtest reconcile — trades_miniweek_calib_corpus_v1_oct_w2_AGGRESSIVE_DAILY_SCALP.parquet

**Slippage model:** ConservativeFillModel, 0.050% adverse slippage on next-bar-open fills.

**Interpretation:** negative delta_$ = conservative model would have been worse than the backtest's ideal fill.

## Aggregate

| metric | value |
|---|---|
| n_trades | 40 |
| total_delta_$ | -2889.25 |
| mean_delta_$ | -72.23 |
| p50_delta_$ | -69.82 |
| p95_delta_$ | -34.39 |
| min_delta_$ | -150.41 |
| max_delta_$ | -18.44 |
| total_delta_R | -2.588 |
| mean_delta_R | -0.0647 |
| pct_trades_worse | 100.0 |

## First 20 trades

| trade_id | symbol | direction | playbook | exit_reason | ideal_entry | conservative_entry | ideal_exit | conservative_exit | delta_$ | delta_R |
|---|---|---|---|---|---|---|---|---|---|---|
| d5143c64-0bc9-45d7-bcb0-319c8fecb922 | SPY | LONG | Liquidity_Sweep_Scalp | time_stop | 670.32 | 670.6001 | 670.3 | 670.0048 | -42.57 | -0.0213 |
| 872c7a72-a6c1-40a2-9db3-1588047ed08e | SPY | SHORT | Liquidity_Sweep_Scalp | time_stop | 670.515 | 670.1448 | 670.7601 | 671.0954 | -52.21 | -0.0522 |
| ef15001b-03c5-49e5-b43b-5936ff8e187f | SPY | SHORT | Morning_Trap_Reversal | SL | 671.0 | 670.6545 | 671.0 | 671.3205 | -73.92 | -0.037 |
| cc52564f-bce6-419f-bc4e-cd7f02fefc06 | SPY | LONG | Liquidity_Sweep_Scalp | time_stop | 670.79 | 671.0654 | 671.09 | 670.7445 | -45.95 | -0.0459 |
| a3ee1761-0e62-4029-bfbc-86e63dfd57c1 | QQQ | LONG | Engulfing_Bar_V056 | time_stop | 607.19 | 607.5136 | 607.445 | 607.1463 | -102.06 | -0.051 |
| 6cfcb46b-0965-46db-b316-e11b43e30ca3 | SPY | LONG | BOS_Scalp_1m | time_stop | 670.8 | 671.1304 | 671.295 | 670.9594 | -99.24 | -0.0496 |
| 6b3dde01-0034-45df-9239-d700cc91c78a | QQQ | LONG | BOS_Scalp_1m | time_stop | 607.33 | 607.6487 | 607.45 | 607.1363 | -103.71 | -0.0519 |
| e48b9b2d-a591-4433-805f-9b965624a7c0 | QQQ | LONG | Engulfing_Bar_V056 | time_stop | 607.19 | 607.4986 | 607.45 | 607.1363 | -102.06 | -0.051 |
| af1c566f-ed58-480f-a757-14eb2e879163 | QQQ | LONG | BOS_Scalp_1m | time_stop | 607.41 | 607.7737 | 607.45 | 607.1663 | -79.64 | -0.0398 |
| 856ac9ee-d2c2-4b39-ab08-a4c8a7a6ea0a | SPY | SHORT | Morning_Trap_Reversal | SL | 670.51 | 670.2547 | 671.85102 | 672.1259 | -39.23 | -0.0392 |
| b074f0ca-2559-49c3-a1e6-c843e240fa03 | QQQ | LONG | Liquidity_Sweep_Scalp | time_stop | 609.16 | 609.4846 | 608.475 | 608.1758 | -100.44 | -0.1004 |
| 3e022582-921b-4399-acfd-842d7fc1a141 | QQQ | LONG | Liquidity_Sweep_Scalp | time_stop | 608.63 | 608.8343 | 608.71 | 608.3657 | -88.32 | -0.0883 |
| 76f41dc7-ebb0-4c11-9bc9-65ca41a3bb44 | QQQ | LONG | Liquidity_Sweep_Scalp | time_stop | 609.2509 | 609.5596 | 608.1101 | 607.8109 | -97.87 | -0.0979 |
| 11fa7efe-faea-4983-ad13-ba6e45b9316e | QQQ | LONG | BOS_Scalp_1m | SL | 608.76 | 609.1544 | 607.39124 | 606.8514 | -150.41 | -0.1504 |
| 6abe3de6-4541-48fe-8fcb-240e31f5b33e | QQQ | LONG | BOS_Scalp_1m | SL | 608.41 | 608.8243 | 607.19318 | 606.8514 | -121.72 | -0.1217 |
| 6d65a065-1421-47af-89b7-8f4d44ebca74 | QQQ | LONG | BOS_Scalp_1m | SL | 608.24 | 608.4641 | 605.1988 | 605.2822 | -22.51 | -0.0225 |
| 35496762-f228-4126-bb48-8a1ce9ba0ff4 | QQQ | LONG | Engulfing_Bar_V056 | time_stop | 604.14 | 604.4421 | 603.79 | 603.4581 | -98.9 | -0.0989 |
| 4fd7d1bc-d424-46a8-a816-1c74f92eb9c3 | SPY | LONG | Engulfing_Bar_V056 | time_stop | 668.25 | 668.5941 | 668.33 | 667.9758 | -98.46 | -0.0985 |
| 11fd9006-0efe-434d-9134-a2c5b47544fd | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 669.73 | 669.3751 | 671.24973 | 671.5706 | -70.95 | -0.071 |
| fa1172d3-f189-40df-9e6b-62e4936f49a3 | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 669.98 | 669.625 | 671.31996 | 671.5906 | -65.69 | -0.0657 |