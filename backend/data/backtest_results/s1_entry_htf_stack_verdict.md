# S1 Stack (C3 entry confirm + C4 HTF gate) — verdict 2026-04-20

## Lever
Stack de deux filtres orthogonaux :
- **Entry confirm** (C3) sur les 4 targets : `require_close_above_trigger: true`, `entry_buffer_bps: 2.0`
- **HTF gate D** (C4) uniquement sur continuation : BOS_Scalp_1m, Engulfing_Bar_V056 — reversal (Morning_Trap, Liquidity_Sweep) laissés hors-gate

YAML: `backend/knowledge/campaigns/s1_entry_htf_stack.yml`.

## Résultats comparés (4 semaines caps actives, allowlist 4 targets)

| Playbook | n | S1 E[R] | base | C3 only | C4 only | S1 | net (−0.065) |
|---|---|---|---|---|---|---|---|
| Engulfing_Bar_V056 (gated) | 38 | **+0.085** | -0.102 | -0.048 | -0.033 | **+0.085** | **+0.020** ✓ |
| Morning_Trap_Reversal (control) | 30 | +0.023 | -0.169 | +0.023 | -0.050 | +0.023 | -0.042 |
| Liquidity_Sweep_Scalp (control) | 47 | -0.005 | -0.034 | -0.006 | -0.006 | -0.005 | -0.070 |
| BOS_Scalp_1m (gated) | 57 | -0.116 | -0.115 | -0.095 | -0.121 | -0.116 | -0.181 |

## Analyse synergie

Pour les targets gated par les DEUX filtres :

- **Engulfing** : Δ_C3=+0.054, Δ_C4=+0.069, **Δ_S1=+0.187** → synergie réelle (additive simple donnerait +0.123).
  - WR grimpe 41.7% → 57.9%.
  - TP1 share 5.6% → 10.5%.
  - Time_stop baisse 38.9% → 50.0% (trades restants durent plus longtemps = trades de meilleure qualité qui atteignent leur horizon).
  - **Interprétation** : entry confirm élimine les stop-hunts, HTF gate élimine les contre-tendances. Les deux filtres attrapent des classes de failures différentes.

- **BOS_Scalp_1m** : Δ_S1 ≈ 0 (-0.116 vs -0.115 baseline). Confirmation que le signal est défaillant en profondeur, aucun filtre n'aide.

Pour les targets control (C3 seul, HTF inchangé) :
- **Morning_Trap** : identique à C3 seul (+0.023). Sanity check OK.
- **Liquidity_Sweep** : identique à C3 seul (-0.005). Sanity check OK.

## Verdict

- **1/4 playbook franchit net E[R]>0 : Engulfing_Bar_V056 +0.020 après slippage, n=38, WR 57.9%.** Premier playbook produit-grade du corpus calibration.
- **Morning_Trap** reste à net -0.042 (meilleur net de MT jamais atteint). Besoin d'un 3ème lever orthogonal pour franchir.
- **BOS_Scalp_1m** : signal dead, aucune combinaison ne récupère. Candidat KILL.
- **Liquidity_Sweep** : plateau à -0.005 gross. Signal marginal mais pas de levier qui bouge le cadran.

## Prochaines étapes proposées

1. **Engulfing S1** → promouvoir en LAB (pas de SAFE tant que pas 2-3 semaines OOS hors corpus).
2. **Morning_Trap** : essayer S2 = C3 + C1 (TP réaliste, tp1=1.10R), voir si MT crosse net zero.
3. **BOS_Scalp_1m** : proposer KILL (DENYLIST) — 3 configs testées, aucune au-dessus de -0.095.
4. **Liquidity_Sweep** : hold, pas de levier trouvé mais pas de régression franche.
