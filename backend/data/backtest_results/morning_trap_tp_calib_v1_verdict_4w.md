# Verdict — Morning_Trap_Reversal TP peak-R calibration v1 (Leg 1.2)

**Playbook** : Morning_Trap_Reversal
**Campagne** : morning_trap_tp_calib_v1
**Date** : 2026-04-22
**Dossier** : [backend/knowledge/playbooks/morning_trap_tp_calib_v1/dossier.md](../../knowledge/playbooks/morning_trap_tp_calib_v1/dossier.md)
**Verdict structure** : §18.3 (5 niveaux)
**Statut** : itération 4/3 **exceptionnelle terminale** §19.3 (CLAUDE.md user-marked "1 dernier levier TP peak-R calibrated avant KILL")

---

## Bloc 1 — Identité du run

| Champ | Valeur |
|---|---|
| Playbook | Morning_Trap_Reversal |
| Version | v1 TP peak-R calib (itération 4/3 §19.3 exception terminale) |
| Période | 4w canonical : jun_w3 (2025-06-16→06-20) + aug_w3 (2025-08-18→08-22) + oct_w2 (2025-10-06→10-10) + nov_w4 (2025-11-17→11-21) |
| Mode | AGGRESSIVE ALLOW_ALL_PLAYBOOKS=true + CALIB_ALLOWLIST=Morning_Trap_Reversal (bypass DENYLIST modes.yml:53) |
| Fill model | Ideal (backtest default) — commission_model=ibkr_fixed, slippage_pct=0.05%, spread_bps=2.0 (config `run_mini_lab_week`) |
| Caps | `--no-relax-caps` (cooldowns + session_cap=2 actifs) ; kill-switch désactivé par défaut (calib corpus pattern) |
| Instruments | SPY, QQQ |
| Corpus | parquet local 1m jun–nov 2025 |
| YAML | [backend/knowledge/campaigns/morning_trap_tp_calib_v1.yml](../../knowledge/campaigns/morning_trap_tp_calib_v1.yml) |
| TP overrides | `tp1_rr: 3.0→0.83 ; tp2_rr: 5.0→1.50 ; breakeven_at_rr: 2.15→0.40 ; trailing_trigger_rr: null→0.50 ; trailing_offset_rr: null→0.25 ; min_rr: 3.0→0.83 ; max_duration_minutes: 155→120` |
| Git SHA | (staged — commit post-verdict) |

---

## Bloc 2 — Métriques

### Agrégat 4 semaines

| Métrique | Valeur | Baseline Morning_Trap cumul B1→B2→C.1→M | Δ vs best prior (C.1) |
|---|---|---|---|
| n | **38** | 32 (C.1) / 34 (M gross) | +6 |
| WR | **34.2 %** | 28 % (C.1) | +6.2 pts |
| E[R] net (costs inclus) | **−0.1321** | **−0.081 (C.1 best)** | **−0.051 (régression)** |
| PF | **0.265** | — | — |
| total_R | **−5.02R** | — | — |
| peak_r p50 | 0.572 | — | — |
| peak_r p60 | 0.661 | 0.827 (audit peakR_vs_TP survivor_v1) | **−0.166** |
| peak_r p70 | 0.727 | — | — |
| peak_r p80 | 0.891 | 2.432 (survivor_v1) / 3.02 (M isolated) | **−1.54 vs survivor** |
| mae_r p20 | −1.006 | −1.054 (survivor) | +0.048 |
| total $PnL | −$6 094 | — | — |

**Observation critique** : peak_r p60 sur 4w canonical (0.661) **bien en dessous** de la donnée survivor_v1 (0.827) utilisée pour calibrer TP1. Le TP1=0.83R visait p60 survivor mais est en réalité au-dessus de p70 (0.727) du corpus 4w. **Calibration overshooted** : TP1 proche de p80 (0.891) = seuls les trades avec excursion maximale touchent TP.

### Exits breakdown
| Exit | Count | % |
|---|---:|---:|
| SL | 23 | 60.5 % |
| TP1 | 9 | 23.7 % |
| eod | 6 | 15.8 % |

### TP reasons
| Reason | Count | % |
|---|---:|---:|
| fixed_rr | 38 | 100 % |

Pas de resolver `liquidity_draw` (schéma legacy Morning_Trap). TP1 0.83R fixe appliqué à tous.

### Per-week (2 weeks < -0.5R check)
| Week | n | E[R] | WR | PF | total_R | Flag |
|---|---:|---:|---:|---:|---:|---|
| jun_w3 | 8 | **−0.297** | 12.5 % | 0.011 | **−2.38** | **< −0.5R ❌** |
| aug_w3 | 10 | **+0.050** | 60.0 % | 2.085 | +0.50 | ✓ |
| oct_w2 | 10 | **−0.322** | 20.0 % | 0.083 | **−3.22** | **< −0.5R ❌** |
| nov_w4 | 10 | +0.007 | 40.0 % | 1.158 | +0.07 | ~ (quasi-BE) |

Meilleure semaine aug_w3 (E[R] +0.050) reste en dessous du bar §0.6 Stage 1 (>0.05R) — touche le seuil sans le dépasser. oct_w2 catastrophique (-3.22R) à elle seule fait basculer l'agrégat.

### Splits §0.4-bis (régime / direction / instrument)

**Par direction** (Morning_Trap = contrarian au pattern → direction = anti-rejet)
| Direction | n | E[R] | WR | peak_r p60 |
|---|---:|---:|---:|---:|
| LONG | 11 | **−0.462** | **9.1 %** | 0.109 |
| SHORT | 27 | **+0.002** | **44.4 %** | 0.804 |

**Asymétrie LONG/SHORT écrasante** : LONG E[R]=−0.46 quasi total, WR 9% (1 win sur 11). SHORT quasi-BE. Même pattern qu'Engulfing Leg 1.1 (uptrend 2025 confirmé).

**Par instrument**
| Symbol | n | E[R] | WR |
|---|---:|---:|---:|
| SPY | 20 | **−0.254** | 20.0 % |
| QQQ | 18 | **+0.004** | 50.0 % |

**Split direction × instrument**
| Subset | n | E[R] | WR | peak_r p60 |
|---|---:|---:|---:|---:|
| QQQ SHORT | 16 | **+0.016** | 56.2 % | 0.854 |
| SPY SHORT | 11 | −0.018 | 27.3 % | 0.696 |
| SPY LONG | 9 | **−0.543** | 11.1 % | 0.107 |
| QQQ LONG | 2 | −0.096 | 0 % | 0.090 |

**Subset le plus favorable** : QQQ SHORT — n=16, E[R]=**+0.016**, WR 56.2 %, peak_r p60=0.854R → **marginal positif en dessous du user bar**. feedback_real_results_bar.md : edges 0 à +0.02R **rejetés explicitement** (bar = E[R]>0.05R Stage 1 et E[R]>0.10R paper).

---

## Bloc 3 — Lecture structurelle

### 3.1 Le signal vit-il réellement ?
**Oui**, exercé : n=38 sur 4 semaines (densité ~1.9/jour × 2 symbols, cap session=2 binding certains jours). TP1 touché 9/38 (23.7 %) vs baseline 8 % (×3). Mais **peak_r p60=0.661 vs calibration cible 0.83R** : l'excursion moyenne observée est plus faible que la donnée historique survivor_v1 (0.827R). La calibration TP visait une médiane d'excursion qui n'existe pas sur ce corpus 4w.

### 3.2 Problème : signal / sortie / mécanique / contexte ?
**Signal + contexte**, pas mécanique. 

1. **Asymétrie LONG/SHORT catastrophique** : LONG E[R]=−0.462, WR 9.1 %, 10/11 trades perdants. SHORT E[R]=+0.002 WR 44.4 %. Pattern "Morning Trap contrarian" LONG = acheter dans un rejet baissier = prendre le contre-pied d'une continuation uptrend 2025. Systématiquement cassé par le trend. **Même pathologie qu'Engulfing Leg 1.1** → confirmation cross-playbook que les signaux contrarian 5m LONG sont structurellement perdants sur ce corpus.
2. **SPY toxique** : E[R]=−0.254 vs QQQ +0.004. Écart ×63× entre symbols. Gaps overnight SPY + rejets micro-pattern 5m moins fiables que QQQ.
3. **2/4 weeks franchement négatives** (< −0.5R) : jun_w3 -2.38R, oct_w2 -3.22R. Variance régime-dominée, pas edge stable.
4. **peak_r p80 (0.891R) ≈ TP1 (0.83R)** : plafond d'excursion quasi-identique au TP1 → pas de levier TP restant, TP1 déjà au ceiling.
5. **RR break-even théorique** : avec TP1=0.83R et SL=1.0R, WR_BE = 54.6 %. Obtenu : 34.2 %. **Déficit 20.4 pts.** QQQ SHORT subset atteint 56.2 % WR (cross BE) mais E[R]=+0.016 ne franchit pas user bar.

### 3.3 Distributions vs baseline
- **Winners capturés** : 9 TP1 hits vs 2-3 baseline ratio. Calibration efficace côté capture mais pas suffisante.
- **Losers non épongés** : 23 SL + 6 eod = 76 % exits non-TP. BE à 0.40R protège peu (winners courts qui reviennent). eod = 15.8 % → signal souvent meurt avant TP.
- **Régression vs meilleur prior C.1** : E[R] −0.081 (C.1) → −0.132 (calib v1). La calibration TP peak-R **dégrade** vs vwap_regime seul. C.1 avait WR 28 % n=32 — cette v1 a WR 34 % n=38 mais E[R] pire. **Le TP 0.83R arrive trop tôt** sur une partie des winners qui auraient atteint plus (mais rares, peak_r p60=0.66).

### 3.4 Case §20 classification
**Cas C dominant (edge absent)** sur le playbook globalement :
- Signal exercé correctement (n=38, peak_r p60 cohérent).
- TP peak-R optimisé au plafond observé (p80≈TP1).
- Asymétrie LONG/SHORT ne laisse aucun universel.
- **Régression** vs best prior C.1 : non seulement on ne franchit pas zéro, on fait pire.
- QQQ SHORT subset +0.016 ne croise **pas** user bar (+0.02 minimum, +0.05 Stage 1, +0.10 paper). Itérer pour stabiliser un quasi-BE = **explicitement rejeté** par feedback_real_results_bar.md.

**Pas de Cas B légitime** : l'hypothèse pièce B (excursion 0.83R exploitable via TP1 calibré) est **falsifiée numériquement** — l'excursion réelle est 0.66R p60, et même en la capturant, WR 34 % reste en dessous de BE 54.6 %. Signal Morning_Trap structurellement incapable de WR cohérent après 4 itérations (B2 BE, C.1 vwap_regime, M baseline, TP peak-R v1).

**Pas de Cas D (codage faux)** : toutes les briques (candlestick detectors, liquidity_sweep, vwap_regime, SL SWING, TP fixed_rr, BE, trailing) sont validées engine_sanity_v2 33/33. Le codage exécute fidèlement l'hypothèse — l'hypothèse est réfutée.

---

## Bloc 4 — Décision

**KILL définitif** → Leg 2 (Quarantine PROMOTE corpus expansion : IFVG_5m_Sweep / VWAP_Bounce_5m / HTF_Bias_15m_BOS) per §0.5 arbre.

### Kill rules pré-écrites (pièce H) atteintes

| Kill rule | Valeur observée | Seuil | Atteinte ? |
|---|---|---|---|
| `E[R]_net ≤ 0` | **−0.1321** | ≤ 0 | ✅ **ATTEINTE** |
| `WR < 40 %` | 34.2 % | < 40 % | ✅ **ATTEINTE** |
| `PF < 1.0` | **0.265** | < 1.0 | ✅ **ATTEINTE** |

**3/3 kill rules atteintes** — maximum. Sortie : KILL définitif + progression Leg 2.

### Gate Stage 1 (§0.6) — évaluation complète pour référence
| Gate | Valeur | Seuil | Passe ? |
|---|---|---|---|
| E[R]_net > 0.05R | −0.1321 | > 0.05R | ❌ |
| n ≥ 15 | 38 | ≥ 15 | ✓ |
| peak_R p60 > 0.5R | 0.661 | > 0.5R | ✓ |
| 0 weeks < −0.5R | 2 weeks < −0.5R (jun_w3 -2.38, oct_w2 -3.22) | 0 | ❌ |
| Split régime §0.4-bis | 1/4 weeks clearly positive ; SPY/QQQ ×63× gap ; LONG/SHORT signe opposé ; QQQ SHORT seul subset positif mais +0.016 < bar | cohérent ≥3 régimes | ❌ |

Gate Stage 1 **FAIL** sur 3/5 critères. Pas de passage Stage 2.

---

## Bloc 5 — Why

### Pourquoi cette décision est rationnelle

1. **Hypothèse falsifiable (pièce B) réfutée** : le mécanisme postulé (peak_R 0.83R exploitable via TP1 calibré) est **contredit** par les données 4w canonical — peak_r p60 réel 0.661 (vs 0.827 survivor_v1). La calibration overshot l'excursion médiane → TP1 proche de p80 = plafond structurel.
2. **WR 34.2 % vs BE 54.6 %** : déficit 20 pts. Pas de room pour stabiliser même en filtrant agressivement (QQQ SHORT only = +0.016R, sous-bar).
3. **Régression vs best prior C.1** (E[R] −0.081 → −0.132) : la calibration TP peak-R **empire** les choses. Morning_Trap a épuisé tous ses leviers :
   - B2 BE patch : -0.147 → -0.123 (+0.024)
   - C.1 vwap_regime : -0.123 → -0.081 (+0.042)
   - M baseline : -0.081 → +0.003 gross / -0.062 net (régime-dépendant)
   - **v1 TP peak-R : +0.003 → -0.132 (régression -0.135)**
4. **§19.3 budget dépassé** : 4 itérations au compteur, exception CLAUDE.md user-marked consommée. "1 dernier levier avant KILL" — levier consommé, résultat FAIL, KILL exécuté.

### Pourquoi on n'itère pas plus

- **Pas de Cas B légitime** : le signal est exercé (n=38, peak_r p60 observé), le schéma de sortie fait ce qu'il doit (TP1 hit ×3 vs baseline). L'edge manque, point.
- **QQQ SHORT subset (n=16, E[R]=+0.016)** : marginal sous-bar. User feedback_real_results_bar.md : "un edge marginal n'est pas un edge, c'est du noise positif". Itérer pour stabiliser = **explicitement rejeté** par le repo.
- **Plafond peak_r p80=0.891R ≈ TP1=0.83R** : pas de levier TP2 restant (déjà calibré 1.50R = bien au-dessus p80). Augmenter TP1 = retour à la pathologie initiale (TP1 3.0R unreachable).
- **4 itérations consommées** : B2, C.1, M, v1. §19.3 budget 3+1 exception épuisé. §10 réouverture branche morte interdite sans hypothèse structurellement nouvelle.
- **Asymétrie LONG/SHORT non-filtrable** : tuner `direction: short` only sur Morning_Trap = fit post-hoc sur régime uptrend 2025. Même asymétrie détectée sur Engulfing Leg 1.1 → pas spécifique à Morning_Trap, biais de marché global 2025.

### Pourquoi on ne tue pas trop tôt
On tue avec 4 itérations documentées cumulant régression (best prior -0.081, actuel -0.132). CLAUDE.md user-marked explicitement que cette calibration était **le dernier levier** avant KILL. Le verdict exécute l'instruction user.

### Pourquoi on ne promeut pas trop tôt
Aucun des 5 critères gate Stage 1 (§0.6) n'est rempli collectivement. 3/3 kill rules atteintes. Promotion interdite par règle absolue §10 + kill rules pré-écrites pièce H.

---

## Impact 5-classes post-Leg 1.2

| Classe | Before Leg 1.2 | After Leg 1.2 |
|---|---|---|
| IMPROVE | 1 (Morning_Trap 4e levier) | **0** — Morning_Trap → ARCHIVED/KILL |
| ARCHIVED post-smoke/4w | 3 (Aplus_01 + Stat_Arb + Engulfing) | **4** (+ Morning_Trap) |

---

## Suite — §0.5 Leg 2 Quarantine PROMOTE corpus expansion

Progression automatique. Leg 1 épuisée (1.1 Engulfing ARCHIVED + 1.2 Morning_Trap KILL terminal).

**Leg 2 candidats** (CLAUDE.md "quarantine" E[R]>0 sur n=3-11, besoin data pas calibration) :
- **2.1 IFVG_5m_Sweep** solo 12 semaines (extension corpus jun-nov 2025)
- **2.2 VWAP_Bounce_5m** solo 12 semaines
- **2.3 HTF_Bias_15m_BOS** solo 12 semaines

Protocole : Caps actives, IdealFillModel, allowlist solo chacun. Score après 3 runs : pick max E[R]_gross avec n ≥ 20. Kill rule cohort : si 0/3 passe E[R] > 0.05R gross + n ≥ 20 → Leg 3 (Aplus_02 Family F Premarket).

**Note corpus** : Leg 2 demande 12 semaines par playbook (corpus expansion). Disponibilité corpus local jun-nov 2025 = ~24 semaines × 2 symbols disponibles. Pas d'escalade User requise (pas de Polygon 18m).

**Préparer next session** : dossier §18 pour `ifvg_5m_sweep_solo_12w` (pièce E = allowlist-solo YAML override baseline IFVG_5m_Sweep, 12 semaines jun-nov). Ou alternative : aborder Leg 3 directement si estimation que 3 quarantine candidats convergeront négatif (vu signaux vocab-borrowing Phase D.2 audit 0/7 MASTER faithful).
