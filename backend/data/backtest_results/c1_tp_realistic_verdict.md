# C1 TP realistic — verdict 2026-04-20

## Lever
TP1/TP2/min_rr alignés sur peak_R empirique (p60 MFE) de calib_corpus_v1, au lieu des fixed RR 2.0-5.0R. BE/SL/duration/trailing inchangés pour isoler l'effet.

Patches appliqués (source: `calib_corpus_v1/` peak_R percentiles) :
- Morning_Trap_Reversal: tp1=1.10R, tp2=2.65R, min_rr=1.10
- Engulfing_Bar_V056: tp1=0.70R, tp2=1.10R, min_rr=0.70
- BOS_Scalp_1m: tp1=0.40R, tp2=0.85R, min_rr=0.40
- Liquidity_Sweep_Scalp: tp1=0.55R, tp2=1.00R, min_rr=0.55

## Config
- 4 semaines caps actives (jun_w3, aug_w3, oct_w2, nov_w4)
- `RISK_EVAL_ALLOW_ALL_PLAYBOOKS=true RISK_EVAL_RELAX_CAPS=false RISK_EVAL_DISABLE_KILL_SWITCH=true`
- Allowlist restreinte aux 4 targets
- YAML: `backend/knowledge/campaigns/c1_tp_realistic.yml`

## Résultats

| Playbook | n | E[R] | baseline | Δ | WR | net (−0.065) |
|---|---|---|---|---|---|---|
| Morning_Trap_Reversal | — | -0.066 | -0.169 | +0.103 | — | -0.131 |
| Engulfing_Bar_V056    | — | -0.070 | -0.102 | +0.032 | — | -0.135 |
| BOS_Scalp_1m          | — | -0.158 | -0.115 | -0.043 | — | -0.223 |
| Liquidity_Sweep_Scalp | — | -0.058 | -0.034 | -0.024 | — | -0.123 |

## Verdict

- **2/4 positifs, 2/4 regress.** Morning_Trap bénéficie le plus (+0.103) — TP à 1.10R au lieu de 2.0R capture les MFE réels.
- BOS_Scalp_1m régresse (-0.043) : TP1 0.40R trop serré pour ce signal, nuit au R/trade sans améliorer la WR.
- Liquidity_Sweep régresse (-0.024) : TP1 0.55R n'a pas aidé la distribution.
- **Gate** : 0/4 atteint net E[R]>0 après slippage -0.065R/trade.
- **Enseignement** : TP calibration seule déplace la distribution (winners plus nombreux) mais ne corrige pas la sélection (signal-quality issue sur BOS/LSweep persistante).
