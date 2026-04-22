# EC audit — verdict (Aplus_03_v2, 4 semaines, caps actives)

## TL;DR

**`entry_confirm_no_commit` est PROTECTEUR, pas destructeur.** Les setups passés ont une peak_R et un hit_TP supérieurs aux rejetés sur toutes les métriques d'excursion. Relaxer le gate ajouterait majoritairement des perdants.

Ceci **contredit l'hypothèse initiale** de l'audit 2 ("gate tue 66-78% des setups clean") : les 66-78% tués sont effectivement des setups inférieurs.

## Protocole

- Instrumentation read-only ajoutée dans `backend/backtest/engine.py` : chaque setup atteignant le gate `entry_confirm` émet 1 record JSONL (rejected ou passed).
- 4 smokes 1-semaine Aplus_03_v2 (jun_w3, aug_w3, oct_w2, nov_w4), caps actives, kill switch actif, allowlist=Aplus_03_v2 seul.
- Replay post-hoc ([`replay_ec_audit.py`](../../scripts/replay_ec_audit.py)) : pour chaque setup, forward-walk 300min de 1m bars, calcule peak_R / mae_R / hit_TP / hit_SL / hit_0.5R / hit_1R.
- Priorité intrabar : SL-avant-TP sur même bar (convention engine).

## Résultats (n=59, 4 semaines mergées)

| Event | n | peak_R p50 | peak_R p80 | mae_R p20 | hit_TP % | hit_SL % | hit_0.5R % | hit_1R % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| **passed** | 14 | +0.611 | **+1.429** | -0.923 | **42.9** | 21.4 | 64.3 | **35.7** |
| **rejected_no_commit** | 45 | +0.390 | +0.761 | -1.076 | 28.9 | **53.3** | 42.2 | 6.7 |

### Lecture métrique par métrique

| Métrique | Passed | Rejected | Δ | Interprétation |
|---|---:|---:|---:|---|
| peak_R p50 | +0.611 | +0.390 | +0.221 | passed capture 22% R de plus à la médiane |
| peak_R p80 | +1.429 | +0.761 | +0.668 | tail-risk haussier : passed atteint 1.43R au p80 vs 0.76R |
| hit_TP % | 42.9% | 28.9% | +14.0pp | passed touchent TP 1.5× plus souvent |
| hit_SL % | 21.4% | 53.3% | −31.9pp | rejetés cognent le SL 2.5× plus souvent |
| hit_+1R % | 35.7% | 6.7% | +29.0pp | 5× plus de "vrais" winners chez passed |
| mae_R p20 | -0.923 | -1.076 | +0.153 | losers rejetés vont plus profond sous l'entry |

**Aucune métrique ne favorise les rejetés.** Le gate sépare correctement les bons des mauvais setups.

## Funnel par semaine

| Semaine | matches | setups_created | after_risk | attempted | opened | EC kill% |
|---|---:|---:|---:|---:|---:|---:|
| jun_w3 | 28 | — | — | 15 | 2 | 86.7% |
| aug_w3 | 42 | — | 15 | 6 | 3 | 50.0% |
| oct_w2 | 40 | 8 | 8 | 8 | 2 | 75.0% |
| nov_w4 | 64 | — | — | 30 | 7 | 76.7% |
| **total** | **174** | — | — | **59** | **14** | **76.3%** |

## Implication

**La question "gate destructeur vs protecteur" est tranchée : protecteur.** Pas de smoke nécessaire avec `require_close_above_trigger: false` — l'hypothèse E[R] croise zéro en relâchant le gate est **fausse** car les setups qu'on ajouterait sont structurellement pires sur peak_R et hit_TP.

### Réarrangement hiérarchie causes

Rappel audits :
1. ~~TP géométriquement inatteignable (10/14)~~
2. ~~entry_confirm tue 2/3 des Aplus~~ — **réfuté** par ce replay
3. scoring cassé (r²=0.006, négatif)
4. filter splits 13 candidats in-sample

**Nouvelle hiérarchie** :
1. **TP géométrie** (inchangé) — Aplus_03_v2 TIGHT (ratio 0.958) mais 73% fallback sur liquidity_draw = schéma sous-exercé. Pistes (plan séparé) : `min_rr_floor` plus bas, `lookback_bars` élargi, ou `swing_k9`.
2. **Signal quality intrinsèque** — même en gardant uniquement les BONS setups (passed = meilleure slice), E[R] reste négatif (-0.019). C'est le signal qui est faible, pas la sélection.
3. **Scoring** — à refondre plus tard, pas en pré-requis E[R]>0 car actuellement non-gateant.
4. **Filter splits** — à utiliser uniquement avec validation holdout (actuel = data-mining trap).

### Pourquoi ceci reframe le diagnostic

Si la gate est protectrice et que les passed ont déjà hit_TP=43% / hit_+1R=36%, le problème n'est pas "on jette trop". Le problème est :
- soit **TP trop ambitieux vs peak achieved** (si TP=2R mais seul 36% atteint 1R → need TP ~0.7-1.0R ou trailing)
- soit **signal faible** (seul 43% des meilleurs setups finissent gagnants — WR 43% avec TP 1R = E[R] ~+0.03 si losers=-1R, mais observed = -0.019, donc les losers moyennent pire qu'1R plein)

### Action proposée (à valider user)

Pas de nouveau smoke avant décision. 3 options sur la table :

**Option Alpha — TP calibration agressive Aplus_03_v2** : tester `min_rr_floor: 0.3` (vs 0.5) et `lookback_bars: 120` (vs 60) pour exercer davantage le schéma `liquidity_draw`. Objectif : faire descendre le % fallback de 73% vers < 40% et voir si les passed deviennent rentables.

**Option Beta — Aplus_01 Family A full (sweep+IFVG+breaker)** : tester si un signal à 3 confluences (vs IFVG isolé d'Aplus_03_v2) suffit à rendre le signal qualité suffisante. Plan séparé, existant (Family A gap identified Phase D.2).

**Option Gamma — Aplus_04_v2 Family B avec le schéma liquidity_draw** : 2e data point pour le schéma (Aplus_04_v1 était fixed_rr, peak_R p80=1.02R → TP 1.0R atteint 23.6%). Si Aplus_04_v2 en schema passe → schéma validé sur 2 familles. Si fail → schéma insuffisant.

## Artefacts

- Instrumentation : [backtest/engine.py](../../backtest/engine.py) (méthode `_emit_ec_audit`).
- Replay script : [scripts/replay_ec_audit.py](../../scripts/replay_ec_audit.py).
- JSONL bruts : `backend/results/labs/mini_week/aplus03_v2_ec_audit/*/ec_audit_*.jsonl`.
- Merged + replay : `backend/results/labs/mini_week/aplus03_v2_ec_audit/{ec_audit_4w_merged.jsonl,replay_ec_audit_4w_merged.{json,md}}`.

## Caveat honnête

- n=14 passed = petit. CI large sur les pourcentages (± ~20pp à 95%).
- Replay 300min horizon — suffit pour 5m playbooks mais peut tronquer winners très lents (≥5h).
- Convention intrabar SL-avant-TP = conservatrice (pénalise les passed sur même-bar hits). Les résultats passed sont donc un **plancher**.
- Aucun mécanisme de coût (slippage, fees) dans le replay — c'est excursion brute, pas r_multiple de trade fermé.
