# Aplus_04_v2 α'' — Family B α''-schema smoke nov_w4

## TL;DR

**2e data point sur le schéma α''. Case C borderline (bien exercé, négatif) sur 1 semaine.**

- n=15, WR 53.3%, E[R] gross=-0.057, total_R=-0.855, PF 0.66, MaxDD 1.97R.
- **tp_reason : 53% `liquidity_draw_swing_k3` / 40% `fallback_rr_no_pool` / 7% `fallback_rr_min_floor_binding`.**
- % fallback = 46.7% → **<50% → Case C** (vs Aplus_03_v2 α'' 72.7% → Case B).
- Sous-univers liquidity_draw (n=8) : E[R] = **-0.012** (quasi-breakeven).
- Sous-univers fallback_rr_no_pool (n=6) : E[R] = **-0.153** (perdant net, plein 1R SL).
- **Le fallback tire l'ensemble vers -0.057**. Si on ne prenait que les setups avec pool k3 dans la band [0.5R, 3.0R], E[R] serait ~-0.012R (BE, pas edge mais pas carnage).

**Lecture framework vs Aplus_03_v2 α''** :

| Metric (nov_w4) | Aplus_03_v2 α'' | Aplus_04_v2 α'' | Δ |
|---|---:|---:|---:|
| n trades | 7 | 15 | **+8** |
| WR | 42.9% | 53.3% | +10.4pp |
| E[R] gross | -0.017 | **-0.057** | -0.040 |
| % liquidity_draw | 27% | **53%** | +26pp |
| % fallback | 73% | 47% | -26pp |
| PF | — | 0.66 | — |

**Schéma mieux exerçable sur Family B (HTF+15m BOS, SLs plus structurels) que sur Family A IFVG 5m isolé.** Mais E[R] global plus négatif à cause du subset `fallback_rr_no_pool` (n=6, -0.153R chacun).

## Protocole

- Overlay [aplus04_v2_alpha_pp.yml](../../knowledge/campaigns/aplus04_v2_alpha_pp.yml) dérivé directement de Aplus_04_HTF_15m_BOS_v1 (Option B). **Seules différences vs v1** :
  - `tp_logic: liquidity_draw` (vs fixed_rr implicit)
  - `tp_logic_params: {draw_type=swing_k3, lookback=60, min_rr_floor=0.5, fallback_rr=2.0, pool_selection=significant, max_rr_ceiling=3.0}`
  - `require_structure_alignment: k3` (nouveau)
- `require_htf_alignment=D`, `require_close_above_trigger=true`, `entry_buffer_bps=2.0` INCHANGÉS.
- `setup_tf=15m`, `entry LIMIT fvg_retest confirmation_tf=1m`, `SL SWING recent_swing padding=3` INCHANGÉS.
- `max_duration=60min`, `breakeven=0.5R`, `trail_rr trigger=0.7 offset=0.3` INCHANGÉS.
- 1 semaine nov_w4 (user gate "pas de long run"), allowlist=Aplus_04 seul, caps actives.

## Résultats nov_w4

### Global
| Metric | v1 fixed_rr (4w aggregate) | α'' nov_w4 (1w) |
|---|---:|---:|
| n | 55 (~14/w) | 15 |
| WR | 47.3% | 53.3% |
| E[R] gross | -0.074 | -0.057 |
| PF | 0.39 | 0.66 |

**Direction bias** : 15/15 SHORT. Nov_w4 est une semaine baissière (D-bias bearish), et `require_htf_alignment: D` gate force la direction.

### tp_reason breakdown (le point qui compte)

| tp_reason | n | WR | mean_r | mae_r p20 | mae_r p50 |
|---|---:|---:|---:|---:|---:|
| **liquidity_draw_swing_k3** | **8 (53%)** | 50% | **-0.012** | -0.647 | -0.219 |
| fallback_rr_no_pool | 6 (40%) | 50% | **-0.153** | -1.005 | -0.966 |
| fallback_rr_min_floor_binding | 1 (7%) | 100% | +0.157 | -0.118 | -0.118 |

**Lecture** :
- Les 8 trades liquidity_draw perdent **12× moins** que les 6 fallback_rr_no_pool (|mean_r| 0.012 vs 0.153).
- Les fallbacks `no_pool` absorbent le 1R plein (mae_r p20 ≈ -1.0) → losers pleins.
- Les liquidity_draw absorbent **~0.65R** (trailing + BE activés, SL jamais atteint) → losers partiels.
- Le **subset liquidity_draw = quasi-breakeven (-0.012R)**. C'est le signal fondamental sur le schéma.

### exit_reason

| exit_reason | count |
|---|---:|
| SL | 14 |
| TP1 | 1 |

Seul 1 TP hit (le fallback_rr_min_floor_binding, target 0.5R). 14 SL — mais 8 trades ont un r_multiple ≠ -1R, ce qui indique que BE (0.5R) ou trailing (0.7R) a été touché avant SL → protection partielle.

### EC gate audit

- 31 events : **15 passed / 16 rejected_no_commit** (ratio ~1:1).
- EC gate fonctionnel sur Family B, comparable à ratio Aplus_03_v2.

## Lecture framework Case A/B/C

**Rappel seuils user** (§ O5.2.bis du plan) :
- Case A : E[R] > 0, n ≥ 15, PF > 1.
- Case B : E[R] ≤ 0 MAIS % fallback > 50% OU n < 15.
- Case C : E[R] ≤ 0, % fallback < 30%, n ≥ 20, structure gate fonctionnel.

**Classification Aplus_04_v2 α''** :
- E[R] = -0.057 ≤ 0 → pas Case A.
- % fallback = 46.7% → entre 30% et 50% → **Case C borderline** (plus proche C que B).
- n = 15 (juste au seuil Case B, juste sous seuil Case C n≥20).
- Structure gate non-wired côté v1 (YAML n'avait pas `require_structure_alignment` avant overlay — fonctionnel post-overlay, cf EC audit balance).

**Lecture honnête** : **Case C borderline**. Le schéma est majoritairement exerçable (53%), les trades où il s'exerce sont quasi-BE (-0.012), les trades où il ne s'exerce pas (fallback no_pool) sont catastrophes (-0.153). Sur 1 semaine n=15, CI ultra-large — ne permet pas de conclure "edge" ou "mort" avec certitude.

## Combinaison avec Aplus_03_v2 α'' (2 data points)

| Playbook | Family | Signal | n | E[R] | % schéma | Case |
|---|---|---|---:|---:|---:|---|
| Aplus_03_v2 α'' | A | IFVG 5m isolé | 7 | -0.017 | 27% | B (sous-exercé) |
| Aplus_04_v2 α'' | B | HTF_D + 15m BOS | 15 | -0.057 | 53% | C borderline |

**Ce qu'on apprend** :
1. **Le % d'exercice du schéma dépend du signal** : IFVG 5m isolé produit des cheap pools proches (k3 dans la band rare → 27%). HTF+15m BOS produit des SLs plus structurels (recent_swing 15m = plus loin) → pool k3 dans la band plus souvent (53%).
2. **Quand le schéma s'exerce, le résultat est quasi-BE, pas edge clair** (-0.012R sur n=8 Aplus_04).
3. **Quand le schéma ne s'exerce pas, le fallback 2R est systématiquement perdant** (100% SL, mae p20 = -1.0R).

**Pas encore 2 Case C convergents.** On a 1 Case B (Aplus_03_v2) + 1 Case C borderline (Aplus_04_v2). Le bear case ferme sur le schéma exigerait 2 Case C convergents avec n > 20 chacun.

## Itérations possibles (hors scope, plan séparé)

1. **Rejeter les setups sans pool k3 dans la band** plutôt que fallback. Si le subset liquidity_draw est BE et le fallback est perdant, ne pas trader quand pas de pool trouvé. Changement = `tp_logic_params.fallback_rr: null` ou hook "require_pool_in_band".
2. **Aplus_04_v2 extension 4 semaines** pour confirmer Case C avec n > 20 et CI plus serré. Peut-être E[R] tourne autour de -0.03 à -0.06 → bear ferme. Ou peut-être nov_w4 est un outlier bearish → lecture différente sur jun/aug/oct.
3. **Aplus_01 full** (Family A cluster sweep+IFVG+breaker) — vrai test du MASTER Family A avec confluences. Ouvre plus d'axes (3 signals) mais testerait si un signal riche compense le fallback. Pré-requis infra : `require_liquidity_sweep` + `require_entry_confirm_1m` ne sont pas wirés (cf CLAUDE.md "1m confirm-in-zone state machine absent").

## Caveats

- **1 semaine smoke** (user gate). n=15 trades → CI ultra-large sur E[R]. Toute conclusion ferme nécessiterait ≥4 semaines.
- **100% SHORT** (D-bias bearish sur nov_w4) → pas d'évaluation directionnelle équilibrée. Éventuellement jun/aug/oct produirait un mix LONG/SHORT différent.
- **Comparaison nov_w4 vs Aplus_04 v1** : v1 n'a pas de breakdown per-week lisible dans [b_aplus04_v1_verdict.md](b_aplus04_v1_verdict.md). La comparaison 4w aggregate vs 1w nov_w4 est indicative, pas rigoureuse.
- **Structure gate k3** fonctionnel sur Aplus_03_v2 (44% rejection). Non-audité séparément ici (reuse du même moteur, même code).
- **Conservative slippage non passé** (ConservativeFillModel) — E[R] net+slippage serait ~-0.10R à -0.12R (extrapolation du pass α'' Aplus_03_v2 : -0.045R de slippage).

## Décision

**Pas de conclusion ferme sur le schéma α''.** 2 data points (Aplus_03_v2 + Aplus_04_v2) montrent :
- Le schéma est structurellement exerçable (27-53% selon signal).
- Quand il s'exerce, E[R] tourne autour de -0.01 à -0.02 (quasi-BE).
- Quand il ne s'exerce pas (fallback), E[R] tourne autour de -0.15 (perdant plein 1R).
- Net : E[R] global -0.02 à -0.06 selon la mix.

**Option la plus propre à tester ensuite (décision user)** : l'itération "rejet setups sans pool" — 1 YAML overlay, 0 code, 1 semaine smoke. Si le subset liquidity_draw seul atteint E[R] > 0 en isolation (sans être contaminé par le fallback), ça débloque la conversation sur le schéma. Sinon, bear ferme justifié → pivot Aplus_01 Family A full (infra sprint 2-3 jours).

## Artefacts

- Overlay : [`aplus04_v2_alpha_pp.yml`](../../knowledge/campaigns/aplus04_v2_alpha_pp.yml).
- Résultats : [`results/labs/mini_week/aplus04_v2_alpha_pp/aplus04_v2_alpha_pp_nov_w4/`](../../results/labs/mini_week/aplus04_v2_alpha_pp/aplus04_v2_alpha_pp_nov_w4/).
- Code schéma (inchangé depuis α'') : [`tp_resolver.py`](../../engines/execution/tp_resolver.py), [`directional_change.py`](../../engines/features/directional_change.py).
- Verdicts précédents : [α'' Aplus_03_v2](alpha_pp_tp_resolver_verdict.md), [δ swing_k9](delta_k9_verdict.md), [axe 1 EC audit](axe1_ec_audit_on_alpha_pp_verdict.md), [b_aplus04_v1](b_aplus04_v1_verdict.md).
