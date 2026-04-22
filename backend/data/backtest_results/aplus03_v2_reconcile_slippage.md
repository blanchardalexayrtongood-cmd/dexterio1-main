# Paper-vs-backtest reconcile — aplus03_v2_all_trades.parquet

**Slippage model:** ConservativeFillModel, 0.050% adverse slippage on next-bar-open fills.

**Interpretation:** negative delta_$ = conservative model would have been worse than the backtest's ideal fill.

## Aggregate

| metric | value |
|---|---|
| n_trades | 22 |
| total_delta_$ | -1171.64 |
| mean_delta_$ | -53.26 |
| p50_delta_$ | -51.28 |
| p95_delta_$ | -34.8 |
| min_delta_$ | -89.75 |
| max_delta_$ | -33.98 |
| total_delta_R | -0.982 |
| mean_delta_R | -0.0446 |
| pct_trades_worse | 100.0 |

## First 20 trades

| trade_id | symbol | direction | playbook | exit_reason | ideal_entry | conservative_entry | ideal_exit | conservative_exit | delta_$ | delta_R |
|---|---|---|---|---|---|---|---|---|---|---|
| 1044a096-6e79-4d8b-9959-d807d8183011 | QQQ | LONG | Aplus_03_v2 | SL | 577.555 | 577.8888 | 575.922445 | 575.642 | -52.82 | -0.0264 |
| 949cbe8e-e0db-48c9-808a-e1a25d72a5a1 | QQQ | LONG | Aplus_03_v2 | SL | 577.335 | 577.5886 | 575.922665 | 575.642 | -45.95 | -0.023 |
| ebdfd3e1-3ec9-4ff8-8126-1a3ce47da8b5 | QQQ | SHORT | Aplus_03_v2 | SL | 576.1775 | 575.8969 | 576.1775 | 576.338 | -37.94 | -0.0379 |
| 996eff06-ef97-4edd-bb4c-f32e9dffeac9 | QQQ | SHORT | Aplus_03_v2 | SL | 574.71 | 574.2627 | 574.0149274999999 | 574.4071 | -72.19 | -0.0722 |
| ff711b9c-36eb-4f55-b6bf-2dda643b6ebd | QQQ | SHORT | Aplus_03_v2 | SL | 574.09 | 573.643 | 572.2797724999999 | 572.386 | -47.58 | -0.0476 |
| 963ad9f0-c0a5-4f9b-b87c-af09380e961c | QQQ | SHORT | Aplus_03_v2 | SL | 573.095 | 572.8135 | 571.39952375 | 571.8858 | -66.03 | -0.066 |
| 1a03628d-8f20-47be-8a43-a7bcbb1d08e6 | SPY | LONG | Aplus_03_v2 | time_stop | 636.755 | 637.0284 | 635.56 | 635.2822 | -42.99 | -0.043 |
| 5224cea7-43ca-4780-8322-7b9505f3b462 | QQQ | SHORT | Aplus_03_v2 | TP1 | 564.79 | 564.5176 | 563.378025 | 563.5816 | -41.41 | -0.0414 |
| 81512811-7baa-43b5-9082-8cf17776f9a5 | QQQ | SHORT | Aplus_03_v2 | TP1 | 564.93 | 564.7075 | 562.88 | 563.1714 | -44.71 | -0.0447 |
| 05551a44-1472-4c79-a321-a67853a20b4e | SPY | SHORT | Aplus_03_v2 | TP1 | 637.03 | 636.5815 | 635.4374250000001 | 636.0279 | -79.99 | -0.08 |
| b34b1590-61ec-4d7c-bc7f-83144d2f3529 | SPY | LONG | Aplus_03_v2 | time_stop | 645.6554 | 645.9628 | 645.33 | 644.9923 | -49.67 | -0.0497 |
| ce460795-6111-4589-b040-9d341a40a235 | SPY | SHORT | Aplus_03_v2 | SL | 597.35 | 596.9614 | 595.9855375 | 596.6782 | -89.75 | -0.0449 |
| 3326ecb6-8fb3-4677-9d12-b40922227e27 | QQQ | SHORT | Aplus_03_v2 | time_stop | 527.05 | 526.7765 | 526.29 | 526.6332 | -57.97 | -0.029 |
| a479adcf-dd47-47be-a519-6a401b91d718 | QQQ | SHORT | Aplus_03_v2 | SL | 609.3901 | 609.1653 | 608.367322525 | 608.4241 | -34.63 | -0.0173 |
| 33896d5d-04f4-409a-b9b6-81e17819d2e8 | SPY | SHORT | Aplus_03_v2 | SL | 671.74 | 671.4141 | 671.0329350000001 | 671.4906 | -57.98 | -0.029 |
| e994bd6e-f17e-49a0-b988-b7722cbf1a86 | SPY | SHORT | Aplus_03_v2 | TP1 | 668.1348 | 667.746 | 666.7093826 | 667.0083 | -50.9 | -0.0509 |
| 011c5737-b194-49a8-b66a-40382623902f | SPY | SHORT | Aplus_03_v2 | TP1 | 660.96 | 660.7195 | 659.85727 | 660.0699 | -33.98 | -0.034 |
| 5ab7e4d7-0b5c-40eb-82c3-d767fb2f2bc0 | SPY | SHORT | Aplus_03_v2 | SL | 658.65 | 658.2107 | 658.65 | 658.9293 | -54.61 | -0.0546 |
| bfd2debb-d20e-487c-87c9-a362f4c4b6d3 | QQQ | SHORT | Aplus_03_v2 | time_stop | 594.73 | 594.3227 | 595.33 | 595.5376 | -51.65 | -0.0517 |
| 642963ed-f3c9-46d1-afb5-e4609f85d9b7 | QQQ | SHORT | Aplus_03_v2 | time_stop | 597.17 | 597.0014 | 596.6741 | 597.1784 | -55.86 | -0.0559 |