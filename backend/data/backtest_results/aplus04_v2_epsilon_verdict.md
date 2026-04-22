# Aplus_04_v2 ε — `reject_on_fallback=true`, schema isolé, nov_w4

## TL;DR

**3e data point convergent négatif sur le schéma α''. E[R]=-0.066 sur le subset liquidity_draw pur isolé.**

- n=12, WR 50%, E[R] gross=-0.066, total_R=-0.787, exit_reason 12 SL / 0 TP.
- **100 % des trades = `liquidity_draw_swing_k3`** (setup rejeté dès qu'aucun pool k3 dans la band).
- 95 setups rejetés via `reject_on_fallback_*` au niveau resolver (69 no_pool + 21 below_floor + 5 beyond_ceiling).
- Structure_alignment gate k3 inchangé : 149 passed / 109 rejected (42 % rejection, non-biaisé direction : 0 reject_long_vs_bear, 109 reject_short_vs_bull — pur artefact D-bias bearish sur la semaine, confirmé par 247 short_evaluated vs 11 long_evaluated).

**Lecture honnête** : le schéma isolé ne tient pas. E[R]=-0.066 est **pire** que le subset liquidity_draw du baseline α'' (-0.012). La différence n'est pas du bruit — elle révèle que le subset "clean" de α'' (n=8) était partiellement un **artefact de cap** (`max_setups_per_session=3` absorbait les fallback setups plus tôt dans la journée → seuls les liquidity_draw favorables atteignaient l'EC gate). En rejetant les fallback côté resolver, les slots se libèrent pour 4 setups liquidity_draw supplémentaires — et ces 4 incluent les pires trades de la semaine (3 mardi 2025-11-18 contre-bias + 2 LONG sur semaine bearish).

## Hypothèse testée

*Si on rejette les setups où le resolver ne trouve pas de pool k3 dans la band `[min_rr_floor, max_rr_ceiling]`, est-ce que le subset `liquidity_draw_swing_k3` seul produit E[R] > 0 ?*

Attendu baseline : subset liquidity_draw du run α'' n=8 → E[R]=-0.012 (quasi-BE). Si ε ≈ baseline → schéma isolable positif. Si ε < baseline → soit bruit (n faible), soit selection bias dans le baseline.

Résultat : **ε isolé = -0.066 (n=12) < baseline subset -0.012 (n=8)**. Confirme selection bias, schéma isolé négatif.

## Protocole

- **Overlay** : [aplus04_v2_epsilon.yml](../../knowledge/campaigns/aplus04_v2_epsilon.yml) dérivé de aplus04_v2_alpha_pp.yml. **Seule différence** : `tp_logic_params.reject_on_fallback: true` (vs absent/false).
- **Code brick** : [`tp_resolver.py`](../../engines/execution/tp_resolver.py) accepte `reject_on_fallback: bool`. Si True et verdict ∈ {no_pool, below_floor, beyond_ceiling} → retourne sentinel tp_price + `reason="reject_on_fallback_<verdict>"`.
- **Wire setup_engine** : [setup_engine_v2.py](../../engines/setup_engine_v2.py) (≈L275) skip setup si `tp_reason.startswith("reject_")`. Prefix signal permet au counter `_tp_reason_stats` d'enregistrer la catégorie avant drop.
- **Tests** : 32/32 PASS sur suite α''/δ/ε (3 nouveaux tests ε ajoutés à `test_tp_resolver_liquidity_draw.py`).
- **Smoke** : 1 semaine nov_w4 (user gate "pas de long run"), allowlist=Aplus_04_HTF_15m_BOS_v1, caps actives, kill-switch actif.

## Résultats détaillés

### Global (n=12, tous liquidity_draw_swing_k3)

| Metric | Valeur |
|---|---:|
| n | 12 |
| WR | 50.0 % |
| E[R] gross | **-0.066** |
| total_R | -0.787 |
| 0 TP hits | 12/12 SL |
| peak_R p50 / p80 | 0.70 / 0.79 |
| mae_R p20 / p50 | -0.96 / -0.27 |
| tp_rr médian | 1.05 |

### Par direction

| Direction | n | E[R] | total_R |
|---|---:|---:|---:|
| LONG | 2 | **-0.441** | -0.881 |
| SHORT | 10 | +0.009 | +0.094 |

Les 2 LONG = catastrophes sur semaine bearish. Les 10 SHORT = quasi-BE mais sans edge.

### Par jour

| Date | n | E[R] | total_R |
|---|---:|---:|---:|
| 2025-11-17 | 3 | +0.008 | +0.025 |
| 2025-11-18 | 3 | **-0.263** | -0.789 |
| 2025-11-19 | 3 | +0.138 | +0.413 |
| 2025-11-21 | 3 | -0.145 | -0.436 |

Mardi 2025-11-18 concentre la perte (absent dans le baseline α'' — expliqué plus bas).

### tp_reason_stats (setup-level, 149 setups post-structure-gate)

| tp_reason | count | % |
|---|---:|---:|
| liquidity_draw_swing_k3 | 54 | 36 % |
| reject_on_fallback_no_pool | 69 | 46 % |
| reject_on_fallback_below_floor | 21 | 14 % |
| reject_on_fallback_beyond_ceiling | 5 | 3 % |

Le resolver écarte **64 %** des setups qui auraient fait du fallback 2R en α''. Sur les 36 % restants (54 setups liquidity_draw), 29 atteignent l'EC gate, 12 passent → trades.

### structure_alignment k3 (inchangé vs α'')

| Metric | ε | α'' baseline |
|---|---:|---:|
| evaluated | 258 | ~261 |
| rejected | 109 (42 %) | ~115 (44 %) |
| reject_long_vs_bear | 0 | ~0 |
| reject_short_vs_bull | 109 | ~115 |
| long_evaluated | 11 | ~11 |
| short_evaluated | 247 | ~250 |

Le gate structure k3 est identique. reject_on_fallback n'interagit pas avec lui (gate structure appliqué AVANT resolver).

## Comparaison ε vs α'' baseline

### Niveau subset liquidity_draw seul

| Metric | α'' subset (n=8) | ε pur (n=12) | Δ |
|---|---:|---:|---:|
| n | 8 | 12 | +4 |
| WR | 50 % | 50 % | 0 |
| E[R] | -0.012 | **-0.066** | **-0.054** |
| peak_R p80 | 0.74 | 0.79 | +0.05 |
| mae_R p20 | -0.65 | -0.96 | -0.31 |

**Les 4 trades supplémentaires** (1 le 17/11 17:44 + 3 le 18/11 18:59/19:29/20:44) sont les "slots libérés" par rejet des fallback setups. Ils concentrent la perte :
- 3 trades mardi 18/11 : total -0.789R (la journée a 3 EC-passed setups qui en α'' auraient été partiellement capés par fallback_no_pool et n'auraient jamais trade). Mardi = journée la plus bearish de la semaine, 2 LONG comptent ici.
- 1 trade 17/11 17:44 : -0.40R environ.

**Interprétation** : le subset liquidity_draw du baseline α'' était `-0.012` partiellement parce que les `max_setups_per_session=3` + les 2-3 premiers setups par jour étaient souvent des fallback_no_pool qui absorbaient les slots **avant** que les setups liquidity_draw suivants (dans la journée) ne puissent trader. Le subset `-0.012` était donc un sous-échantillon **favorisé par l'heure de la journée** (tôt dans la session NY, mouvements moins étendus, pool k3 plus proche et plus rentable).

Quand on libère les slots (ε), on voit les trades liquidity_draw **tardifs** qui se produisent dans la journée — et ceux-là sont en moyenne plus mauvais (direction counter-bias plus probable, vol déjà exploité, etc.).

### Niveau portfolio 1 semaine

| Metric | α'' total (n=15) | ε (n=12) | Δ |
|---|---:|---:|---:|
| E[R] | -0.057 | -0.066 | -0.009 |
| total_R | -0.855 | -0.787 | +0.068 |
| WR | 53 % | 50 % | -3 pp |

**ε n'améliore pas E[R]** vs le mix α''. Il réduit les trades fallback (-0.15R chacun en α'') mais les trades liquidity_draw additionnels compensent la gain. Ratio quasi-nul au niveau portfolio 1 semaine.

## Lecture framework Case A/B/C

**Seuils rappel** (§ O5.2.bis plan) :
- Case A : E[R] > 0, n ≥ 15, PF > 1
- Case B : E[R] ≤ 0 MAIS % fallback > 50 % OU n < 15
- Case C : E[R] ≤ 0, % fallback < 30 %, n ≥ 20, structure gate fonctionnel

**Classification Aplus_04_v2 ε** :
- E[R] = -0.066 ≤ 0 → pas Case A
- % fallback = 0 % (c'est le design) mais n = 12 < 15 → **Case B par seuil n**
- Ou lecture alternative : n < 20 pour Case C → Case B borderline

Mais la lecture Case B ici est trompeuse : ε rejette les fallback **par construction**. On ne peut pas dire "sous-exercé" parce que la mesure du schéma isolé est exactement ce qu'on voulait. Le framework Case A/B/C suppose que "% fallback" mesure l'inefficacité du schéma, or ici 100 % des trades sont schéma.

**Lecture honnête hors framework** : **3e data point négatif convergent** sur le schéma α''.

## Synthèse 3 data points

| Playbook | Family | Signal | n trades | E[R] | % schéma | Interprétation |
|---|---|---|---:|---:|---:|---|
| Aplus_03_v2 α'' | A | IFVG 5m isolé | 22 | -0.019 | 27 % | Case B (sous-exercé) |
| Aplus_04_v2 α'' | B | HTF_D + 15m BOS | 15 | -0.057 | 53 % | Case C borderline |
| **Aplus_04_v2 ε** | B | HTF_D + 15m BOS (schéma isolé) | 12 | **-0.066** | 100 % | Schéma isolé < 0 |

**Convergence** :
1. Le schéma α'' + structure k3 ne crosse pas zéro, **quel que soit** le niveau d'exercice (27 %, 53 %, 100 %).
2. Les "subsets liquidity_draw quasi-BE" (-0.01 à -0.02) étaient partiellement des artefacts de sélection via caps.
3. Le signal lui-même (IFVG 5m isolé, HTF+15m BOS avec gate k3) est le plafond — le TP structurel (pool k3 dans band) ne débloque pas d'edge sur ces signaux.

## Verdict de sprint

**Option ε tranche la question** :
- Hypothèse réfutée : "le subset liquidity_draw isolé porte de l'edge". Sur Family B nov_w4 n=12, il ne le porte pas.
- 3 data points convergents négatifs sur 2 familles (A + B) → **bear ferme sur le schéma α'' tel qu'implémenté** (liquidity_draw swing_k3 + structure_alignment k3).
- Le schéma n'est pas mort en principe, mais il n'est pas le levier qui crée l'edge sur les signaux testés. **Le bottleneck est la composition du signal**, pas le TP.

## Next steps (décision user)

Trois options hiérarchisées, le plan les prévoit :

### Option A — Aplus_01 Family A full (sweep+IFVG+breaker)
- **Hypothèse** : le signal IFVG 5m isolé est trop mince. Signal riche (cluster = sweep de liquidité + IFVG + breaker block) peut carrier de l'edge là où IFVG seul ne le fait pas.
- **Requiert infra** : `require_liquidity_sweep` + `require_entry_confirm_1m` + breaker pattern detector. 2-3 jours dev. Cf [CLAUDE.md]("1m confirm-in-zone state machine absent").
- **Risque** : si Aplus_01 Case C (n>20, bien exercé négatif), bear ferme sur Family A en entier. 1 semaine infra + 1 semaine smoke = 2-3 semaines avant verdict.
- **Valeur** : dernier axe signal non-testé. Si négatif → baseline empirique bear case complet.

### Option B — Paper baseline sur survivors existants
- **Contexte** : survivor_v1 verdict (2026-04-20) — 4 survivors (News_Fade +0.001, Engulfing -0.010, Session_Open -0.014, Liquidity_Sweep -0.036). Best 3-pack E[R]=-0.009 (n=48 sur 4 semaines).
- **Action** : passer les 3 (ou 4) en paper trading IBKR/Alpaca paper account, 2-4 semaines, mesurer slippage réel + exécution réelle + discipline.
- **Valeur** : sort du labyrinthe backtest, mesure ce qui compte (discipline humaine + infra paper-ready validée via Phase W).
- **Risque** : aucun playbook E[R] > 0 confirmé, paper baseline = étude de discipline/infra, pas validation d'edge.

### Option C — Polygon 18 mois
- **Hypothèse** : 4 semaines = CI trop large pour verdict ferme. 18 mois = 40-80× le sample → CI serré.
- **Coût** : téléchargement Polygon + retest corpus existant. 1-2 semaines.
- **Valeur** : verdict robustness pour tout ce qui a déjà asymptoté à E[R]≈0. Si confirmé négatif sur 18 mois → classify terminal.
- **Risque** : si survivors restent -0.01 à -0.04 sur 18 mois, même conclusion mais avec CI serré → force la décision A ou B.

**Ma recommandation** (non-contraignante) : **A puis B**. Aplus_01 est le **dernier signal non-testé du MASTER**. Si Aplus_01 Case A → 1er playbook product-grade. Si Case C → paper baseline est le prochain step structuré. 2-3 semaines entre maintenant et décision paper.

Mais c'est le call user — 3 semaines infra vs 2 semaines paper direct.

## Caveats

- **1 semaine smoke** (user gate). n=12 → CI ultra-large sur E[R]. Extrapolation sur 4 semaines = -0.10 à +0.05 possible.
- **100 % SHORT biais** : 10/12 SHORT (D-bias bearish nov_w4). Pas d'évaluation LONG équilibrée — les 2 LONG sont des outliers catastrophiques qui tire E[R] de -0.01 (SHORT seul) vers -0.07 (all).
- **Structure gate k3 non-audité isolément** : hérite de la validation α'' (fonctionnel, rejection ~42-44 %).
- **Conservative slippage non appliqué** : extrapolation -0.045R de slippage (pass α'' Aplus_03_v2) → E[R] net+slippage ≈ **-0.11R**.
- **Code brick réutilisable** : `reject_on_fallback` est maintenant une brique du vrai moteur (pas un patch). Disponible pour tout playbook futur via YAML.

## Artefacts

- Overlay : [`aplus04_v2_epsilon.yml`](../../knowledge/campaigns/aplus04_v2_epsilon.yml).
- Résultats : [`results/labs/mini_week/aplus04_v2_epsilon/aplus04_v2_epsilon_nov_w4/`](../../results/labs/mini_week/aplus04_v2_epsilon/aplus04_v2_epsilon_nov_w4/).
- Code brick : [`tp_resolver.py`](../../engines/execution/tp_resolver.py) (param `reject_on_fallback`), [`setup_engine_v2.py`](../../engines/setup_engine_v2.py) (skip prefix `reject_`).
- Tests : 3 nouveaux dans [`test_tp_resolver_liquidity_draw.py`](../../tests/test_tp_resolver_liquidity_draw.py) (32/32 PASS).
- Verdicts précédents : [α'' Aplus_04_v2](aplus04_v2_alpha_pp_verdict.md), [α'' Aplus_03_v2](alpha_pp_tp_resolver_verdict.md), [δ swing_k9](delta_k9_verdict.md), [axe 1 EC audit](axe1_ec_audit_on_alpha_pp_verdict.md).
