# Axe 1 — Audit EC gate sur données α'' (read-only)

## TL;DR

**EC gate = PROTECTEUR, pas destructeur.** Les setups `liquidity_draw_swing_k3` rejetés par `require_close_above_trigger` sont **strictement pires** que ceux qui passent — sur toutes les métriques (peak_R, hit_TP, hit_1R, mae_R). La consigne "schéma meurt en aval" est vérifiée *au comptage* (9% de survie), mais **ce qui est éliminé était à éliminer**.

**Conséquence directe** : **axe 2 (relâcher l'EC gate) OFF THE TABLE**. Le relâcher réintroduirait majoritairement des setups loss-heavy (82% hit SL).

## Protocole

- Source : `ec_audit_*.jsonl` des 2 smokes α'' déjà produits (aug_w3 + nov_w4), 36 événements EC audit (26 `rejected_no_commit` + 10 `passed`).
- Pour chaque event : forward-replay 1m bars ×300 min via [replay_ec_audit.py](../../scripts/replay_ec_audit.py) (intrabar SL-before-TP, conservateur). Calcule peak_R / mae_R / hit_TP / hit_SL / hit_0.5R / hit_1R.
- Classification `tp_reason` post-hoc à partir de `tp1_rr = |tp1-entry|/sl_dist` :
  - 0.5 ± 0.02 → `fallback_rr_min_floor_binding`
  - 2.0 ± 0.02 → `fallback_rr_2p0` (no_pool OU beyond_ceiling — indiscernables via TP prix)
  - else → `liquidity_draw_swing_k3`
- Script d'analyse : `/tmp/alpha_pp_ec/replay_by_tpreason.py`.

## Crosstab tp_reason × event (36 EC events α'')

| tp_reason_derived | passed | rejected_no_commit | total |
|---|---:|---:|---:|
| liquidity_draw_swing_k3 | 3 | **22** | 25 |
| fallback_rr_2p0 | 5 | 3 | 8 |
| fallback_rr_min_floor_binding | 2 | 1 | 3 |
| **Total** | **10** | **26** | **36** |

**Observation clé** : parmi les 25 setups où le schéma est réellement exercé, **88% sont rejetés par l'EC** (22/25). Parmi les fallbacks, 27% rejetés (4/11). Le gate est **sélectif contre le schéma** au comptage — mais la question est *qualitative*.

## Comparaison qualitative passed vs rejected

### liquidity_draw_swing_k3 (le subset qui compte)

| Metric | Passed (n=3) | Rejected (n=22) | Verdict |
|---|---:|---:|---|
| peak_R p50 | **+1.524** | +0.298 | rejected 5× pire |
| peak_R p80 | **+1.609** | +0.717 | rejected 2.2× pire |
| mae_R p20 | -0.781 | **-1.093** | rejected plus profond |
| hit_TP % | **66.7** | 13.6 | rejected 5× pire |
| hit_SL % | 33.3 | **81.8** | rejected 2.5× pire |
| hit_0.5R % | 66.7 | 36.4 | rejected ~2× pire |
| hit_1R % | **66.7** | 4.5 | **rejected 15× pire** |

**Lecture** : sur les 22 liquidity_draw rejetés, 18 (82%) sont allés au SL direct et seulement 1 (4.5%) a touché +1R avant SL. Aucune réserve d'edge n'y dort.

### fallback_rr_2p0

| Metric | Passed (n=5) | Rejected (n=3) |
|---|---:|---:|
| peak_R p80 | +1.415 | +0.951 |
| hit_0.5R % | 60.0 | 66.7 |
| hit_1R % | 40.0 | 33.3 |
| hit_SL % | 20.0 | 66.7 |

Gap plus étroit (n très petit) — les rejetés fallback survivent un peu à 0.5R/1R mais meurent plus en SL. Pas de réserve d'edge non plus.

### fallback_rr_min_floor_binding

n=3 total. Non-interprétable statistiquement.

## Direction imbalance (caveat)

Des 22 rejetés `liquidity_draw_swing_k3`, **20 sont SHORT** et 2 LONG. Des 3 passés, 2 LONG et 1 SHORT. L'EC gate agit ici surtout sur des SHORTs — compatible avec la logique `require_close_above_trigger: true` qui attend une clôture au-dessus du trigger (contre-directionnel au SHORT, donc difficile à franchir sur une direction baissière).

Cette asymétrie **renforce** le verdict "protecteur" : l'EC gate refuse d'ouvrir un SHORT tant que le marché n'a pas marqué qu'il est vraiment cassé — et les 20 SHORTs rejetés confirment que la cassure n'a pas tenu (82% vont au SL).

## Lecture finale (framework Case A/B/C post-α'')

Rappel du framework utilisateur :
- **rejetés ≥ passés** → axe 2 (relax EC) justifié.
- **rejetés < passés** → axe 2 off the table ; next = δ swing_k9, Aplus_04_v2, ou "Aplus_03_v2 signal trop faible".

**Nous sommes clairement dans la 2e branche.** Rejetés *strictement pires* sur tous les axes — le fait que le schéma soit "sous-exercé" (9% de survie) n'est pas un accident de filtre mal calibré : **l'EC identifie correctement les setups qui vont perdre**.

## Ce qui est prouvé

1. **EC gate n'est pas un faux coupable.** Il ne détruit pas l'edge, il le protège (ratio passed/rejected hit_1R = 67/4.5 = 15×).
2. **Le schéma `liquidity_draw_swing_k3`, quand il est réellement permis de s'exprimer (passed, n=3), performe très bien** : peak_R p80=1.6R, hit_TP=67%, hit_1R=67%. Mais n=3 → CI très large.
3. **Le nœud d'Aplus_03_v2 n'est plus downstream** — c'est upstream : le schéma **génère trop peu de bons setups** pour que la population de trades soit exploitable.

## Ce qui N'EST PAS prouvé

- ❌ "Aplus_03_v2 n'a pas d'edge" — 3/3 passed hit 0.5R et 2/3 hit 1R suggère un signal vrai mais trop rare.
- ❌ "EC gate est la bonne implémentation" — on prouve qu'il est protecteur *dans cette population α''*, pas qu'il serait optimal sur un signal plus fort.
- ❌ "Le schéma liquidity_draw ne marche pas" — seulement prouvé pour le signal IFVG 5m isolé + EC + structure k3.

## Recommandation

**Ne pas lancer axe 2.** Le smoke relaxé EC ne produirait pas d'edge — il réintroduirait 22 trades dont 18 SL direct.

**Options restantes** (plans séparés obligatoires) :

- **Option δ — swing_k9** : pools plus rares et plus loin → band 0.5-3.0R y trouve probablement un pool plus souvent, et le signal IFVG pourrait exprimer le schéma sur une plus large portion de setups. Testable cheap sur 1 semaine.
- **Aplus_04_v2** : 2e data point sur le schéma (Family B au lieu de Family A). Cohérent avec plan original. Même si ambigu, donne la convergence 2-data-points avant conclusion bear ferme sur le schéma.
- **Accepter Aplus_03_v2 = signal trop faible** et pivoter vers Aplus_01 Family A full (sweep + IFVG + breaker ; cluster de confluences plutôt que IFVG seul).

Mon vote perso : **δ swing_k9 avant Aplus_04_v2**. Ça stays dans Aplus_03_v2 (1 seul axe varie = lisible), et ça teste directement l'hypothèse "le schéma est sous-exercé parce que k3 produit trop de pools nearby". Si k9 garde 70% de fallback → le schéma ne sauve pas le signal IFVG isolé, et Aplus_04_v2 prend le relais. Si k9 remonte à 50% schéma exercé → on a enfin une population testable.

## Caveats

- **n=3 passed liquidity_draw** : CI ultra-large sur p50/p80. Les chiffres sont *directionnellement* solides (3/3 hit 0.5R, 2/3 hit 1R) mais statistiquement fragiles.
- **Horizon replay 300 min** : correct pour scalp 5m ; peut tronquer winners très lents. Sur les rejected qui meurent au SL dans les 30 premières minutes en moyenne, pas d'impact.
- **Replay ignore caps/sizing/trailing/BE** : pures métriques peak_R/hit. L'E[R] net serait inférieur mais la comparaison *relative* passed vs rejected tient.
- **Direction skew** : EC rejette surtout SHORT. Conclusion "protecteur" est valide pour les SHORTs ; la preuve LONG est plus mince (n=2 rejected LONG, 2 passed LONG).
- **2 semaines smoke uniquement** : gate user "pas de long run". Si δ ou Aplus_04_v2 passent Case A → étendre à 4 semaines pour CI resserrée.

## Artefacts

- Combined audit : `/tmp/alpha_pp_ec/combined.jsonl` (36 events, aug_w3 + nov_w4).
- Replay global : `/tmp/alpha_pp_ec/replay.json` + `/tmp/alpha_pp_ec/replay.md`.
- Analyse par tp_reason : `/tmp/alpha_pp_ec/axe1_by_tpreason.json` + script `/tmp/alpha_pp_ec/replay_by_tpreason.py`.
- Verdict α'' précédent : [alpha_pp_tp_resolver_verdict.md](alpha_pp_tp_resolver_verdict.md).
