# C3 Entry confirm — verdict 2026-04-20

## Lever
Gate d'entrée exigeant que la bougie 1m **ferme** au-delà du trigger + buffer de 2 bps, au lieu de trigger intrabar uniquement.

- YAML flags : `require_close_above_trigger: true`, `entry_buffer_bps: 2.0`
- Appliqué aux 4 targets
- Code : [playbook_loader.py:149-155](backend/engines/playbook_loader.py#L149-L155) parsing, [engine.py:1806-1856](backend/backtest/engine.py#L1806-L1856) gate enforcement
- Tests : `backend/tests/test_entry_confirmation_gate.py` 4/4 pass

## Config
Identique à C1/C2. YAML: `backend/knowledge/campaigns/c3_entry_confirm.yml`.

## Résultats

| Playbook | n | E[R] | baseline | Δ | WR | exit_mix | net |
|---|---|---|---|---|---|---|---|
| Morning_Trap_Reversal | 30 | **+0.023** | -0.169 | **+0.192** | 30.0% | SL63/TP30/eod7 | -0.042 |
| Engulfing_Bar_V056    | 36 | -0.048 | -0.102 | +0.054 | 41.7% | SL56/ts39/TP6 | -0.113 |
| BOS_Scalp_1m          | 54 | -0.095 | -0.115 | +0.020 | 40.7% | ts50/SL46/TP4 | -0.160 |
| Liquidity_Sweep_Scalp | 49 | -0.006 | -0.034 | +0.028 | 36.7% | ts69/SL29/TP2 | -0.071 |

## Verdict

- **4/4 positifs.** Premier lever qui améliore tous les targets simultanément.
- **Morning_Trap_Reversal devient gross positif** (E[R] +0.023, WR 30%) — première fois un target calib croise zéro. Net -0.042 reste négatif mais plus proche qu'aucun autre lever/phase.
- Mécanisme : la confirmation de close élimine les entrées sur stop-hunt/wick qui partent immédiatement en SL. Sur Morning_Trap (reversal) : SL share passe de ~77% à 63%, TP1 share grimpe à 30%.
- **Gate** : 0/4 net E[R]>0, mais **Morning_Trap à -0.042 est le meilleur net jamais obtenu** sur tout le corpus calibration.
- **Enseignement** : filtre d'entrée = levier dominant. Valide l'hypothèse "signal-quality ceiling" — c'est l'entrée pas le TP qui pose problème.
