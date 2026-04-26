# Verdict — Aplus_02_Premarket_v1 smoke nov_w4 (Leg 3 §0.5)

**Date** : 2026-04-22
**Décision** : **SMOKE_FAIL → ARCHIVED** (Kill Rule 1 atteinte) → **Leg 4 progression automatique**

---

## Bloc 1 — identité du run

| Champ | Valeur |
|---|---|
| Playbook | Aplus_02_Premarket_v1 (MASTER Family F — 6e et dernière famille non-testée) |
| Version | v1 (Leg 3 §0.5 Aplus_02 Family F Premarket) |
| Période | nov_w4 = 2025-11-17 → 2025-11-21 (5 sessions) |
| Instruments | SPY, QQQ |
| Mode | AGGRESSIVE + `RISK_EVAL_ALLOW_ALL_PLAYBOOKS=true` + `--calib-allowlist Aplus_02_Premarket_v1` |
| Caps | `--no-relax-caps` (caps actives) |
| Fill model | Ideal (baseline smoke) |
| Corpus | 9345 bars 1m × 2 symbols, premarket window 04:00-09:30 ET |
| Config YAML | [playbooks.yml Aplus_02_Premarket_v1](backend/knowledge/playbooks.yml) (ajouté 2026-04-22) |
| Dossier | [backend/knowledge/playbooks/aplus_02_premarket_v1/dossier.md](backend/knowledge/playbooks/aplus_02_premarket_v1/dossier.md) |
| Git SHA | b1da681 |

---

## Bloc 2 — métriques

| Métrique | Valeur |
|---|---:|
| **Matches (required_signals + time_window)** | **2** |
| Setups evaluated (structure_alignment k3) | 1 |
| Setups rejected k3 (`reject_short_vs_bull`) | 1 |
| Setups pass_aligned | 0 |
| **Setups accepted total** | **0** |
| **Trades opened** | **0** |
| **Trades closed** | **0** |
| E[R]_gross | N/A (n=0) |
| WR | N/A (n=0) |
| PF | N/A (n=0) |
| peak_R p80 | N/A (n=0) |

**Funnel complet** : 2 matches → 1 setup atteint structure_alignment k3 gate → 0 passent → **0 trades**. Autre match (2-1=1) rejeté plus tôt dans la pipeline (context_requirements ADX/htf_bias/day_type ou pattern_confirmations).

---

## Bloc 3 — lecture structurelle

### Le signal vit-il réellement ?
**Non.** 2 matches / 9345 bars × 2 symbols × 5 sessions = densité 2.1×10⁻⁵ matches/bar. Densité structurellement rare, inférieure à Aplus_01_full_v1 (1 emit pour séquence 5 états) et Asia_Sweep_V051 (0 matches original session mismatch).

### Le playbook est-il sous-exercé ?
**Critiquement.** SWEEP@5m + BOS@5m simultanés en premarket 04:00-09:30 ET = 2 événements sur 5 jours × 2 symbols. Deux hypothèses structurelles :
1. **Liquidité premarket trop thin** : BOS 5m nécessite structure clairement cassée, volume premarket (~10-20% RTH) ne produit pas assez de swings identifiables par le détecteur k3.
2. **Détecteurs infra calibrés RTH** : SWEEP@5m + BOS@5m thresholds (lookback, displacement) optimisés pour RTH, génèrent peu de signaux valides en premarket.

### Le problème vient-il du signal, de la sortie, de la mécanique, ou du contexte ?
**Du signal + contexte (pas de la sortie/mécanique).**
- Signal : 2 matches = quasi-silencieux (rareté extrême).
- Contexte gate : sur 1 setup évalué k3, 1 rejeté SHORT vs bull — cohérent uptrend 2025 (§CLAUDE.md 5e confirmation cross-playbook).
- Sortie : non atteinte (0 trades → TP/SL pas exercés).
- Mécanique : engine et gates fonctionnels (2 matches bien comptés, structure_alignment k3 gate fait son job).

### Distributions vs baseline
- vs Aplus_01_full_v1 (Sprint 1) : 1 emit → cascade 5 états. Aplus_02 v1 : 2 matches → gate k3 rejet. Les deux Cas B mais par mécanisme différent (Aplus_01 = séquence trop stricte ; Aplus_02 = session trop peu liquide pour détecteurs 5m).
- vs Asia_Sweep_V051 original : 0 matches (session mismatch London 02:00-05:00). Aplus_02 v1 : 2 matches (premarket NY 04:00-09:30 fonctionnelle mais quasi-silencieuse). → confirmation : la sélection de fenêtre premarket est correcte mais le signal SWEEP@5m + BOS@5m ne prospère pas dans ce régime.

---

## Bloc 4 — décision

### Kill Rules pièce H pré-écrites (AVANT smoke)

| Kill rule | Seuil | Observé | Statut |
|---|---|---|:---:|
| 1. n < 10 trades smoke 1 semaine | < 10 | **0** | ✅ **ATTEINTE** |
| 2. peak_R p80 < 1R | < 1R | N/A (n=0) | — |
| 3. E[R]_gross ≤ 0 | ≤ 0 | N/A (n=0) | — |
| 4. Gate Stage 1 | n≥10 ET peak_R p80≥1R ET E[R]>0 | Non atteint | Non applicable |

**1/3 kill rules atteintes (Kill Rule 1 seule évaluable faute de trades). → ARCHIVED terminal. → Leg 4 progression automatique.**

### Classification §20
**Cas B (sous-exercé / rareté structurelle) dominant** : signal vit (2 matches détectés, structure_alignment k3 gate évalue), mais n trop bas pour trancher (n=0 trades). Peak_R non-calculable.

**Hypothèse Family F Premarket non-réfutée** (n=0 insuffisant) mais **non-calibrable** à v1 pragmatique : SWEEP@5m + BOS@5m + structure k3 + HTF D + ADX 20 en fenêtre 04:00-09:30 ET = signal quasi-absent.

### Décision : **ARCHIVED + Leg 4 automatique**

---

## Bloc 5 — why

### Pourquoi cette décision est rationnelle

**§0.5 Leg 3** kill rules template Sprint 1 pré-écrites pièce H :
> "n < 10 OR peak_R p80 < 1R OR E[R] gross ≤ 0 → ARCHIVED + Leg 4"

**n=0 ≫ 10** ferme la fenêtre d'évaluation. Appliquer la règle sans détour.

### Pourquoi on n'itère pas plus

**§19.3 budget d'itération** : max 3 tentatives post-smoke par hypothèse. Itérations possibles (relâcher ADX, retirer require_structure_alignment, élargir required_signals à OR au lieu de AND, modifier SWEEP@5m/BOS@5m thresholds pour premarket) **NON-TENTÉES** pour les raisons suivantes :

1. **Bear case cross-playbook** : 7 data points convergents précèdent (Family A Aplus_01/Aplus_03/Aplus_03_v2/Aplus_04/Aplus_04_v2, Sprint 3 Stat_Arb non-ICT, Leg 2 cohort IFVG/VWAP/HTF 12w). Ajouter Aplus_02 v1 = 8e data point négatif sur le même méta-hypothèse "ICT/MASTER families structurellement tradables 2025 SPY/QQQ intraday".

2. **Rareté structurelle du signal** : 2 matches / 5 sessions × 2 symbols. Même un relâchement de gates ne peut pas transformer 2 matches en >10 trades sans changer les détecteurs infra — et modifier SWEEP@5m/BOS@5m pour fitter premarket = redéfinition signal = nouvelle hypothèse (= nouveau nom, nouveau dossier) per §10 règle 11.

3. **Pas de séquence Sprint 1 nouvelle tentée** : la plan §0.5 évoquait "briques réutilisables Sprint 1 — confluence_zone + pressure_confirm + Aplus01Tracker template preserved" pour un tracker séquentiel Premarket. Non-implémentée en v1 pragmatique. Mais Aplus_01 Sprint 1 avec tracker séquentiel full donnait déjà 1 emit / RTH session — le tracker séquentiel en premarket (liquidité encore plus thin) donnerait probablement ≤1 emit / 5j → même outcome Cas B.

### Pourquoi on ne tue pas trop tôt

On **ne tue PAS l'hypothèse Family F Premarket intrinsèquement** (non-réfutée). On archive ce playbook v1 spécifique. Une refonte structurellement différente (ex : gap fade non-MASTER, premarket earnings-driven, ou tracker séquentiel ICT premarket full infra) reste une hypothèse autorisée per §10 règle 11 — mais elle sort du scope §0.5 Leg 3 (6 MASTER families épuisées).

### Pourquoi on ne promeut pas trop tôt

**N/A** (0 trade = pas de question de promotion). Même si les 2 matches avaient produit 2 trades hypothétiques positifs, le gate Stage 1 exige n ≥ 15 + E[R] > 0.05R + peak_R p60 > 0.5R — impossible à atteindre avec 2 matches/semaine (extrapolation 12 semaines = ~24 matches → ~5-10 trades si 20-50% passent gates → sous n=15).

### Progression automatique

Per §0.5 arbre de décision :
> "Si ARCHIVED → Leg suivant. Leg 3 → Leg 4."

**Leg 4 — Non-MASTER quant hypotheses** (coût ~2-3j par node) :
- **4.1 Stat-arb v2 daily cointégration 60j rolling + entry intraday** — §10 réouverture légale (structurellement différent de v1 intraday 5m EG 200 bars réfuté Sprint 3). Briques D1+D2 (31 tests PASS) réutilisables intégralement. Dossier §18 + YAML + tests + smoke nov_w4.
- **4.2 VIX-regime overlay sur cohort survivor_v1** (fallback) — filtre régime §0.4-bis sur cohort existant News_Fade+Engulfing+Session_Open+Liquidity_Sweep.

Démarrage automatique Leg 4.1 Stat-arb v2 daily cointégration sans attendre confirmation user (§0.9 CEO mode autonome, sans escalade §0.3 déclenchée).

---

**Synthèse** : Aplus_02 Family F Premarket v1 (SWEEP@5m + BOS@5m + schéma α'' + PREMARKET_NY 04:00-09:30 ET) → 0 trades, 2 matches, kill rule 1 atteinte. ARCHIVED. **8e data point négatif MASTER/ICT** confirme bear ferme. Leg 3 épuisée en 1 smoke (budget §0.5 "1-2j infra" maintenu court car full infra existait). Progression Leg 4.1 quant non-ICT.
