# C2 Trailing aggressive — verdict 2026-04-20

## Lever
Trailing ratchet déclenché à 0.5R avec offset 0.2R sur les 4 targets. TP/SL/BE inchangés pour isoler l'effet du trailing seul.

Patches :
- trailing_mode: `trail_rr`
- trailing_trigger_rr: 0.5 (vs 0.8-1.0 avant)
- trailing_offset_rr: 0.2
- Appliqué à Morning_Trap_Reversal, Engulfing_Bar_V056, BOS_Scalp_1m, Liquidity_Sweep_Scalp

## Config
Identique à C1. YAML: `backend/knowledge/campaigns/c2_trailing_aggressive.yml`.

## Résultats

| Playbook | E[R] | baseline | Δ | net |
|---|---|---|---|---|
| Morning_Trap_Reversal | -0.099 | -0.169 | +0.070 | -0.164 |
| Engulfing_Bar_V056    | -0.096 | -0.102 | +0.006 | -0.161 |
| BOS_Scalp_1m          | -0.131 | -0.115 | -0.016 | -0.196 |
| Liquidity_Sweep_Scalp | -0.056 | -0.034 | -0.022 | -0.121 |

## Verdict

- **2/4 positif mais marginal.** Morning_Trap +0.070 (trail serre sur les winners 1-2R, evite le retour à BE). Engulfing +0.006 négligeable.
- BOS + LSweep régressent : trailing 0.5R se fait toucher trop tôt sur ces signaux chop, les wins potentiels sortent en ratchet SL.
- **0% exits en `trail_exit`** dans le parquet : le mécanisme trailing met à jour le SL mais l'exit reste attribué `SL` quand le prix revient au niveau ratché. Lever actif malgré le label.
- **Gate** : 0/4 net E[R]>0.
- **Enseignement** : trailing aggressive aide seulement quand peak_R habite >1R (Morning_Trap). Sur chop signals (BOS/LSweep), serre prematurément. Pas un lever universel.
