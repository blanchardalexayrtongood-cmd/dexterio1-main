# Research audits consolidated — 2026-04-21

Quatre audits read-only exécutés après certification engine (37/37 tests), visant à identifier où se casse l'edge hors engine. Zéro long run.

## Résultats bruts

| # | Audit | Script | Verdict | Fichier |
|---|---|---|---|---|
| 1 | peak_R vs TP | [audit_peakR_vs_TP.py](../../scripts/audit_peakR_vs_TP.py) | 10/14 playbooks **GEOMETRY_CONDEMNED**, 4 TIGHT, 0 OK | [audit_peakR_vs_TP.md](audit_peakR_vs_TP.md) |
| 2 | entry_confirm funnel | [audit_entry_confirm.py](../../scripts/audit_entry_confirm.py) | 3 playbooks MASTER-faithful : **66-78% des setups clean tués par entry_confirm_no_commit** | [audit_entry_confirm.md](audit_entry_confirm.md) |
| 3 | scoring predictive power | [audit_scoring_power.py](../../scripts/audit_scoring_power.py) | **r²=0.006, corrélation NÉGATIVE (-0.078), monotonicity FALSE** (Grade C > Grade A en E[R]) | [audit_scoring_power.md](audit_scoring_power.md) |
| 4 | filter splits | [audit_filter_splits.py](../../scripts/audit_filter_splits.py) | 13 edge candidates in-sample (petits n, risque overfitting), thème récurrent : **SHORT > LONG**, **Monday**, **tight SL** | [audit_filter_splits.md](audit_filter_splits.md) |

---

## Le bot a 4 problèmes structurels indépendants

### Problème 1 — TP géométriquement inatteignable (10/14 playbooks)

**Fait** : le ratio `peak_R p80 / TP_RR` est < 0.80 pour 10 playbooks sur 14 avec n ≥ 15. Le marché n'offre presque jamais assez de MFE pour toucher le TP demandé.

**Pire cas** :
- ORB_Breakout_5m : ratio 0.235 (peak_R p80 = 0.47R vs TP = 2.0R)
- SCALP_Aplus_1 : ratio 0.289 (peak_R p80 = 0.58R vs TP = 2.0R)
- NY_Open_Reversal : ratio 0.405 (peak_R p80 = 1.21R vs TP = 3.0R)
- RSI_MeanRev_5m : ratio 0.422

**Tight** (ratio 0.80-1.10) :
- Aplus_03_IFVG_Flip_5m (R.3) : 1.048 — TP 0.70R atteint marginalement
- Aplus_04_HTF_15m_BOS_v1 : 1.015 — TP 1.0R à peine atteignable (23.6% de TP1)
- Aplus_03_v2 : 0.958 — TP via liquidity_draw, borderline
- Morning_Trap_Reversal : 0.811

**Implication** : la calibration incrémentale (BE, trailing, max_duration) ne résoudra PAS ces cas. C'est la **géométrie TP** qui est fausse, pas la mécanique exit. Solutions :
- TP dynamique sur liquidity_draw (déjà implémenté Option A v2, non-confirmé)
- Abaisser TP_RR sur les playbooks vers peak_R p60 (calibration TP)
- KILL les playbooks dont le ratio < 0.5 et qui ont déjà été tunés (ORB, SCALP_Aplus_1)

### Problème 2 — entry_confirm tue les setups MASTER-faithful (66-78%)

**Fait** : trois playbooks seulement utilisent `require_close_above_trigger: true + entry_buffer_bps: 2.0` dans leur YAML. Ce sont exactement les instantiations MASTER-faithful :

| Playbook | Attempted | Opened | EC kill % (est) |
|---|---:|---:|---:|
| Aplus_04_HTF_15m_BOS_v1 | 245 | 55 | **77.6%** |
| Aplus_03_v2 | 72 | 22 | **69.4%** |
| Aplus_03_IFVG_Flip_5m (R.3) | 105 | 35 | **66.7%** |

Tous les autres playbooks (non-Aplus_XX) ont EC kill% = 0 — ils n'utilisent pas cette confirmation.

**Implication** :
- Les 3 playbooks Family A/B testés ont perdu 2/3 de leurs setups avant exécution.
- On ignore si les setups rejetés étaient meilleurs ou pires que ceux exécutés (pas instrumentés dans les trades parquet — ils n'existent pas comme trade).
- **Action de recherche prioritaire** : instrumentation — logger les setups rejetés par `entry_confirm_no_commit` avec leur peak_R/mae_R théoriques (via replay), comparer à ceux exécutés. Si les rejetés sont meilleurs → gate destructeur. Si pires → gate protège.
- **Alternative court terme** : smoke 1-semaine Aplus_03_v2 avec `require_close_above_trigger: false` pour voir n=22 → n≈70 et où va E[R].

### Problème 3 — scoring est décoratif (et parfois contre-productif)

**Fait** : sur 5780 trades, le `match_score` explique **0.6% de la variance** du r_multiple, avec **corrélation négative** (-0.078). Les grades A/B/C sont **inversement monotones** : Grade C (n=453) avg_R = -0.033 > Grade A (n=3458) avg_R = -0.092.

Par playbook, les corrélations les plus prononcées sont **toutes négatives** :
- IFVG_5m_Sweep : r = -0.49 (p=0.004)
- Aplus_04_HTF_15m_BOS_v1 : r = -0.30 (p=0.027)
- DAY_Aplus_1 : r = -0.21 (p<0.001)

**Implication** :
- Le scoring ne discrimine pas. Un trade "A" n'est pas meilleur qu'un "C" — **il est souvent pire**.
- Le système de grading A+/A/B dans les YAML (`grade_thresholds`) est cosmétique. Filtrer par `quality: A_plus` dans les promotions futures n'apporte aucune protection.
- **Hypothèse probable** : les poids du scoring (`fvg_quality`, `context_strength`, `pattern_quality`) sont arbitraires, pas fittés. Ils pénalisent peut-être les conditions où l'edge existe vraiment (signal rare = bas score).
- **Action** : ne pas ajouter d'étage ML au-dessus du score (anti-pattern MASTER). Plutôt **refondre les poids par régression** sur un holdout, OU **abandonner le score** et utiliser seuils binaires (détecté / pas détecté).

### Problème 4 — edge candidates existent mais sont fragiles

**Fait** : 13 buckets ont avg_R > 0 avec n ≥ 15. Thèmes récurrents :
- **Direction SHORT** : Engulfing_Bar_V056 SHORT-only +0.063 (n=18), BOS_Scalp_1m SHORT-only +0.040 (n=26). Biais LONG-bias → SHORT fonctionnent mieux.
- **Monday** : Engulfing Monday +0.045 (n=16), BOS_Scalp Monday +0.029 (n=24).
- **Tight SL** (quintile 1) : Aplus_04 +0.024 (n=23, WR 69.6%), Liquidity_Sweep +0.007 (n=22).
- **QQQ-only** : Engulfing QQQ +0.044 (n=32).

**Caveat fort** :
- Tous in-sample sur ce corpus 4-semaines. Classic data-mining trap.
- n<30 pour la plupart → intervalles de confiance énormes.
- Combiné avec Problème 3 (scoring cassé), les "edges" détectés peuvent être des artefacts.

**Implication** :
- Ne **PAS** créer de YAML contraints par ces filtres sans validation holdout.
- À utiliser uniquement comme **pistes** pour designer un filtre à tester out-of-sample (fold training/test distinct, ou semaine non utilisée ici).
- Le signal structurel le plus robuste est le **direction bias** (SHORT systématiquement mieux sur certains playbooks) — peut valoir une ligne d'investigation.

---

## Hiérarchie des causes pour E[R]<0 systématique

Basé sur l'ampleur des findings :

1. **#2 entry_confirm kill 66-78%** — impact massif, potentiellement réparable immédiatement (flag off + smoke), mais risque inconnu (les 2/3 rejetés pourraient être les pires setups).
2. **#1 TP géométriquement faux (10/14)** — impact structurel. Option A v2 (liquidity_draw) est l'infrastructure de réponse, mais non-confirmée. Alternative immédiate : abaisser TP sur les pires cas.
3. **#3 scoring cassé** — impact systémique mais réparable hors critical path (le scoring affecte l'allocation/grade, pas la décision d'entrée elle-même pour la plupart des playbooks).
4. **#4 filters subset** — 13 candidats in-sample. ROI faible sans validation holdout, mais gratuit à garder en tête.

---

## Prochaine étape recommandée (1 action, no long run)

**Smoke 1-semaine Aplus_03_v2 sans `require_close_above_trigger`** (patch YAML local, pas commit) :
- Oct_w2 seul (1 semaine).
- Attendu : attempted 72 → opened ~65 (au lieu de 22).
- Si E[R] reste ≤ 0 avec n triplé → Problème 2 n'était pas le coupable, retour à Problème 1 (TP).
- Si E[R] croise zéro avec n triplé → entry_confirm destructeur confirmé, action immédiate = relax gate ou instrumentation rejects.

**Coût** : 1 smoke ≈ 2-3 minutes engine time. Répond à la question critique "entry_confirm protège ou détruit ?".

**Alternative plus conservative** (user "no long run") : instrumentation pure — logger les setups `entry_confirm_no_commit` avec peak_R/mae_R théoriques via replay ExecutionEngine sans ouvrir position. Zéro trades en plus, diagnostic pur. Nécessite ajouter ~20 lignes de code dans [setup_engine_v2.py](../../engines/setup_engine_v2.py) ou [execution_engine.py](../../engines/execution/execution_engine.py).

---

## Ce qu'on sait maintenant vs hier

**Hier** : "aucun playbook E[R]>0, engine peut-être fautif".
**Aujourd'hui** : engine clean (37/37), **4 coupables identifiés** : TP cassé sur 10/14, entry_confirm tue 2/3 des Aplus, scoring négativement corrélé, direction bias SHORT non-exploité.

**Aucune de ces 4 causes n'avait été identifiée quantitativement avant.** Les audits ont basculé le diagnostic de "vague" vers "ciblé".
