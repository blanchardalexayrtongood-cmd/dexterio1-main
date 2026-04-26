# Verdict — Engulfing_Bar_V056 TP peak-R calibration v1 (Leg 1.1)

**Playbook** : Engulfing_Bar_V056
**Campagne** : engulfing_v056_tp_calib_v1
**Date** : 2026-04-22
**Dossier** : [backend/knowledge/playbooks/engulfing_v056_tp_calib_v1/dossier.md](../../knowledge/playbooks/engulfing_v056_tp_calib_v1/dossier.md)
**Verdict structure** : §18.3 (5 niveaux)

---

## Bloc 1 — Identité du run

| Champ | Valeur |
|---|---|
| Playbook | Engulfing_Bar_V056 |
| Version | v1 TP peak-R calib (itération 1/3 §19.3) |
| Période | 4w canonical : jun_w3 (2025-06-16→06-20) + aug_w3 (2025-08-18→08-22) + oct_w2 (2025-10-06→10-10) + nov_w4 (2025-11-17→11-21) |
| Mode | AGGRESSIVE ALLOW_ALL_PLAYBOOKS=true + CALIB_ALLOWLIST=Engulfing_Bar_V056 |
| Fill model | Ideal (backtest default) — commission_model=ibkr_fixed, slippage_pct=0.05%, spread_bps=2.0 (config `run_mini_lab_week`) |
| Caps | `--no-relax-caps` (cooldowns + session_cap actifs) ; kill-switch désactivé par défaut (calib corpus pattern) |
| Instruments | SPY, QQQ |
| Corpus | parquet local 1m jun–nov 2025 |
| YAML | [backend/knowledge/campaigns/engulfing_v056_tp_calib_v1.yml](../../knowledge/campaigns/engulfing_v056_tp_calib_v1.yml) |
| TP overrides | `tp1_rr: 2.0→0.68 ; tp2_rr: 4.0→1.20 ; breakeven_at_rr: 1.0→0.40 ; trailing_trigger_rr: 1.0→0.50 ; trailing_offset_rr: 0.5→0.25 ; min_rr: 2.0→0.68` |
| Git SHA | (staged — commit post-verdict) |

---

## Bloc 2 — Métriques

### Agrégat 4 semaines

| Métrique | Valeur | Baseline Engulfing (survivor_v1 cohort) | Δ |
|---|---|---|---|
| n | **38** | 26 | +12 |
| WR | **47.4 %** | 38.5 % | +8.9 pts |
| E[R] net (costs inclus) | **−0.0875** | −0.010 | **−0.078** |
| PF | **0.34** | 0.535 | −0.195 |
| total_R | **−3.33R** | −0.26R | −3.07R |
| peak_r p50 | 0.512 | 0.276 | +0.236 |
| peak_r p60 | 0.661 | 0.663 | −0.002 |
| peak_r p80 | 0.733 | 1.07 | −0.337 |
| total $PnL | −$3 212 | — | — |

### Exits breakdown
| Exit | Count | % |
|---|---:|---:|
| SL | 14 | 36.8 % |
| TP1 | 14 | 36.8 % |
| time_stop | 10 | 26.3 % |

### Per-week (0 weeks < -0.5R check)
| Week | n | E[R] | WR | PF | total_R | Flag |
|---|---:|---:|---:|---:|---:|---|
| jun_w3 | 8 | −0.162 | 37.5 % | 0.21 | **−1.30** | **< −0.5R ❌** |
| aug_w3 | 10 | −0.081 | 30.0 % | 0.36 | −0.81 | **< −0.5R ❌** |
| oct_w2 | 10 | −0.147 | 40.0 % | 0.12 | **−1.47** | **< −0.5R ❌** |
| nov_w4 | 10 | **+0.026** | 80.0 % | 1.54 | +0.26 | ✓ (seule positive) |

### Splits §0.4-bis (régime / direction / instrument)

**Par direction** (régime proxy : Engulfing direction reflète contexte tendance intraday)
| Direction | n | E[R] | WR | peak_r p60 |
|---|---:|---:|---:|---:|
| LONG | 19 | −0.116 | 36.8 % | 0.435 |
| SHORT | 19 | −0.059 | **57.9 %** | 0.703 |

**Par instrument**
| Symbol | n | E[R] | WR |
|---|---:|---:|---:|
| SPY | 15 | **−0.210** | 20.0 % |
| QQQ | 23 | −0.008 | 65.2 % |

**Subset le plus favorable** : QQQ SHORT — n=14, E[R]=**−0.008**, WR=71.4 %, peak_r p80=0.746R → **quasi-BE, pas edge**. Bar repo user feedback_real_results_bar.md : edges marginaux E[R] 0 à +0.02 **rejetés explicitement**.

---

## Bloc 3 — Lecture structurelle

### 3.1 Le signal vit-il réellement ?
**Oui**, exercé : n=38 sur 4 semaines (densité ~2.4/jour × 2 symbols), peak_r p60=0.661 préservé vs baseline (0.663). La calibration TP a fait ce qu'elle devait : **TP1 touché 14 fois (36.8 %)** vs baseline 7.7 % — x4.8. Le moteur émet et capture des winners au niveau p60 comme prévu.

### 3.2 Problème : signal / sortie / mécanique / contexte ?
**Signal + contexte, pas mécanique.** La calibration fonctionne techniquement (TP1 hit rate élevé), mais :
1. **Le signal Engulfing isolé n'a pas d'edge directionnel LONG en 2025** (uptrend SPY +8 % période, rejets haussiers → continuation, rejets baissiers → correction rapide — hypothèse émise pièce B confirmée dans le sens : SHORT 57.9 % WR vs LONG 36.8 %).
2. **SPY structurellement toxique pour ce signal** (E[R]=−0.21, WR=20 %) vs QQQ quasi-BE (E[R]=−0.008, WR=65.2 %). Possible : gaps overnight SPY, bruit rejets micro-pattern 5m.
3. **Trois des quatre semaines sont franchement négatives** (< −0.5R per-week). Seule nov_w4 sauve l'aggrégat partiellement. Variance extrême = **régime-dependent pur**, pas edge stable.
4. **peak_r p80=0.733** (vs baseline 1.07) : le corpus 4w est **moins expansif** que survivor_v1 → le ceiling 0.68R est déjà proche du p80. Plus de place à extraire via TP.

### 3.3 Distributions vs baseline
- **Winners capturés** : 14 TP1 hits vs 2 baseline. Calibration efficace côté capture.
- **Losers non épongés** : 14 SL + 10 time_stop = 63 % exits non-TP. Le BE à 0.40R protège peu (touché puis revient). Time_stop = 10/38 → signal meurt avant TP sur 26 %.
- **RR break-even théorique** : avec TP1=0.68R et SL=1.0R, WR_BE = 59.5 %. Obtenu : 47.4 %. **Déficit de 12 pts.** QQQ SHORT subset atteint 71.4 % WR mais E[R] reste quasi-BE parce que les winners moyens ne dépassent pas 0.68R significativement (TP plafonne) et les losers prennent 1R plein.

### 3.4 Case §20 classification
**Cas C dominant (edge absent)** sur le playbook globalement, avec un sous-cas **B (sous-exercé) possible sur QQQ SHORT subset** (n=14, E[R] quasi-BE). Mais :
- QQQ SHORT E[R]=−0.008 **ne croise pas la bar repo** (feedback_real_results_bar.md : > +0.02R minimum, > +0.10R pour promotion).
- Filtrer sur direction/instrument = tuner post-hoc sur corpus de validation = fit à l'histoire, pas à l'hypothèse (§10 règle 11, §19.3 budget).
- Plafond peak_r p80=0.733R quasi au niveau TP1 actuel → pas de levier TP restant.

→ **Pas de Cas B légitime** : l'hypothèse (pièce B) est que Engulfing 5m isolé produit 0.66R peak d'excursion exploitable. Vérifié sur les chiffres, mais **les losers cassent le RR**. Signal structurellement borderline.

---

## Bloc 4 — Décision

**ARCHIVED** → node 1.2 Morning_Trap TP peak-R calibré (progression automatique §0.5 arbre Leg 1).

### Kill rules pré-écrites (pièce H) atteintes

| Kill rule | Valeur observée | Seuil | Atteinte ? |
|---|---|---|---|
| `E[R]_net ≤ 0` | **−0.0875** | ≤ 0 | ✅ **ATTEINTE** |
| `WR < 40 %` | 47.4 % | < 40 % | ❌ non |
| `PF < 1.0` | **0.34** | < 1.0 | ✅ **ATTEINTE** |

**2/3 kill rules atteintes. Un suffit.** Sortie : ARCHIVED + progression node 1.2.

### Gate Stage 1 (§0.6) — évaluation complète pour référence
| Gate | Valeur | Seuil | Passe ? |
|---|---|---|---|
| E[R]_net > 0.05R | −0.0875 | > 0.05R | ❌ |
| n ≥ 15 | 38 | ≥ 15 | ✓ |
| peak_R p60 > 0.5R | 0.661 | > 0.5R | ✓ |
| 0 weeks < −0.5R | 3 weeks < −0.5R | 0 | ❌ |
| Split régime §0.4-bis | 1/4 weeks positive ; SPY vs QQQ ×25× gap ; LONG vs SHORT signe opposé | cohérent ≥3 régimes | ❌ |

Gate Stage 1 **FAIL** sur 3/5 critères. Pas de passage Stage 2.

---

## Bloc 5 — Why

### Pourquoi cette décision est rationnelle

1. **Hypothèse falsifiable (pièce B) réfutée dans sa totalité** : le mécanisme postulé (peak_R ~0.66R exploitable via TP1 calibré à 0.68R) est vérifié *numériquement* mais la contrepartie (WR > 59.5 % BE pour RR 0.68:1) n'est **pas obtenue** (47.4 %). L'excursion existe, elle n'est pas assez fiable.
2. **3/4 weeks négatives < −0.5R** : variance dominée par régime, pas par skill. Aucun filtre mono-axiale (direction, instrument) ne produit E[R] > +0.02R (user bar).
3. **Budget §19.3 respecté** : itération 1/3 seulement, mais **§20 Cas C (edge absent)** = pas d'itération Cas B/A légitime. "Tuer" est la bonne action, pas insister.

### Pourquoi on n'itère pas plus
- **Pas de Cas B légitime** : le signal est exercé (n=38, peak_r p60=0.66), le schéma de sortie fait ce qu'il doit (TP1 hit x4.8). L'edge manque, point.
- **QQQ SHORT subset (n=14, E[R]=−0.008)** : quasi-BE ≠ edge. User feedback_real_results_bar.md : "un edge marginal n'est pas un edge, c'est du noise positif". Itérer pour stabiliser un quasi-BE est **explicitement rejeté** par le repo.
- **Plafond peak_r p80=0.733R ≈ TP1=0.68R** : pas de levier TP2 restant (déjà calibré 1.20R = au-dessus p80). Augmenter TP1 = retour à la pathologie initiale (TP1 2.0R unreachable).
- **Next node 1.2 Morning_Trap peak_R p80=3.02R** (cf CLAUDE.md tableau) → levier TP peak-R beaucoup plus prometteur à tenter.

### Pourquoi on ne tue pas trop tôt
On ne tue pas — on ARCHIVE le playbook avec verdict §18.3 documenté, infra YAML + dossier préservés pour re-référence. Engulfing_Bar_V056 conserve son statut `ALLOWLIST` de base dans [playbooks.yml](../../knowledge/playbooks.yml) au cas où un filtre régime futur (VIX band, day-of-week) redevient pertinent — mais cette calibration TP v1 est close.

### Pourquoi on ne promeut pas trop tôt
Aucun des 5 critères gate Stage 1 (§0.6) n'est rempli collectivement. Promotion interdite par règle absolue §10.

---

## Suite — §0.5 Leg 1.2 Morning_Trap TP peak-R

Progression automatique. Dossier §18 + YAML override + 4w canonical + verdict même protocole. Morning_Trap peak_R p80=3.02R (CLAUDE.md) → levier TP beaucoup plus expansif. Précédents sur Morning_Trap : cumul B1→B2→C.1 = −0.147 → −0.081 (3 itérations déjà consommées). Per CLAUDE.md : "1 dernier levier TP peak-R calibrated avant KILL" — §19.3 exception explicite user-marked. Cette calibration v1 = itération 4 exceptionnelle terminale.
