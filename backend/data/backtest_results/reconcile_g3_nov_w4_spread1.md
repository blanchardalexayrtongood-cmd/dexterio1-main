# Paper-vs-backtest reconcile — trades_miniweek_calib_corpus_v1_nov_w4_AGGRESSIVE_DAILY_SCALP.parquet

**Slippage model:** ConservativeFillModel, next-bar-open fills with 0.050% extra slippage + 1.00 bps spread (§0.7 G3, asymmetric buyer/seller).

**Interpretation:** negative delta_$ = conservative model would have been worse than the backtest's ideal fill.

## Aggregate

| metric | value |
|---|---|
| n_trades | 40 |
| total_delta_$ | -3793.83 |
| mean_delta_$ | -94.85 |
| p50_delta_$ | -97.08 |
| p95_delta_$ | -39.52 |
| min_delta_$ | -206.04 |
| max_delta_$ | 77.29 |
| total_delta_R | -3.404 |
| mean_delta_R | -0.0851 |
| pct_trades_worse | 97.5 |

## First 20 trades

| trade_id | symbol | direction | playbook | exit_reason | ideal_entry | conservative_entry | ideal_exit | conservative_exit | delta_$ | delta_R |
|---|---|---|---|---|---|---|---|---|---|---|
| 62b8c948-2b0f-403e-8058-701e06d6ba46 | QQQ | SHORT | Morning_Trap_Reversal | SL | 608.6113 | 608.4247 | 611.1936113 | 611.6067 | -98.35 | -0.0492 |
| 14dbd4d9-0346-4c3f-a249-969827e307b5 | QQQ | SHORT | BOS_Scalp_1m | SL | 608.92 | 608.6346 | 611.19392 | 611.6067 | -114.51 | -0.0573 |
| 8e9e46f7-5e4d-4900-9b86-758c613f5ed0 | QQQ | SHORT | Morning_Trap_Reversal | SL | 610.4885 | 610.0537 | 611.709477 | 612.1371 | -140.56 | -0.0703 |
| fe5e6a45-9237-414e-931c-3a9283b4a184 | QQQ | SHORT | BOS_Scalp_1m | SL | 610.4874 | 610.2636 | 611.7083748 | 612.1371 | -106.35 | -0.0532 |
| 54d5609c-2e8b-48cc-8e8e-396d55a6cf0e | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 671.985 | 671.5468 | 673.3319849999999 | 673.9241 | -114.36 | -0.0572 |
| 06d83b7f-a592-49ce-8dff-72b50c7c35f3 | QQQ | SHORT | Engulfing_Bar_V056 | SL | 611.17 | 610.9232 | 609.347925 | 609.9608 | -134.96 | -0.135 |
| 329f6fa4-467d-44c0-b983-f11015dd301f | QQQ | LONG | Liquidity_Sweep_Scalp | time_stop | 611.63 | 612.1371 | 607.95 | 607.7251 | -58.55 | -0.0586 |
| 1427cdfb-d9d7-4649-aca6-d377a9c95174 | SPY | SHORT | Liquidity_Sweep_Scalp | time_stop | 671.56 | 671.1371 | 670.98 | 671.4626 | -96.89 | -0.0969 |
| b42294ba-a2e6-4c41-a832-b43d18e33809 | SPY | SHORT | BOS_Scalp_1m | SL | 671.545 | 671.1071 | 668.7773175 | 669.7316 | -206.04 | -0.103 |
| 1f275b61-afb5-4317-b0ed-5470d0492b64 | QQQ | SHORT | Engulfing_Bar_V056 | SL | 609.2 | 609.1143 | 606.9846 | 607.6043 | -110.75 | -0.1108 |
| 2f1435b2-9060-49f2-96e9-b48e88668cce | QQQ | SHORT | Engulfing_Bar_V056 | SL | 600.205 | 599.9498 | 601.4054100000001 | 600.6702 | 77.29 | 0.0773 |
| bce698b1-94ea-40ec-9dc8-f70deec1730b | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 663.14 | 662.7121 | 662.074942 | 662.2271 | -41.76 | -0.0418 |
| 76361457-2823-4c6c-a09e-066bea734f15 | QQQ | SHORT | Engulfing_Bar_V056 | SL | 600.08 | 599.7 | 598.61504 | 599.9147 | -134.38 | -0.1344 |
| c64614f2-1dba-4d02-80cf-54023ad20a83 | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 661.85 | 661.4329 | 659.691555 | 659.9357 | -71.42 | -0.0714 |
| 3a471094-575c-4a9a-aa1c-951b05a8aa29 | SPY | SHORT | Liquidity_Sweep_Scalp | SL | 660.96 | 660.6534 | 659.431638 | 659.9357 | -88.37 | -0.0884 |
| cb8dd745-63cd-4fb2-aabd-4ab5643e63a2 | QQQ | LONG | Morning_Trap_Reversal | SL | 594.86 | 595.0068 | 593.67028 | 593.3288 | -39.55 | -0.0396 |
| 48b4382e-6662-49c7-ba9f-fadf00fd2d57 | QQQ | SHORT | BOS_Scalp_1m | SL | 597.3904 | 597.1215 | 594.24509712 | 595.0168 | -167.53 | -0.1675 |
| 0a5b51e7-0c56-4bc8-83b0-6f91035df88f | SPY | SHORT | BOS_Scalp_1m | SL | 660.13 | 659.7139 | 657.297039 | 658.0646 | -171.63 | -0.1716 |
| b21fe344-0df3-4f35-9e3f-b408af32758b | QQQ | SHORT | Morning_Trap_Reversal | SL | 598.05 | 597.7212 | 598.05 | 598.2087 | -39.0 | -0.039 |
| 7fcda771-1bb9-4b95-ad69-ccdf41c9aeb0 | SPY | LONG | BOS_Scalp_1m | time_stop | 663.525 | 663.8831 | 661.83 | 661.3929 | -59.64 | -0.0596 |