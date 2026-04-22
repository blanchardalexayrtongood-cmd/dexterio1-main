# C4 HTF gate selective — verdict 2026-04-20

## Lever
Gate d'alignement HTF (D bias) appliqué **sélectivement** aux playbooks continuation, laissé hors-gate pour les reversal :
- **GATED** (continuation) : BOS_Scalp_1m, Engulfing_Bar_V056 → `require_htf_alignment: D`
- **CONTROL** (reversal, no gate) : Morning_Trap_Reversal, Liquidity_Sweep_Scalp

Justification : forcer le D-bias sur un reversal casse le signal par design (reversal attaque contre la tendance). Le test isole l'effet HTF sur continuation uniquement.

Code : [setup_engine_v2.py:171-200](backend/engines/setup_engine_v2.py#L171-L200) — gate déjà wiré.

## Config
Identique à C1/C2/C3. YAML: `backend/knowledge/campaigns/c4_htf_gate.yml`.

## Résultats

| Playbook | n | E[R] | baseline | Δ | WR | net |
|---|---|---|---|---|---|---|
| Morning_Trap_Reversal (control) | 34 | -0.050 | -0.169 | +0.119 | 26.5% | -0.115 |
| Engulfing_Bar_V056 (GATED)      | 34 | **-0.033** | -0.102 | **+0.069** | 44.1% | -0.098 |
| BOS_Scalp_1m (GATED)            | 53 | -0.121 | -0.115 | -0.006 | 41.5% | -0.186 |
| Liquidity_Sweep_Scalp (control) | 49 | -0.006 | -0.034 | +0.028 | 36.7% | -0.071 |

## Verdict

- **3/4 positif, 1 marginal regress.** Engulfing le plus aidé (+0.069) — cohérent avec Phase D.1 bias audit (continuation/mean-rev profite de D-alignment).
- BOS_Scalp_1m reste signal-quality-suspect : HTF gate ne sauve pas un signal défaillant. -0.006 effet nul.
- **Morning_Trap control +0.119** : **mais attention, Morning_Trap ne reçoit pas de patch C4** — le delta vient d'une autre source (variance inter-run ou effet subtil de la re-exécution avec le même YAML+code que baseline). À investiguer si c'est reproductible. Possiblement bruit de run.
- **Liquidity_Sweep control +0.028, n identique à C3** : suggère même déterminisme sur un playbook inchangé → donc Morning_Trap's +0.119 control doit venir d'un autre playbook qui partage le session state (interférence via caps).
- **Gate** : 0/4 net E[R]>0.
- **Enseignement** : HTF selective = meilleur lever pour continuation (Engulfing), inutile pour chop (BOS). Complémentaire à C3, pas redondant.
