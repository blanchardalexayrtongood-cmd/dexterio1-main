# Option α'' — tp_resolver "significant pool" : verdict PARTIAL FIX

## TL;DR

**Le resolver est sémantiquement corrigé mais le trade-level E[R] ne bouge pas.** Le fix `pool_selection: significant` + `max_rr_ceiling: 3.0` déplace 5-7% des setups vers de vraies "significant pools" au lieu de pools clippés au floor — mais les setups qui survivent au gate `entry_confirm` sont ceux où "farthest-in-band" = "nearest" (pool unique ou absent), donc trade-level outcomes = byte-identiques à baseline.

**Observation trade-level** : les setups survivent rarement jusqu'au trade quand ils exercent le schéma (3/32 = 9% de survie) vs fallback (7/28 = 25%). Filtre downstream (EC gate dominant + caps + position-sizing) plus restrictif sur les `liquidity_draw_swing_k3` setups — cause exacte à isoler (plan séparé).

**Verdict** : α'' est un fix correct et réutilisable (`pool_selection: nearest` vs `significant`), mais **ne débloque pas le Case B d'Aplus_03_v2**. Le problème n'était pas que le nearest-pool produisait des mauvaises TP — c'est que l'EC gate tue les trades qui auraient bénéficié du fix.

## Protocole

- Modif [tp_resolver.py](../../engines/execution/tp_resolver.py) : `pool_selection` ∈ {`nearest` (default), `significant`}. Mode `significant` filtre les pivots dans la band `[min_rr_floor, max_rr_ceiling]` et pick **farthest** (au lieu de nearest).
- Nouveau reason : `fallback_rr_pool_beyond_ceiling` quand tous les pools sont au-delà du ceiling.
- Overlay [aplus03_v2_alpha_pp.yml](../../knowledge/campaigns/aplus03_v2_alpha_pp.yml) : seules diffs vs v2 baseline = `pool_selection: significant` + `max_rr_ceiling: 3.0`. Lookback 60, min_rr_floor 0.5 INCHANGÉS (isole 1 axe).
- 2 smokes 1-semaine caps actives, allowlist=Aplus_03_v2 seul : aug_w3 + nov_w4.
- Tests : 17 cases PASS (10 legacy backcompat + 7 nouveaux significant mode).

## Résultats setup-level (ce qui change)

**aug_w3** (15 setups):
| tp_reason | Baseline (nearest) | α'' (significant) | Δ |
|---|---:|---:|---:|
| liquidity_draw_swing_k3 | 9 | 8 | -1 |
| fallback_rr_no_pool | 5 | 5 | 0 |
| fallback_rr_min_floor_binding | 1 | 1 | 0 |
| fallback_rr_pool_beyond_ceiling | 0 | 1 | +1 |

**nov_w4** (47 setups):
| tp_reason | Baseline | α'' | Δ |
|---|---:|---:|---:|
| liquidity_draw_swing_k3 | 21 | **24** | **+3** |
| fallback_rr_no_pool | 15 | 15 | 0 |
| fallback_rr_min_floor_binding | 11 | **7** | **-4** |
| fallback_rr_pool_beyond_ceiling | 0 | 1 | +1 |

**Aggregate (62 setups)** : liquidity_draw **30 → 32** (+2), min_floor **12 → 8** (-4), beyond_ceiling **0 → 2** (+2). Le fix fait exactement ce qu'il prétend — 4 setups qui étaient "clippés au floor" sous nearest exposent maintenant un pool significatif plus loin, et 2 setups où tous les pools étaient au-delà de 3R tombent proprement en fallback (au lieu de picker un pool hors band).

## Résultats trade-level (ce qui ne change pas)

**aug_w3** (n=3, identique des 2 côtés) :
| Metric | Baseline | α'' | Δ |
|---|---:|---:|---:|
| n | 3 | 3 | 0 |
| WR | 100% | 100% | 0 |
| E[R] | +0.026 | +0.026 | 0 |
| tp_reason | 3× no_pool | 3× no_pool | 0 |

**nov_w4** (n=7, 6/7 byte-identiques, 1 setup reclassé) :
| Metric | Baseline | α'' | Δ |
|---|---:|---:|---:|
| n | 7 | 7 | 0 |
| WR | 42.9% | 42.9% | 0 |
| E[R] | -0.017 | -0.017 | 0 |
| PF | 0.41 | 0.41 | 0 |

Le trade #6 nov_w4 (SHORT @ 597.17, 2025-11-19 17:44) :
- Baseline : TP=593.561 (0.5R floor binding), reason=`fallback_rr_min_floor_binding`
- α'' : TP=593.050 (pool réel à ~0.57R), reason=`liquidity_draw_swing_k3`
- Les 2 exitent au `time_stop` avec peak_r=0.166 → R-multiple identique (-0.0206)

**Aggregate 2-week trade outcomes** : n=10, E[R]≈-0.004 — identique baseline et α''.

## Lecture : pourquoi le setup-level change mais pas le trade-level

Cross-tabulation `tp_reason` × `passed → trade` sur les 2 semaines :

| tp_reason | setups α'' | setups devenus trades | taux survie downstream |
|---|---:|---:|---:|
| liquidity_draw_swing_k3 | 32 | 3 | **9%** |
| fallback_rr_min_floor_binding | 8 | 2 | 25% |
| fallback_rr_no_pool | 20 | 5 | 25% |
| fallback_rr_pool_beyond_ceiling | 2 | 0 | 0% |

Les setups `liquidity_draw_swing_k3` survivent ~3× moins souvent au pipeline downstream (EC gate + caps + position-sizing) que les fallback. C'est pour ça que fixer nearest→significant n'a **aucun effet observable à l'exécution** : les trades qui auraient bénéficié du fix ne fire majoritairement jamais.

**Cause exacte non-isolée ici** : l'EC gate est le filtre dominant (audit 4w 2026-04-20 montrait 76% kill rate global sur Aplus_03_v2), mais caps et position-sizing aussi peuvent jouer. Un audit EC ciblé sur α'' (axe 1 ci-dessous) trancherait.

## Case A/B/C framework (per plan O5.2.bis)

**Non applicable telle quelle** — α'' n'est pas une itération qui teste le schéma. C'est une correction du resolver pour que le schéma, quand il est utilisé, soit correctement évalué.

Relecture honnête :
- **Case B (mal exercé) reste** : 52% liquidity_draw au setup level mais 9% survit l'EC gate. Le schéma est encore sous-testé par les trades qui passent.
- **Le vrai blocker n'est plus le resolver** — c'est le gate EC qui filtre sélectivement contre les schema-exercés setups. Ceci est un NOUVEAU diagnostic issu de α''.

## Implications

**Ce qui est prouvé** :
1. Le fix `pool_selection: significant` est correct sémantiquement (verif unit tests + setup-level tp_reason shift).
2. Lookback 60 + min_rr_floor 0.5 + ceiling 3.0 est un band raisonnable — quasi tous les pools y sont (2 beyond_ceiling sur 62 setups = <3%).
3. Le resolver n'est PAS le bottleneck d'Aplus_03_v2 — l'EC gate l'est.

**Ce qui N'EST PAS prouvé** :
- ❌ "α'' améliore E[R]" — trade-level identique sur 2 semaines.
- ❌ "Le schéma `liquidity_draw` porte de l'edge" — les trades où il est exercé (n=3 sur 2 semaines) sont trop peu pour conclure.
- ❌ "Family A marche/ne marche pas" — cf verdict Aplus_03_v2 Case B initial, inchangé.

## Artefacts

- Code : [`tp_resolver.py:95-177`](../../engines/execution/tp_resolver.py#L95-L177) (nouveau `_select_pool` + params).
- Tests : [`test_tp_resolver_liquidity_draw.py`](../../tests/test_tp_resolver_liquidity_draw.py) (17 PASS).
- Overlay : [`aplus03_v2_alpha_pp.yml`](../../knowledge/campaigns/aplus03_v2_alpha_pp.yml).
- Résultats : [`results/labs/mini_week/aplus03_v2_alpha_pp/`](../../results/labs/mini_week/aplus03_v2_alpha_pp/).

## Recommandation prochain sprint

Ne PAS enchaîner Aplus_04_v2 ni Aplus_01 sur le resolver corrigé en espérant un résultat différent — le bottleneck identifié est l'EC gate, pas le resolver.

Trois axes séparés (plans distincts) :

**axe 1 — audit EC gate sur Aplus_03_v2 (read-only)** : re-exécuter [scripts/audit_entry_confirm.py](../../scripts/audit_entry_confirm.py) sur α'' (les 2 semaines déjà produites) pour quantifier exactement combien de `liquidity_draw_swing_k3` setups sont rejetés vs passés, et voir les peak_r / mae_r post-rejection. Si 91% des liquidity_draw setups rejetés mais avec peak_r > 0.8R → c'est bien une destruction d'edge. Cheap.

**axe 2 — relaxer l'EC gate sur α'' (1 smoke nov_w4)** : overlay avec `require_close_above_trigger: false` → combien de `liquidity_draw_swing_k3` deviennent trades ? Teste "si le gate est relaxé, le schéma produit-il un edge ?" — c'est la question de fond.

**axe 3 — Option δ (swing_k9)** : swing_k9 produit des pivots plus rares et plus loin, donc le band 0.5-3.0R y trouvera plus souvent un pool unique = trades plus divergents vs nearest sur k3. Plan séparé obligatoire (hors scope O1.1 sprint courant).

## Caveat

- 2 semaines smoke uniquement (user constraint "pas de long run tant qu'on n'est pas dans le vert"). CI larges sur n=10 total.
- `min_floor_binding` réduit de 12 → 8 est le signal le plus fort du fix, mais aucun de ces 4 setups reclassés n'a survécu à l'EC gate sur ce corpus.
- Le verdict "EC gate = nouveau bottleneck" s'appuie sur une cross-tab n=62 setups → directionellement solide mais à confirmer sur 4 semaines si ré-investi.
