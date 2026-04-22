# Option δ — Aplus_03_v2 "swing_k9" : verdict NO-GO

## TL;DR

**δ ne débloque rien.** Sur nov_w4, `draw_type: swing_k9` produit **0/7 trades avec `liquidity_draw_swing_k9`** et **7/7 en `fallback_rr_no_pool`**. Le resolver ne trouve **jamais** de pivot k9 dans la fenêtre de lookback=60 bars. Tous les trades tombent sur le fallback 2R, E[R]=-0.016 (vs α'' -0.017, quasi-identique).

**Hypothèse testée réfutée** : "k3 produit des cheap pools proches, k9 force des vrais targets exploitables." En réalité, **k9 produit zéro pool dans 60 bars de 5m** — kappa=9×ATR est un seuil trop haut pour qu'un move intraday génère assez de pivots.

**Conséquence décisionnelle** : les 3 variantes testées du schéma `liquidity_draw` sur IFVG 5m isolé (α nearest k3, α'' significant k3, δ significant k9) oscillent toutes autour de E[R] ≈ -0.02. Le goulot n'est plus le schéma TP. **C'est le signal IFVG 5m isolé.**

## Protocole

- Overlay [aplus03_v2_delta_k9.yml](../../knowledge/campaigns/aplus03_v2_delta_k9.yml) dérivé directement de α''. **Seule différence** : `draw_type: swing_k3 → swing_k9`.
- Tout le reste (pool_selection=significant, max_rr_ceiling=3.0, min_rr_floor=0.5, lookback_bars=60, require_close_above_trigger=true, structure_alignment=k3) **INCHANGÉ**.
- 1 smoke nov_w4 (user constraint "pas de long run"), allowlist=Aplus_03_v2 seul, caps actives.
- Code : `_ALLOWED_DRAW_TYPES = {"swing_k3", "swing_k9"}` + map `_DRAW_TYPE_TO_PIVOT_KEY` dans [tp_resolver.py](../../engines/execution/tp_resolver.py). Reason string générée par f"liquidity_draw_{draw_type}".
- Tests : 15/15 PASS (3 nouveaux swing_k9 : reads k9 pivots not k3, pool hit emits k9 reason, significant mode SHORT mirror).

## Résultats nov_w4

| Metric | α'' (k3) | **δ (k9)** | Δ |
|---|---:|---:|---:|
| n trades | 7 | 7 | 0 |
| E[R] | -0.017 | **-0.016** | +0.001 |
| WR | 42.9% | 42.9% | 0 |
| peak_R p80 | 0.632 | 0.677 | +0.045 |
| mae_R p20 | -0.395 | -0.395 | 0 |
| total_R | -0.118 | -0.108 | +0.010 |

**Trade-level quasi-identique.** δ a une peak_R p80 marginalement supérieure (+0.05R), mais dans le bruit.

## tp_reason breakdown (le point qui compte)

**α'' nov_w4** :
| tp_reason | count |
|---|---:|
| liquidity_draw_swing_k3 | 3 (43%) |
| fallback_rr_min_floor_binding | 2 (29%) |
| fallback_rr_no_pool | 2 (29%) |

**δ nov_w4** :
| tp_reason | count |
|---|---:|
| **fallback_rr_no_pool** | **7 (100%)** |
| liquidity_draw_swing_k9 | 0 |
| fallback_rr_min_floor_binding | 0 |

**0% schéma exercé sur δ** (vs 43% sur α''). Tous les trades δ tombent au fallback 2R.

## Pourquoi k9 ne trouve rien

Rappel des kappas : `detect_structure_multi_scale(kappas=(1.0, 3.0, 9.0))`. Sigma du zigzag = `kappa × ATR`. Sur 5m intraday, ATR typique ~0.3-0.6$ (SPY/QQQ). k9 exige un swing de **2.7-5.4$** pour émettre un pivot. Dans 60 bars × 5min = 5 heures de marché, les moves cumulés suffisants sont rares — souvent 0-1 pivots k9 dans la fenêtre, pas suffisant pour produire un pool in-direction.

**k3** (sigma = 3×ATR ≈ 1-2$) → pivots bien plus fréquents → pool trouvé dans 43% des cas.
**k9** (sigma = 9×ATR ≈ 3-5$) → pivots rares → pool trouvé dans **0% des cas**.

Kappa=9 n'est pas adapté à un lookback intraday de 60 bars. Il faudrait soit un lookback beaucoup plus long (plusieurs jours), soit un kappa intermédiaire (k5/k7) non câblé.

## EC audit cohérent avec α''

| Metric | α'' | δ |
|---|---:|---:|
| EC passed | 7 | 7 |
| EC rejected_no_commit | 23 | 23 |
| structure k3 rejection | (cf α'' verdict) | — (identique) |

Les upstream filters (structure_alignment=k3, EC, session, scoring) sont inchangés entre α'' et δ, donc les candidats qui atteignent le resolver sont les mêmes. La seule différence est le TP calculé ensuite.

## Lecture selon les seuils user

Rappel du framework utilisateur :
- "liquidity_draw_* monte franchement" → **FAIL** (0% vs 43%, ça s'effondre).
- "fallback_rr_* baisse franchement" → **FAIL** (100% fallback vs 57%, ça monte).
- "n trades passés enfin un peu moins ridicule" → FAIL (n=7 identique, EC passed=7 identique).
- "peak_R/E[R] bougent dans le bon sens" → quasi-nul (peak_R p80 +0.045R, E[R] +0.001R).

**Aucun des 4 seuils franchi.** δ échoue à sa propre hypothèse.

## Conclusion solide (conditionnelle à 2 weeks)

Les 3 variantes testées sur IFVG 5m isolé (Aplus_03_v2) :

| Variante | % schéma exercé | E[R] nov_w4 | Verdict |
|---|---:|---:|---|
| v2 baseline (α nearest k3) | 45% (nov_w4 30%) | -0.019 (4w) | Case B sous-exercé |
| **α''** (significant k3) | 27% (aug+nov) | -0.017 (nov) | Case B persistant (axe 1 EC = protecteur) |
| **δ** (significant k9) | **0%** | -0.016 (nov) | schéma dégénéré complet |

**Le schéma ne produit pas d'edge sur IFVG 5m isolé, quelle que soit la scale de pivots testée.** Le signal lui-même est trop faible ou trop rare pour que la structure TP change le cadran. Caveat stats : toujours n petit, α'' passed liquidity_draw n=3 → signal *peut* exister, juste sous-exercé.

**Ce qui N'EST PAS prouvé** :
- ❌ "liquidity_draw ne marche pas" — seulement testé sur IFVG 5m isolé, Aplus_03_v2. Family B (Aplus_04) et Family A full (Aplus_01 sweep+IFVG+breaker) jamais testés avec le schéma.
- ❌ "swing_k9 n'est jamais utile" — testé uniquement avec lookback=60 bars. Un lookback plus long (240 bars = full day) pourrait produire des pivots k9 exploitables. Pas de plan pour tester maintenant (user gate "pas de long run" + discipline "1 axe").
- ❌ "Le schéma ne fonctionne sur aucun playbook" — 1 playbook seulement a été testé sur 2 semaines smoke.

## Recommandation next

Mon vote : **Aplus_04_v2** pour obtenir le 2e data point attendu par le framework original.

Pourquoi :
- Axe 1 a validé que EC est protecteur → plus rien à tenter côté filtre.
- δ a validé que le resolver ne peut pas sauver IFVG 5m isolé → le signal est le plafond.
- **Aplus_04_v2** (Family B, HTF_15m_BOS) change le signal tout en gardant le schéma (significant k3, EC, structure_alignment k3). C'est exactement le 2e data point prévu par le framework Case A/B/C.
- Si Aplus_04_v2 donne aussi Case B/C → **2 data points convergents**, conclusion bear ferme : le schéma YAML tel qu'implémenté ne suffit pas, pivot vers Aplus_01 Family A full (cluster confluences).
- Si Aplus_04_v2 donne Case A → le schéma porte de l'edge sur Family B mais pas sur IFVG isolé → conclusion nuancée sur le signal, pas sur le schéma.

Alternative : **Aplus_01 full** (sweep+IFVG+breaker) directement — skip le 2e data point schéma, aller tester le cluster MASTER. Mais ça ouvre plus d'axes simultanément (3 signals au lieu de 1), lecture moins propre.

## Caveats

- **1 semaine smoke** (user gate). n=7 trades → CI ultra-large sur E[R] single-week.
- **Lookback=60 non-varié** par discipline. Si on veut *réellement* exonérer k9, il faudrait un test avec lookback=240. Pas prévu ici.
- **`require_structure_alignment: k3` garde** — on teste k9 sur le resolver mais le gate structure reste k3. Cohérent (k3 = "direction du marché", k9 = "cible de liquidité") mais un ajustement possible (k9 uniforme) n'est pas testé.
- **Pas de CI formelle** sur la différence α'' vs δ. À 7 trades × 1 semaine, toute conclusion "E[R] quasi-identique" est stat-soft. Le signal fort est qualitatif : **0% schéma exercé**, observable directement sans stats.

## Artefacts

- Code : [`tp_resolver.py:47-48, 130-134, 164`](../../engines/execution/tp_resolver.py) (ALLOWED_DRAW_TYPES + DRAW_TYPE_TO_PIVOT_KEY + f-string reason).
- Tests : [`test_tp_resolver_liquidity_draw.py:290-358`](../../tests/test_tp_resolver_liquidity_draw.py) (3 nouveaux swing_k9), 15/15 PASS.
- Overlay : [`aplus03_v2_delta_k9.yml`](../../knowledge/campaigns/aplus03_v2_delta_k9.yml).
- Résultats : [`results/labs/mini_week/aplus03_v2_delta_k9/`](../../results/labs/mini_week/aplus03_v2_delta_k9/).
- Verdicts précédents : [α''](alpha_pp_tp_resolver_verdict.md), [axe 1 EC audit](axe1_ec_audit_on_alpha_pp_verdict.md).
