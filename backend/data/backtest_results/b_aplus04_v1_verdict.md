# Aplus_04_HTF_15m_BOS_v1 — verdict Option B (2026-04-21)

## Protocole

- **Playbook** : `Aplus_04_HTF_15m_BOS_v1` (Family B MASTER: D/4H bias + 15m BOS + 1m entry on pullback).
- **Overlay** : [b_aplus04_v1.yml](backend/knowledge/campaigns/b_aplus04_v1.yml) — copie de `playbooks.yml` + nouveau playbook basé sur `HTF_Bias_15m_BOS` (denylisté), avec contraintes relâchées et TP calibré bas :
  - `setup_tf: 15m` ✓ (supporté schema actuel)
  - `max_setups_per_session: 3` (vs 1 sur le template)
  - `conditions.pattern_confirmations: 1` (vs 2)
  - `context_requirements.adx_min: null` (vs 20)
  - `require_htf_alignment: D` (D-bias obligatoire)
  - `require_close_above_trigger: true` + `entry_buffer_bps: 2.0` (recette S1)
  - TP calibré : `tp1_rr: 1.0`, `tp2_rr: 2.0`, `breakeven_at_rr: 0.5`, `trailing_trigger_rr: 0.7`, `trailing_offset_rr: 0.3`
- **Conditions** : 4 semaines (jun_w3 + aug_w3 + oct_w2 + nov_w4), SPY+QQQ, caps actives, allowlist restreinte à Aplus_04_HTF_15m_BOS_v1.

## Résultats — 0/4 weeks positives

| Week | n | E[R] | WR | PF | sumR | MaxDD_R |
|---|---:|---:|---:|---:|---:|---:|
| jun_w3 | 12 | -0.075 | 41.7% | 0.408 | -0.897 | 2.35 |
| aug_w3 | 15 | -0.074 | 60.0% | 0.446 | -1.108 | 1.81 |
| oct_w2 | 13 | -0.099 | 30.8% | 0.078 | -1.282 | 1.31 |
| nov_w4 | 15 | -0.050 | 53.3% | 0.557 | -0.756 | 1.37 |
| **Total** | **55** | **-0.074** | **47.3%** | **0.389** | **-4.043** | — |

Net après slippage (-0.065R/trade) : **E[R] net = -0.139**.

## Lecture mécaniste

Distribution exits : 41 SL / 13 TP1 / 1 eod.

**peak_R distribution** :
| seuil | hits | % |
|---|---:|---:|
| ≥ 0.3R | 41/55 | 74.5% |
| ≥ 0.5R | 35/55 | 63.6% |
| ≥ 0.7R | 29/55 | 52.7% |
| **≥ 1.0R (TP1 level)** | **13/55** | **23.6%** |
| ≥ 1.5R | 1/55 | 1.8% |

**MAE (loser SL cleanliness)** :
- |mae_r| p50 = 1.02R, p75 = 1.07R → SL loss clean à ~1R (SWING detector fonctionne).

**avg winner R = +0.099, avg loser R = -0.228.**

## Interprétation

1. **Signal produit du mouvement mais borné** : peak_R p50 = 0.71R, p80 = 1.02R. TP1 à 1.0R = marginal pour 23% des trades seulement.

2. **Pathologie identique à R.3 (Aplus_03)** :
   - R.3 : peak_R p80 = 0.73R, TP1 0.70R atteint 10 fois, E[R] = -0.055.
   - Aplus_04 v1 : peak_R p80 = 1.02R, TP1 1.0R atteint 13 fois, E[R] = -0.074.
   - **Les winners saturent au niveau TP fixé, les losers consomment 1R plein via SL swing**. Ratio gross_loss/gross_profit = 6.61/2.57 = **2.57×**.

3. **Ce que la calibration TP ne peut pas corriger** : dans Aplus_04 v1 comme dans R.3, descendre TP1 encore plus bas (ex 0.50R) augmenterait le n mais les winners 0.5R × WR ne couvriraient jamais les losers 1R × (1-WR) sauf si WR > 67%. WR observé = 47%, jamais vu > 60% sur 4 semaines. **Mathématique impossible à 1:1 ou pire**.

4. **HTF D-bias alignment n'a pas sauvé** : le gate `require_htf_alignment: D` est censé n'autoriser que les setups dans le sens de la tendance journalière. Phase D.1 bias audit avait déjà montré que D-alignment seule n'unlock pas d'edge (-0.068 aligned vs -0.108 counter, ne croise pas zéro). Aplus_04 v1 le confirme à n=55.

## Verdict gate Option B

**❌ FAIL** : 0/4 weeks E[R] > 0, total E[R] = -0.074 / net -0.139.

Ne croise pas le seuil product-grade ni la neutralité.

## Conséquence — Option A devient justifiée repo-backed

Ce résultat ferme le bear case sur les Family B/F en l'état du schema YAML actuel :

- **Family A complète** (Aplus_01 Sweep+IFVG+Breaker) requiert `tp_logic: liquidity_draw` (absent).
- **Family B minimale** (Aplus_04 v1 testée ici) : E[R]=-0.074 / net -0.139 sans liquidity-draw TP.
- **Family F** (Aplus_02 Premarket Sweep) requiert premarket session context (absent).

**Les 3 Families MASTER non-testées partagent toutes la même dépendance au schema** :
1. `tp_logic: liquidity_draw` — TP au prochain session high/low, PAS fixed RR.
2. Premarket session context — nécessaire pour Aplus_02.
3. 1m confirm-in-zone state machine — entry précis post-IFVG.

Aplus_04 v1 testée en schema actuel prouve : **fixed RR TP est structurellement inadapté à ces signaux MASTER**. La pathologie est identique à Aplus_03 R.3. 2 data points convergents → **pattern confirmé**.

## Reclassification post-Option B

**Aplus_04_HTF_15m_BOS_v1 → REWRITE partial** (même diagnostic que Aplus_03_IFVG_Flip_5m R.3).

Raisons :
- Signal génère peak_R exploitable (p50 0.71R, p80 1.02R) — **pas null**.
- Mais TP fixed RR @ n'importe quelle valeur ne produit pas E[R] > 0 (winners cappés, losers full 1R).
- Requis : `tp_logic: liquidity_draw` pour que winners capturent draw complet (session high/low) au lieu du TP rigide.

## Prochaine étape proposée

**Option A — Schema YAML extension** devient la seule voie productive pour Family A/B/F :

1. Ajouter `tp_logic: liquidity_draw` au loader (accepter, stocker).
2. Implémenter lookup liquidity pools dans `setup_engine_v2` / `take_profit_logic` — mapper chaque setup au nearest session high (SHORT) / low (LONG) qui n'a pas été tagué/balayé.
3. Test unitaire : un setup LONG avec swing low à -1R doit produire TP = session high le plus proche au-dessus de entry (pas un fixed 2R).
4. Re-tester Aplus_04_v2 (même playbook mais TP = liquidity_draw) et Aplus_03 R.4 en parallèle.
5. Si ≥1 franchit net E[R] > 0 → **1er vrai MASTER validé**, schema extension payoff.
6. Si 0/2 → la problème n'est pas le TP, c'est le signal. Family A/B n'ont pas d'edge même en TP idéal. Pivot terminal ou Polygon 18m.

**Coût estimé Option A** : 1-2 jours engine work + 2 re-runs 4-semaines.

## Règles absolues (post-B)

- Ne plus investir de calibration sur Aplus_04_v1 ou Aplus_03_R.3 en schema actuel. 2 data points = TP fixed RR est structurellement inadapté.
- Ne pas créer Aplus_01/02 sans schema `tp_logic: liquidity_draw`.
- Maintenir règle n ≥ 30 ET cross-weeks ≥ 3/4 avant promotion.
