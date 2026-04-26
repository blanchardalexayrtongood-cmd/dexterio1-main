# Leg 4.2 — VIX-regime overlay sur cohort survivor_v1 — VERDICT

**Date** : 2026-04-22
**Dossier** : [backend/knowledge/overlays/leg42_vix_regime_v1/dossier.md](../../knowledge/overlays/leg42_vix_regime_v1/dossier.md)
**Harness** : [backend/scripts/leg42_vix_overlay.py](../../scripts/leg42_vix_overlay.py)
**Output** : [backend/results/labs/mini_week/leg42_vix_overlay_v1/leg42_vix_overlay_results.json](../../results/labs/mini_week/leg42_vix_overlay_v1/leg42_vix_overlay_results.json)
**Décision** : **FAIL → ARCHIVED → ESCALADE USER §0.3 point 3**

---

## Bloc 1 — identité du run

- **Type** : overlay régime read-only, pas de nouveau playbook (plan §0.5 Leg 4.2)
- **Cohort** : `survivor_v1` restreint à 4 survivors (News_Fade + Engulfing_Bar_V056 + Session_Open_Scalp + Liquidity_Sweep_Scalp) cf [survivor_v1_verdict.md](survivor_v1_verdict.md)
- **Période** : 4 semaines canoniques (jun_w3 + aug_w3 + oct_w2 + nov_w4)
- **Source VIX** : `yfinance ^VIX daily close` 2025-05-01 → 2025-12-01 (147 rows, min 14.22 / max 26.42)
- **Gate** : prior-day VIX_close ∈ [15, 25] (bande fertile mean-rev §0.4-bis)
- **Merge** : `pandas.merge_asof` trading_date ↔ VIX prior-day strict (pas de look-ahead)

## Bloc 2 — métriques

### Baseline cohort (4 survivors × 4w, pas de filtre)

| Métrique | Valeur |
|---|---:|
| n | 90 |
| WR | 36.7 % |
| E[R] gross | **−0.0215** |
| PF | 0.769 |
| total_R | −1.94 R |

### Subset VIX fertile [15, 25] (ON gate)

| Métrique | Valeur |
|---|---:|
| n | **82** (91.1 % de baseline) |
| WR | 32.9 % |
| E[R] gross | **−0.0284** |
| PF | 0.713 |
| Δ E[R] vs baseline | **−0.0069 (PIRE)** |

### Subsets contrôle §0.4-bis

| Bande | n | share | E[R] | WR |
|---|---:|---:|---:|---:|
| low <15 | 8 | 8.9 % | **+0.0497** | 75.0 % |
| fertile [15,25] | 82 | 91.1 % | −0.0284 | 32.9 % |
| panic ≥25 | 0 | 0 % | — | — |

### Per-playbook baseline vs fertile

| Playbook | n_base | E[R]_base | n_fertile | E[R]_fertile | Δ |
|---|---:|---:|---:|---:|---:|
| News_Fade | 10 | +0.0012 | 9 | −0.0089 | **−0.0101** |
| Engulfing_Bar_V056 | 26 | −0.0101 | 24 | −0.0006 | +0.0095 |
| Session_Open_Scalp | 12 | −0.0139 | 10 | −0.0340 | **−0.0201** |
| Liquidity_Sweep_Scalp | 42 | −0.0361 | 39 | −0.0487 | **−0.0126** |

**3/4 playbooks strictement pires en subset fertile**. Seul Engulfing_Bar_V056 s'améliore marginalement (+0.0095 sur n=24).

## Bloc 3 — lecture structurelle

### Le filtre régime VIX est destructeur

Le résultat contre-intuitif est empiriquement clair : **restreindre la cohort à la bande "fertile" aggrave E[R] de −0.0069R** sur le corpus complet. Les 8 trades en bande low (<15) ont **E[R]=+0.0497 WR=75 %** — précisément les trades que l'overlay rejette.

Interprétation :
- Les 9 % de trades "low-vol" (VIX<15) étaient les meilleurs trades de la cohort, pas les pires.
- La thèse "vol fertile [15,25] = mean-rev edge" (plan §5.3 P4, plan §0.4-bis) ne tient pas sur ce corpus.
- L'hypothèse économique ("low-vol = trending, pas mean-rev") est **réfutée empiriquement** sur jun-nov 2025 SPY/QQQ intraday.

### Le corpus était déjà saturé en régime fertile

- 91.1 % des trades étaient déjà en bande [15,25] avant filtrage (corpus jun-nov 2025 ≈ VIX 15-20 dominant, cf plan §0.4-bis "vol fertile dominant").
- Le filtre ne retire que 9 % du corpus — et ces 9 % sont les outliers qui **portaient les meilleurs trades**.
- **0 trades en bande panic (≥25)** dans le corpus jun-nov 2025 → la bande panic ne peut pas être testée localement.

### Cross-check avec Phase D.1 bias audit

Ce résultat confirme le pattern Phase D.1 (`bias_audit_v1_verdict.md`) : aucun gate régime universel (HTF D, HTF 4H, combined D∧4H, maintenant VIX 15-25) ne débloque d'edge sur la cohort survivor_v1. Les 4 survivors sont à ~E[R]≈−0.02 à −0.04 indépendamment du régime. **Le bottleneck n'est pas le contexte de marché — c'est le signal lui-même.**

### Pourquoi n=8 VIX<15 outperform n'est pas load-bearing

E[R]=+0.0497 WR=75 % sur n=8 est statistiquement anecdotique (intervalle de confiance très large). Ne justifie pas la construction d'un overlay "VIX<15 only" ni la promotion d'aucun playbook. Noté mais pas actionnable (user bar +0.10R E[R] net + n>30 cf [real_results_bar.md](../../.claude/projects/-home-dexter-dexterio1-main/memory/feedback_real_results_bar.md)).

## Bloc 4 — décision

### Kill rules pièce H (pré-écrites dossier §18)

| # | Règle | Résultat |
|---|---|---|
| 1 | Subset cohort E[R] gross ≤ 0.05R | **FAIL** (−0.0284 << 0.05) |
| 2 | n subset < 30 | PASS (82 ≥ 30) |
| 3 | E[R] subset < E[R] baseline | **FAIL HARD** (filtre régime destructeur) |

**2/3 kill rules atteintes (kill rule 3 particulièrement sévère)** → **ARCHIVED**.

### Cas §20

- **Cas C dominant** (edge absent) : filtre VIX ne déverrouille rien ; subset strictement pire sur 3/4 playbooks.
- **Cas D secondaire** (hypothèse économique fausse) : thèse "vol fertile = mean-rev SPY/QQQ intraday" réfutée empiriquement sur corpus jun-nov 2025.

## Bloc 5 — why

### Pourquoi cette décision est rationnelle

- Subset fertile E[R] = −0.0284 vs baseline −0.0215 : **filtre régime REDUCES edge** au lieu de l'isoler — résultat directionnellement opposé à l'hypothèse, pas marginal.
- 91 % du corpus déjà dans la bande cible : l'overlay n'a pas de leverage sur ce corpus ; pour le tester honnêtement il faudrait un corpus où la distribution VIX est plus équilibrée (ex : période 2022 bear market avec VIX 20-35 fréquent, ou 2020-2021 avec VIX 10-40 range).
- 3/4 playbooks de la cohort dégradent en subset fertile — pas de clash entre playbooks, effet cohérent.

### Pourquoi on n'itère pas plus (§19.3)

- Budget §19.3 : itération 1/3 consommée. Les 2 itérations restantes peuvent être utilisées pour :
  - `VIX ∈ [12, 20]` (bande alternative) — mais le subset low-vol n=8 est trop rare pour tester proprement (§19.3 contre-indique de fitter l'histoire).
  - VIX percentile-based (VIX rank 20-80) — même problème de saturation corpus.
- Plan §0.5 est explicite : Leg 4.2 est un **fallback** après Legs 1-4 fail. Les 2 branches restantes de la §0.5 arbre ne sont pas des "overlays régime" — Leg 5 est ESCALADE USER.

### Pourquoi on ne tue pas trop tôt

- On ne tue pas — les 4 playbooks restent au statut qu'ils avaient avant (cf CLAUDE.md). L'overlay Leg42_VIX_Regime_Overlay_v1 est archivé, pas les playbooks sous-jacents.
- Les briques (VIX fetcher yfinance, merge_asof prior-day) sont triviales et réutilisables si futur besoin régime classifier.

### Pourquoi on ne promeut pas trop tôt

- Même en subset fertile, aucun playbook individuel n'atteint E[R] net+slippage > 0.10R (user bar [feedback_real_results_bar.md](../../.claude/projects/-home-dexter-dexterio1-main/memory/feedback_real_results_bar.md)).
- Engulfing_Bar_V056 fertile E[R]=−0.0006 (quasi-BE) : marginal edge refusé par user bar (feedback préalable, Leg 1.1 QQQ SHORT rejeté).

### Progression §0.5 arbre — **ESCALADE USER §0.3 point 3**

Plan §0.3 points d'escalade user :
> **3. Arbre §0.5 Legs 1-4 épuisés sans Stage 2 PASS** → nouvelle hypothèse fondamentale requise (ML, nouvelle data source, refonte).

État au 2026-04-22 :
- **Leg 1.1** Engulfing_Bar_V056 TP peak-R calib v1 : ARCHIVED 4w FAIL Cas C
- **Leg 1.2** Morning_Trap TP peak-R calib v1 : ARCHIVED KILL terminal 3/3 kill rules
- **Leg 2.1** IFVG_5m_Sweep solo 12w : ARCHIVED Cas C
- **Leg 2.2** VWAP_Bounce_5m solo 12w : ARCHIVED Cas C
- **Leg 2.3** HTF_Bias_15m_BOS solo 12w : ARCHIVED Cas C (meilleur des 3 mais gate cohort FAIL)
- **Leg 3** Aplus_02 Family F Premarket v1 : ARCHIVED SMOKE_FAIL Cas B (6e/6 MASTER families testée)
- **Leg 4.1** Stat_Arb_SPY_QQQ_v2 daily coint : ARCHIVED SMOKE_FAIL Cas C+D (byte-identical v1)
- **Leg 4.2** VIX-regime overlay cohort : **ARCHIVED FAIL Cas C+D (filtre destructeur)**

**0 Stage 2 PASS atteint**. §0.3 point 3 déclenché → **ESCALADE USER**.

Options pour discussion (plan §0.5 Leg 5) :
- **A — ML metalabeling** (Lopez de Prado) sur features engineered post-signal ; mais règle §9.1 "ML après signal gross > 0 robuste" → pas de candidat. Cette règle doit être renégociée ou assouplie.
- **B — HMM regime classifier learned** ; même blocage §9.1.
- **C — Nouvelle data source** : options flow (0-DTE SPY/QQQ), futures term structure (VX, ES), L2 order book (DAS, Polygon L2). Besoin Polygon 18m ingest (déjà rejeté post-Phase A pour "sauver quasi-BE" mais justifié ici comme nouvelle hypothèse fondamentale).
- **D — Refonte architecture** : portfolio construction first (vol-targeting, risk-parity sur univers ≥5 ETF), signal mean-rev après. Changement de direction stratégique.
- **E — Playbook académique spécifique** : tirer un paper peer-reviewed précis (ex : Jegadeesh momentum 3-12m, Avellaneda stat-arb mean-rev PCA-residuals) et l'instancier.
- **F — Accepter bear case** : pas d'edge tradable DexterioBOT 2025 SPY/QQQ intraday via méthodes testées → pivoter vers paper baseline cohort (validation infra paper sans edge positif) OU pause stratégique.

---

## Synthèse 1 ligne

> Leg 4.2 VIX-regime overlay (bande fertile [15,25]) sur cohort survivor_v1 → **filtre destructeur** (subset E[R]=−0.0284 vs baseline −0.0215), 2/3 kill rules atteintes, 10e data point négatif cross-playbook. **Legs 1-4 §0.5 arbre épuisés, 0 Stage 2 PASS → ESCALADE USER §0.3 point 3** pour nouvelle hypothèse fondamentale (ML / data source / refonte / paper baseline).
