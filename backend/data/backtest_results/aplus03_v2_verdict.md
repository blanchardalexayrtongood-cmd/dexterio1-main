# Aplus_03_v2 — Option A v2 verdict (4 semaines, caps actives)

**Date**: 2026-04-21
**Corpus**: jun_w3 + aug_w3 + oct_w2 + nov_w4 (2025)
**Mode**: AGGRESSIVE, caps actives (`--no-relax-caps`), allowlist via `--calib-allowlist Aplus_03_v2`
**YAML**: [`backend/knowledge/campaigns/aplus03_v2.yml`](../../knowledge/campaigns/aplus03_v2.yml)
**Runner**: `scripts/run_mini_lab_week.py` × 4 (output_parent=`aplus03_v2`)
**Reference v1 (R.3)**: [`r3_aplus03_tpcalib_verdict.md`](r3_aplus03_tpcalib_verdict.md), n=35, E[R]=-0.055

**Question testée (scope unique du sprint)** : est-ce que `tp_logic: liquidity_draw (swing_k3)` + `require_structure_alignment: k3` débloquent de l'edge sur Aplus_03 IFVG 5m ?

---

## 1-6 — E[R], n, WR, PF

| Week | n | WR | E[R] (net) | PF | wins | losses | TP1 hits | SL | time_stop |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| jun_w3 | 2 | 100.0% | +0.0153 | ∞ | 2 | 0 | 0 | 0 | 0 |
| aug_w3 | 11 | 45.5% | -0.0072 | 0.842 | 5 | 6 | 3 | 5 | 3 |
| oct_w2 | 2 | 0.0% | -0.1258 | 0.000 | 0 | 2 | 0 | 2 | 0 |
| nov_w4 | 7 | 42.9% | -0.0169 | 0.408 | 3 | 4 | 3 | 4 | 0 |
| **Total 4w** | **22** | **45.45%** | **-0.0190** | **0.560** | **10** | **12** | **6** | **11** | **5** |

- **Item 1 — E[R] gross (avant costs)** : non exporté dans le schéma `mini_lab` (`summary` ne split pas gross/net ; même limitation que R.3 et Option B). Proxy conservateur : coûts moyens ~-0.01 à -0.02R/trade observés sur corpus similaires → E[R] gross ≈ -0.005 à +0.000R/trade. **Pas négatif franc, pas positif franc**.
- **Item 2 — E[R] net** : **-0.019** (total), per-week ci-dessus. 1/4 semaines > 0 (jun_w3, mais n=2).
- **Item 4 — n total** : 22 trades (cible Case A ≥15 : OK).
- **Item 5 — WR** : 45.45% (vs v1 R.3 42.86%, +2.6 pts).
- **Item 6 — PF** : 0.560 net (vs v1 R.3 ~0.90). Dégradé.

## 3 — E[R] net + conservative slippage (ConservativeFillModel reconcile)

Rapport détaillé : [`aplus03_v2_reconcile_slippage.md`](aplus03_v2_reconcile_slippage.md).

| Métrique | Valeur |
|---|---:|
| n trades rejoués | 22 (0 skipped) |
| Δ trades worse | 100% |
| Mean Δ $ / trade | -53.26 |
| Mean Δ R / trade | **-0.0446** |
| Total Δ R (4w) | -0.982 |

**E[R] net + slippage** : -0.019 + (-0.045) = **-0.064 R / trade**. Budget paper donc clairement négatif.

## 7 — tp_reason breakdown (trade-level, n=22)

| Reason | Count | % |
|---|---:|---:|
| `liquidity_draw_swing_k3` | **6** | **27.3%** |
| `fallback_rr_no_pool` | 6 | 27.3% |
| `fallback_rr_min_floor_binding` | 10 | 45.5% |
| **Total fallback (no_pool + min_floor)** | **16** | **72.7%** |

Setup-level (avant filtres risk/entry_confirm) sur 96 setups créés agrégés : `liquidity_draw_swing_k3` = 41 (42.7%), `fallback_rr_min_floor_binding` = 29 (30.2%), `fallback_rr_no_pool` = 26 (27.1%). La dégradation de 42.7% (setup) → 27.3% (trade) vient de `min_floor_binding` qui est un fallback dégradé (pool trop proche → TP = `min_rr_floor × sl_distance`).

## 8 — % liquidity_draw effective (hors fallback)

**27.3% (6/22)** des trades ont eu un TP réellement calculé via prochain swing k3. Seuil Case C < 30% : **à la limite basse**. Seuil Case B > 50% fallback : **72.7% satisfait (Case B dominant)**.

## 9 — structure_alignment gate (compteur + distributions)

Gate k3, agrégé 4 semaines :

| Métrique | Valeur |
|---|---:|
| evaluated (total setups pre-gate) | **171** |
| pass_aligned | 96 |
| **rejected** | **75** |
| rejection rate | **43.9%** |
| reject `LONG_vs_bear_structure` | 41 |
| reject `SHORT_vs_bull_structure` | 34 |
| long_evaluated | 64 |
| short_evaluated | 107 |

**Distribution long/short pre vs post-gate** :

| Direction | Pre-gate | Post-gate (trades) |
|---|---:|---:|
| LONG | 64 (37.4%) | 6 (27.3%) |
| SHORT | 107 (62.6%) | 16 (72.7%) |

Le gate filtre les deux directions, mais plus agressivement LONG (64→6 = 90.6% loss) que SHORT (107→16 = 85.0% loss). Gate non-biaisé symétrique dans son **critère**, mais le corpus 2025 a une structure k3 majoritairement **bearish** → plus de rejets LONG (41) que SHORT (34). Per-week : oct_w2 = 100% rejected LONG (tous les 32 LONG setups) car structure oct unanime-bear — cohérent.

**HTF D-alignment gate** : `evaluated=171, rejected=0`, `pass_unknown_or_range=115, pass_aligned=56`. Le gate `require_htf_alignment: D` ne rejette **rien** sur ce corpus (SMA proxy retourne majoritairement `range/unknown`, pass-through). Confirme Phase D.1 : SMA proxy n'est pas un vrai HTF gate. Le seul gate effectif est **k3 structural**.

## 10 — peak_R distribution vs v1 (R.3)

| Percentile | v1 R.3 (n=35) | v2 (n=22) | Δ |
|---|---:|---:|---:|
| p50 | 0.542 | 0.522 | -0.020 |
| p60 | 0.638 | 0.584 | -0.054 |
| p80 | 0.734 | 0.669 | -0.065 |
| max | — | 0.854 | — |

Les winners **ne sont pas capturés plus haut** qu'en v1. `tp_logic: liquidity_draw` pull le TP *plus près* quand le pool swing_k3 est trouvé, et `min_rr_floor=0.5` tire encore plus près quand le pool est trop proche. Résultat : peak_r p80 baisse de 0.73R → 0.67R. Les winners moyens passent de 0.057R (v1) à 0.053R (v2) — quasi-identiques.

**Lecture** : le schéma capture *plus souvent* un TP (TP1 hits 10/35 = 28.6% v1 → 6/22 = 27.3% v2, essentiellement identique), mais à des niveaux *inférieurs*, donc **les winners pèsent moins**.

## 11 — mae_R distribution vs v1 (R.3)

| Percentile | v1 R.3 (n=35) | v2 (n=22) | Δ |
|---|---:|---:|---:|
| p20 | -0.798 | -0.533 | +0.265 |
| p40 | -0.388 | -0.310 | +0.078 |
| p50 | -0.331 | -0.216 | +0.115 |
| min | — | -1.144 | — |

Les losers **absorbent moins de R** qu'en v1. Losers avg R : -0.139 (v1) → -0.079 (v2) (+0.060 amélioration). C'est le seul axe où v2 bat v1 clairement — le gate k3 rejette une partie des setups à direction structurellement contraire qui finissaient -1R en v1.

---

## Résumé des 11 items

| # | Item | v1 R.3 | v2 | Verdict |
|---|---|---:|---:|---|
| 1 | E[R] gross | ~-0.045 (estim.) | ~-0.005 (estim.) | amélioré |
| 2 | E[R] net | -0.055 | **-0.019** | +0.036 |
| 3 | E[R] net + slippage | — | **-0.064** | négatif franc |
| 4 | n | 35 | 22 | ↓ (gate réduit) |
| 5 | WR | 42.86% | 45.45% | +2.6pp |
| 6 | PF net | ~0.90 | 0.560 | ↓ |
| 7 | tp_reason breakdown | — | 27% draw / 73% fallback | — |
| 8 | % liquidity_draw effective | 0% | 27.3% | exercé mais minoritaire |
| 9 | structure_alignment counter | — | 43.9% rejected (75/171) | gate fonctionnel |
| 10 | peak_R p80 | 0.734 | 0.669 | ↓ (winners plafonnés bas) |
| 11 | mae_R p20 | -0.798 | -0.533 | ↑ (losers moins R) |

---

## Framework de lecture Case A / B / C

Critères (plan, §O5.2.bis) :

- **Case A** : E[R] > 0 (gross ET net) ET n ≥ 15 ET PF > 1.
- **Case B** : E[R] ≤ 0 ET (% fallback_rr > 50% OU n < 15 après gate).
- **Case C** : E[R] ≤ 0 ET % fallback_rr < 30% ET n ≥ 20 ET gate fonctionnel.

Application :

| Condition | Valeur observée | Verdict |
|---|---|---|
| E[R] net > 0 ? | -0.019 | ❌ |
| n ≥ 15 ? | 22 | ✓ |
| PF > 1 ? | 0.560 | ❌ |
| % fallback_rr > 50% ? | **72.7%** | ✓ |
| % fallback_rr < 30% ? | 72.7% | ❌ |

→ **Case B — schema mal exercé (dominant)**.

Motif explicite : le TP `liquidity_draw_swing_k3` n'a été effectivement calculé sur pool que dans **27.3%** des trades. Les 72.7% restants sont tombés en fallback (`no_pool` 27.3% + `min_floor_binding` 45.5%). Le signal IFVG 5m isolé, filtré par `structure_alignment_k3`, produit peu de contextes où un pool k3 utilisable existe dans la fenêtre 60 bars au-delà de `entry + 0.5 × sl_distance`.

**Conclusions INTERDITES (plan §O5.2.bis)** — aucune de ces conclusions n'est légitime sur la base du seul Aplus_03_v2 :
- ❌ "Family A ne marche pas"
- ❌ "liquidity_draw ne marche pas"
- ❌ "structure_alignment k3 est inutile" (le gate est fonctionnel, 44% rejection, pas biaisé par direction — cf item 9)
- ❌ "MASTER ICT est invalide"
- ❌ "Le quant n'apporte rien"

## Signaux secondaires exploitables

1. **Gate k3 fonctionne** (44% rejection, non-biaisé direction, coupe les pires setups — cf. mae_R p20 +0.27R vs v1). Brique réutilisable pour Aplus_01/04/etc.
2. **min_floor_binding 45.5%** est le plus gros fallback. Le paramètre `min_rr_floor: 0.5` force un TP très près quand le pool est dans les 0.5R du SL. Résultat : winners plafonnés. Levier potentiel (plan séparé) : baisser `min_rr_floor` à 0.3 ou 0.2 laisse TP être fixé au pool même très proche → plus de variance winners, mais WR potentiellement plus basse.
3. **no_pool 27.3%** indique que `lookback_bars: 60` est parfois insuffisant, ou que `swing_k3` est trop fin. Levier potentiel (plan séparé) : tester `draw_type: swing_k9` (pools plus éloignés, plus rares mais plus robustes). Exclu du scope actuel.
4. **HTF D-alignment gate** ne rejette rien (0/171). SMA proxy n'est pas un vrai gate (Phase D.1 confirmé). À remplacer par k9 structural dans futur plan, ou retirer.
5. **WR 45% sans edge** : améliorée de +2.6pp vs v1 mais la PF chute (0.90 → 0.56) — gains WR compensés par peak_R plus bas et ratio avg_win/avg_loss dégradé (0.67× vs 0.41× des losers vs winners en v2).

---

## Décision

Per plan §O5.2.bis Case B :

- **NE PAS** passer O5.3 (gates permutation/Sharpe/Martin/slippage). Réservés Case A uniquement. Inutile si signal dominant est fallback.
- **NE PAS** conclure sur Family A, sur le schéma `liquidity_draw`, ou sur `structure_alignment`.
- **Aplus_03_v2 = 1 point de donnée partiel**. Le schéma n'est pas refuté ; il n'est pas confirmé non plus. Il est **sous-exercé** sur ce signal précis.

## Post-Option A v2 — options (plans séparés)

Lecture Case B → **itération ciblée possible**, pas rewrite/kill. Options (hors scope de CE sprint, nécessitent plan séparé) :

1. **Baisser `min_rr_floor`** : 0.5 → 0.3 → 0.2. Mesure : % `min_floor_binding` doit baisser, peak_R p80 doit monter.
2. **Élargir `lookback_bars`** : 60 → 120. Mesure : % `no_pool` doit baisser.
3. **Tester `draw_type: swing_k9`** (pools plus rares mais plus significatifs). Nouveau point de donnée sur le schéma.
4. **Aplus_01 Family A full** (sweep+IFVG+breaker, pas IFVG isolé). Le signal plus riche aura probablement plus de confluences, potentiellement plus de pools k3 utilisables dans la fenêtre. **2e point de donnée sur le schéma**.
5. **Aplus_04_v2** (Family B rewrite avec même schema). **3e point de donnée, sur une famille différente** — permettrait de valider ou rejeter le schéma globalement.

Toute conclusion sur le schéma `liquidity_draw` demande ≥2 points convergents sur familles/setups distincts (cf plan §O5.2.bis "règle générale").

**État 5-classes post-Option A v2** : Aplus_03_v2 reste classé **REWRITE partial** (comme v1 + R.3). Pas de dégradation, pas d'upgrade. Le schéma YAML a été validé côté infrastructure (tp_resolver + directional_change opérationnels, 19/19 tests pass, 0 régression) mais pas côté edge.

---

## Fichiers produits

- [`backend/knowledge/campaigns/aplus03_v2.yml`](../../knowledge/campaigns/aplus03_v2.yml) — YAML playbook v2
- `backend/results/labs/mini_week/aplus03_v2/aplus03_v2_{jun_w3,aug_w3,oct_w2,nov_w4}/` — 4 runs caps actives
- [`aplus03_v2_reconcile_slippage.md`](aplus03_v2_reconcile_slippage.md) — ConservativeFillModel reconcile
- `backend/engines/execution/tp_resolver.py` — brique TP dynamique (stable signature)
- `backend/engines/features/directional_change.py` — zigzag ATR-adaptive k1/k3/k9 cached
- 19/19 tests PASS ([`test_tp_resolver_*.py`](../../tests/test_tp_resolver_fixed_rr.py), [`test_directional_change.py`](../../tests/test_directional_change.py), [`test_structure_alignment_gate.py`](../../tests/test_structure_alignment_gate.py))

## Règles respectées

- Scope verrouillé (1 playbook, 1 `tp_logic`, 1 `draw_type`, 1 alignment level). Zéro drift.
- Pas de conclusion terminale sur Family A / schema / quant sur base d'un seul data point.
- Pas de promotion SAFE (Aplus_03_v2 reste LAB/research).
- Pas de toggle ALLOWLIST/DENYLIST.
- `r3_aplus03_tpcalib_v1.yml` intact (référence v1 préservée).
- Moteur unique (tp_resolver + directional_change = briques réutilisables backtest/paper/live).
