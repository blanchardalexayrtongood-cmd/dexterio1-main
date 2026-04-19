# Phase B0.1b — Normal-caps re-run delta (oct_w2)

**Config:** `RISK_EVAL_ALLOW_ALL_PLAYBOOKS=true RISK_EVAL_DISABLE_KILL_SWITCH=true` **without** `RISK_EVAL_RELAX_CAPS` (cooldown 5min + session cap 10/playbook active).

**Window:** 2025-10-06..2025-10-10, SPY+QQQ.

## Portfolio delta

| metric | RELAX_CAPS=true | caps ACTIVE | delta |
|---|---|---|---|
| trades | 1877 | 48 | **-97.4%** |
| total_R | -222.63 | -11.59 | - |
| E[R] | -0.1186 | -0.2414 | **-0.1228 (WORSE)** |

## Per-playbook delta

| playbook | relax_trades | normal_trades | reduction | E[R]_relax | E[R]_normal | verdict |
|---|---|---|---|---|---|---|
| DAY_Aplus_1_Liquidity_Sweep_OB_Retest | 866 | 12 | 98.6% | -0.2395 | -0.5180 | SPAM + DESTROYER (stays DENYLIST) |
| SCALP_Aplus_1_Mini_FVG_Retest_NY_Open | 927 | 22 | 97.6% | -0.0123 | **-0.1388** | SPAM confirmed — low E[R] was averaging artifact |
| NY_Open_Reversal | 5 | 1 | 80% | -0.0853 | -0.0613 | (DENYLIST after B0.3 anyway) |
| ORB_Breakout_5m | 4 | 0 | 100% | +0.0048 | n/a | (DENYLIST after B0.3 anyway) |
| Morning_Trap_Reversal | 8 | 4 | 50% | -0.1173 | -0.3529 | too few trades under caps |
| Liquidity_Sweep_Scalp | 15 | 6 | 60% | -0.0410 | -0.1355 | too few trades under caps |
| Engulfing_Bar_V056 | 10 | 0 | 100% | -0.0215 | n/a | **starved** — daily cap consumed by Aplus_1 |
| BOS_Scalp_1m | 15 | 0 | 100% | -0.0690 | n/a | **starved** |
| News_Fade | 5 | 2 | 60% | +0.0020 | +0.0208 | stable |
| Session_Open_Scalp | 5 | 1 | 80% | -0.0233 | -0.0709 | too few |
| IFVG_5m_Sweep | 6 | 0 | 100% | -0.0218 | n/a | **starved** |
| RSI_MeanRev_5m | 8 | 0 | 100% | -0.0518 | n/a | **starved** |
| VWAP_Bounce_5m | 1 | 0 | 100% | +0.0136 | n/a | **starved** |
| HTF_Bias_15m_BOS | 1 | 0 | 100% | +0.0304 | n/a | **starved** |
| FVG_Fill_Scalp | 1 | 0 | 100% | -0.0077 | n/a | **starved** |

## Conclusions

1. **SCALP_Aplus_1 confirmed SPAM:** under caps, only 22 trades (vs 927) and E[R] crashes from -0.01 to **-0.14**. The "low-negative" E[R] under RELAX was a large-sample averaging artifact — most of the spam trades were small-loss / time-stop, masking the real negative edge. **Excluded from B1 calibration** (confirming B0.1a).
2. **DAY_Aplus_1 worse under caps:** 12 trades, E[R]=-0.52 (vs -0.24). Stays DENYLIST.
3. **Cap-starvation confirmed for CALIBRATE candidates:** Engulfing_Bar_V056, BOS_Scalp_1m (both E[R] candidates in Phase A verdict) fire **0 trades** in oct_w2 when caps are active AND SCALP/DAY Aplus_1 are running. Root cause: `CIRCUIT_MAX_TRADES_DAY_SYMBOL=12` is consumed by the high-frequency Aplus_1 playbooks, starving downstream playbooks on the same symbol/day.
4. **B0.4 design validated:** the restricted CALIB_ALLOWLIST (4 candidates, SCALP/DAY excluded) is necessary to get clean session data. Running the full allowlist with caps would give zero trades for Engulfing/BOS_Scalp/Liquidity_Sweep/Morning_Trap in production conditions — exactly the situation B0.4 is designed to fix.

## Artifact
- [fair_oct_w2_normalcaps/](../../results/labs/mini_week/fair_oct_w2_normalcaps/)
