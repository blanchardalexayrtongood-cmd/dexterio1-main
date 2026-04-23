# Paper-vs-backtest reconcile — trades_miniweek_calib_corpus_v1_jun_w3_AGGRESSIVE_DAILY_SCALP.parquet

**Slippage model:** ConservativeFillModel, next-bar-open fills with 0.050% extra slippage + 1.00 bps spread (§0.7 G3, asymmetric buyer/seller).

**Interpretation:** negative delta_$ = conservative model would have been worse than the backtest's ideal fill.

## Aggregate

| metric | value |
|---|---|
| n_trades | 40 |
| total_delta_$ | -4485.37 |
| mean_delta_$ | -112.13 |
| p50_delta_$ | -90.11 |
| p95_delta_$ | -49.02 |
| min_delta_$ | -595.56 |
| max_delta_$ | -30.01 |
| total_delta_R | -4.109 |
| mean_delta_R | -0.1027 |
| pct_trades_worse | 100.0 |

## First 20 trades

| trade_id | symbol | direction | playbook | exit_reason | ideal_entry | conservative_entry | ideal_exit | conservative_exit | delta_$ | delta_R |
|---|---|---|---|---|---|---|---|---|---|---|
| 0510c14c-9f9e-41f0-b180-1b6c72dd8cb2 | SPY | SHORT | Liquidity_Sweep_Scalp | time_stop | 603.05 | 602.6982 | 604.07 | 604.4424 | -59.39 | -0.0297 |
| 9df1424e-e78b-411e-8e5d-3043758dbd3d | SPY | LONG | Engulfing_Bar_V056 | time_stop | 601.53 | 601.8909 | 603.46 | 603.0879 | -121.68 | -0.0608 |
| cc190627-e4f2-4891-98fa-ce6d915104f9 | QQQ | LONG | Engulfing_Bar_V056 | time_stop | 531.83 | 531.6688 | 534.69 | 534.3692 | -30.01 | -0.015 |
| 72b2d2b1-4961-4ece-90f4-a8e4f9ffa9bf | SPY | LONG | BOS_Scalp_1m | time_stop | 601.37 | 601.7508 | 603.3331 | 602.933 | -129.63 | -0.0648 |
| 7f3ad163-a322-490b-a0ce-dbd9e2838d67 | QQQ | LONG | BOS_Scalp_1m | time_stop | 531.73 | 532.0891 | 534.54 | 534.2593 | -120.28 | -0.0601 |
| 7c283399-4dad-4bae-94d9-a2625a8aadc5 | QQQ | LONG | BOS_Scalp_1m | time_stop | 532.06 | 532.3592 | 534.7401 | 534.4291 | -114.1 | -0.0571 |
| 23ff2e48-537b-4131-83be-a2d93be5ece3 | SPY | LONG | Liquidity_Sweep_Scalp | time_stop | 603.24 | 603.6019 | 603.2979 | 602.948 | -58.37 | -0.0584 |
| 7d941243-d52c-4ec8-bd3e-c676c78de8a8 | QQQ | SHORT | Liquidity_Sweep_Scalp | time_stop | 534.16 | 533.6846 | 534.04 | 534.3604 | -152.8 | -0.1528 |
| 3da1bb5b-d61a-49dd-a4f6-4e17f5c01e86 | SPY | SHORT | Morning_Trap_Reversal | TP1 | 603.435 | 603.1179 | 599.81439 | 600.5801 | -88.79 | -0.0444 |
| 33fd475f-f96a-4bf3-be2f-b60b3c63ab6a | QQQ | LONG | Liquidity_Sweep_Scalp | SL | 532.19 | 532.6094 | 530.96781 | 530.6314 | -109.58 | -0.1096 |
| b5d5a97f-68aa-4be7-9ea3-b2bee830e965 | QQQ | LONG | BOS_Scalp_1m | SL | 532.18 | 532.4993 | 530.96782 | 530.6314 | -95.08 | -0.0951 |
| 7b1a490e-b9f5-46d5-baec-45ea810a5850 | SPY | SHORT | Engulfing_Bar_V056 | SL | 600.43 | 600.0198 | 599.78043 | 600.5301 | -198.34 | -0.1983 |
| 92db509b-ffd3-4b32-807b-ee7610845363 | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 599.99 | 599.615 | 601.18998 | 601.3206 | -64.71 | -0.0647 |
| fb051ee9-a763-4d20-bb7c-0cb0943a739c | SPY | LONG | Liquidity_Sweep_Scalp | time_stop | 600.545 | 600.8403 | 601.155 | 600.8793 | -47.4 | -0.0474 |
| 6f870a17-951b-4579-bade-61b75ef50246 | QQQ | LONG | Engulfing_Bar_V056 | time_stop | 532.15 | 532.4693 | 532.225 | 531.8407 | -135.79 | -0.1358 |
| e2d11064-04d4-488d-90f2-982ed0eacfe7 | SPY | LONG | BOS_Scalp_1m | time_stop | 601.14 | 601.4907 | 601.26 | 600.9492 | -54.9 | -0.0549 |
| 7cdb67b5-460b-4e66-8bef-6e09b2652054 | QQQ | LONG | BOS_Scalp_1m | time_stop | 533.04 | 533.3998 | 533.11 | 532.8351 | -120.6 | -0.1206 |
| 983f602e-d3bc-4c68-a6ed-689cafbdfc87 | SPY | SHORT | BOS_Scalp_1m | SL | 597.8102 | 597.4813 | 599.0058204000001 | 599.3794 | -118.02 | -0.118 |
| d433bca3-9dbb-41e4-b4f1-b6455ee9dbcd | QQQ | SHORT | Morning_Trap_Reversal | SL | 529.7 | 529.4022 | 530.7594 | 531.1185 | -93.28 | -0.0933 |
| ebfeb648-130c-4a28-aaba-2b9f9a91a85d | QQQ | LONG | BOS_Scalp_1m | SL | 529.53 | 529.8677 | 530.5420409999999 | 530.3066 | -108.9 | -0.1089 |