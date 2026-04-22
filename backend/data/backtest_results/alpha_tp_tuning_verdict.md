# Option α — TP calibration Aplus_03_v2 : verdict FAIL

## TL;DR

**α tune dans la mauvaise direction.** `lookback_bars: 60→120` et `min_rr_floor: 0.5→0.3` font passer les TP **plus près** de l'entry, pas plus loin. Le `tp_reason` "vraiment exercé" (`liquidity_draw_swing_k3`) est tombé de 27% baseline à **0%** sur aug_w3 tuned ; le floor binding monte à 73%. E[R] recule.

**Cause racine identifiée** : `_find_next_pool` ([tp_resolver.py:146-176](../../engines/execution/tp_resolver.py#L146-L176)) sélectionne le pivot **le plus proche** dans la direction du trade (`min` pour LONG, `max` pour SHORT). Élargir la fenêtre ne fait qu'exposer des pivots **plus anciens mais plus proches** d'entry, qui dégradent le TP au lieu de le pousser vers une vraie liquidité.

## Protocole

- Overlay [aplus03_v2_tp_tuned.yml](../../knowledge/campaigns/aplus03_v2_tp_tuned.yml) : **seules** différences vs v2 baseline = `min_rr_floor 0.3` + `lookback_bars 120`. Reste identique (schema `liquidity_draw swing_k3`, structure gate k3, SL/BE/trailing).
- 2 smokes 1-semaine, caps actives, kill switch actif, allowlist=Aplus_03_v2 seul :
  - oct_w2 (n baseline = 2, petit)
  - aug_w3 (semaine n baseline la plus représentative de v2)
- Comparaison directe vs `aplus03_v2_ec_audit/aplus03_v2_aug_w3` (même playbook, baseline v2 avec `lookback 60, floor 0.5`).

## Résultats aug_w3 (la semaine la plus informative)

| | Baseline v2 (lookback 60, floor 0.5) | Tuned α (lookback 120, floor 0.3) | Δ |
|---|---:|---:|---:|
| n | 3 | 11 | +8 |
| WR | 100% (n=3, tiny) | 45.5% | — |
| E[R] | +0.026 | **-0.021** | -0.047 |
| peak_r p50 | 0.906 | 0.355 | -0.551 |
| peak_r p80 | 0.940 | 0.542 | -0.398 |
| tp_reason liquidity_draw_swing_k3 | 0% | **0%** | 0 |
| tp_reason fallback_rr_no_pool | 100% | 27% | -73pp |
| tp_reason fallback_rr_min_floor_binding | 0% | **73%** | +73pp |

**Résultats oct_w2** : n=2 (baseline 2 aussi), E[R]=-0.132, tp_reason 50/50 no_pool/min_floor_binding, `liquidity_draw_swing_k3` = 0. Trop petit pour conclure mais confirme le pattern.

## Pourquoi α dégrade au lieu d'améliorer

Lecture du resolver :

```python
# _find_next_pool — engines/execution/tp_resolver.py:146-176
LONG  → nearest HIGH pivot strictly above entry (smallest such price)
SHORT → nearest LOW  pivot strictly below entry (largest such price)
```

Le resolver **minimise la distance entry↔pool**. Deux implications :

1. **`lookback_bars: 60→120`** élargit la fenêtre des pivots éligibles. Les pivots supplémentaires (60-120 bars ago) qui entrent dans le set peuvent être **plus proches** d'entry que les pivots du set 60-bars. Comme le resolver pick le plus proche → le TP descend vers un pivot récent à 0.1-0.3R au lieu d'utiliser le fallback 2R ou un pool > 0.5R.
2. **`min_rr_floor: 0.5→0.3`** baisse le plancher protecteur. Un pool à 0.3-0.5R qui binded sur le floor 0.5R baseline est maintenant classifié `liquidity_draw_swing_k3` — MAIS le TP effectif est placé à 0.3R, pas 0.5R. Conjugué avec (1), les winners deviennent minuscules (0.3R × WR ~45% = E[R] structurellement négatif).

**Résultat net** : 100% du pool "liquidity_draw" baseline a disparu (dilué en min_floor_binding par expansion lookback), et les fallback no_pool sont tombés à 27% (lookback 120 trouve des pivots là où 60 en manquait). La catégorie "vraiment exercée" est **0%** sur les 2 smokes.

## Lecture Case A/B/C

**Case B confirmé au carré** : le schéma `liquidity_draw` n'est PAS exercé plus — il est exercé **pire**. `liquidity_draw_swing_k3` passe de 27% baseline à 0% tuned.

**Ce qui est prouvé** :
- Le knob `min_rr_floor` DOWN + `lookback_bars` UP est la mauvaise direction.
- La logique "nearest pool" du resolver est incompatible avec l'intent MASTER (draw to **significant** liquidity, pas nearest pivot).

**Ce qui N'EST PAS prouvé** (ne pas généraliser) :
- ❌ "`liquidity_draw` ne marche pas" — le schéma n'a toujours pas été réellement testé.
- ❌ "swing_k3 est inadapté" — le resolver favorise nearest, le problème peut être la sélection, pas le niveau.
- ❌ "Family A morte" — cf verdict Aplus_03_v2 Case B initial.

## Options pour vraiment exercer le schéma (plan séparé)

Trois axes, **non tentés dans ce sprint** (scope α verrouillé consumé) :

**α' — Inverser le tuning** : `min_rr_floor: 0.5→0.8` (force des winners ≥ 0.8R même sur pool proche) + garder `lookback_bars: 60`. Simule "TP soit vers un pool ≥ 0.8R, soit fallback 2R". Plus fidèle à l'intent "don't scalp below half your risk". Coût = 1 smoke aug_w3.

**α'' — Redesign resolver : "significant pool"** : changer `_find_next_pool` pour retourner le pool le plus **proche d'une cible R cible** (ex : le pivot dont la distance entry↔pool se rapproche le plus de 1.5R), pas le plus proche absolu. Coût = ~30 lignes code + tests + 1 smoke. Plus risqué (sémantique resolver changée, affecte compat tests).

**Option δ — swing_k9** : hors scope explicite du plan Option A v2 (§ "Hors scope"). Pivots k9 sont plus rares et plus loin naturellement. Coût = autoriser `swing_k9` dans `_ALLOWED_DRAW_TYPES` + 1 smoke. Plan séparé **obligatoire** car explicitement exclu du sprint actuel.

## Recommandation

**Ne pas enchaîner sur γ ni β avant d'avoir réellement exercé le schéma.** Courir Aplus_04_v2 ou Aplus_01 avec la même logique resolver reproduira le même pattern (nearest pool → floor binding) et donnera 2 data points faussement convergents Case B → on ne saura toujours rien du schéma.

**Ordre proposé** :
1. **α'** (1 smoke, 30min) — test peu coûteux de l'axe inversé. Si `liquidity_draw_swing_k3` passe > 50% ET E[R] bouge → schéma exercé honnêtement, on enchaîne sur γ/β.
2. Si α' aussi Case B → **α''** (redesign resolver). Plan séparé courant, 1 demi-journée + gates régression.
3. Si α'' Case B aussi → δ (swing_k9) OU accepter que le signal IFVG isolé ne produit pas de pools exploitables (Case C structurel).

## Artefacts

- YAML overlay : [aplus03_v2_tp_tuned.yml](../../knowledge/campaigns/aplus03_v2_tp_tuned.yml).
- Résultats : [`results/labs/mini_week/aplus03_v2_tp_tuned/aplus03_v2_tp_tuned_{oct_w2,aug_w3}/`](../../results/labs/mini_week/aplus03_v2_tp_tuned/).
- Baseline comparé : [`results/labs/mini_week/aplus03_v2_ec_audit/aplus03_v2_aug_w3/`](../../results/labs/mini_week/aplus03_v2_ec_audit/aplus03_v2_aug_w3/).

## Caveat

- n aug_w3 = 11 (tuned) vs 3 (baseline). WR 100% baseline est artefact sample minuscule, pas preuve de supériorité. Le vrai signal est le tp_reason breakdown — 100% no_pool baseline → 0% liquidity_draw_swing_k3 utilisable tuned.
- Pas de run jun/nov : le pattern est clair sur aug_w3 (semaine la + fournie). Courir 2 semaines de plus coûterait du temps sans renverser le diagnostic.
- Pas de coûts/slippage ajoutés ici (comparaison gross vs gross).
