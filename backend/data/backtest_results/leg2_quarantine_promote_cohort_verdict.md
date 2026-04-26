# Verdict — Leg 2 Quarantine PROMOTE corpus expansion (3 × 12w solo)

**Source** : CEO arbre §0.5 Leg 2
**Date** : 2026-04-22
**Playbooks** : IFVG_5m_Sweep, VWAP_Bounce_5m, HTF_Bias_15m_BOS
**Dossiers** : [ifvg](../../knowledge/playbooks/ifvg_5m_sweep_solo_12w/dossier.md) · [vwap](../../knowledge/playbooks/vwap_bounce_5m_solo_12w/dossier.md) · [htf](../../knowledge/playbooks/htf_bias_15m_bos_solo_12w/dossier.md)
**Verdict structure** : §18.3 (5 niveaux par playbook + synthèse cohorte)

---

## Bloc 1 — Identité des runs

| Champ | Valeur commune |
|---|---|
| Corpus | 12 semaines jun-nov 2025 local parquet 1m (2x instruments SPY+QQQ) |
| Weeks | jun_w2, jun_w3, jul_w2, jul_w4, aug_w1, aug_w3, sep_w2, sep_w4, oct_w1, oct_w2, nov_w2, nov_w4 |
| Mode | AGGRESSIVE + ALLOW_ALL_PLAYBOOKS + `--calib-allowlist <playbook>` (isole chaque solo) |
| Caps | `--no-relax-caps` (cooldown 5min + session_cap actifs) ; kill-switch par défaut |
| Fill model | Ideal (baseline — slippage budget -0.065R/trade noté pour promotion éventuelle) |
| Baseline YAML | Chaque playbook **as-is** (pas d'override) — per plan §0.5 "besoin data, pas calibration" |
| Git SHA | (staged — commit post-verdict) |

---

## Bloc 2 — Métriques agrégat 12 semaines

### IFVG_5m_Sweep (Leg 2.1)

| Métrique | Valeur |
|---|---|
| n | 118 |
| WR | 39.8 % |
| E[R] gross | **−0.071** |
| PF | 0.512 |
| total_R | −8.38R |
| peak_r p50/p60/p80 | 0.454 / 0.689 / 1.123 |
| mae_r p20 | −1.059 |
| Exits | SL 82 (69.5 %) / eod 35 (29.7 %) / TP1 **1 (0.8 %)** |
| Weeks < −0.5R | **6/12** |

**Per-week** : aug_w1 +1.37R (WR 70 %), aug_w3 +0.88R (70 %), jul_w4 +0.97R (50 %), nov_w4 +0.44R (70 %) = 4 positives · jul_w2 −1.09R, jun_w2 −1.98R, jun_w3 −1.89R, nov_w2 −4.20R, oct_w1 −0.49R, oct_w2 −1.67R, sep_w2 −0.02R, sep_w4 −0.70R = 8 negatives.

**Splits** :
- LONG n=86 WR 37.2 % E[R]=**−0.090** / SHORT n=32 WR 46.9 % E[R]=−0.021
- SPY n=55 WR 36.4 % E[R]=**−0.095** / QQQ n=63 WR 42.9 % E[R]=−0.050
- **TP 3.0R structurellement inatteignable** : peak_r p80=1.123 << TP1=3.0 ; 1 TP hit sur 118 trades.

### VWAP_Bounce_5m (Leg 2.2)

| Métrique | Valeur |
|---|---|
| n | 36 |
| WR | 41.7 % |
| E[R] gross | **−0.068** |
| PF | 0.425 |
| total_R | −2.45R |
| peak_r p50/p60/p80 | 0.214 / 0.456 / 1.509 |
| mae_r p20 | −0.585 |
| Exits | time_stop 22 (61.1 %) / TP1 8 (22.2 %) / SL 6 (16.7 %) |
| Weeks < −0.5R | 4/11 (jun_w3 silent — 0 trades) |

**Per-week** : aug_w3 +0.52R (WR 60 %), jun_w2 +0.21R (100 %, n=2), nov_w4 +0.14R (100 %, n=1), sep_w2 +0.05R, sep_w4 +0.17R = 5 positives · aug_w1 −0.41R, jul_w2 −0.55R, jul_w4 −0.79R, nov_w2 −0.32R, oct_w1 −0.56R, oct_w2 −0.90R = 6 negatives.

**Splits** :
- LONG n=31 WR 38.7 % E[R]=−0.088 / SHORT n=5 WR 60 % E[R]=+0.056 (n trop bas pour conclure)
- SPY n=22 WR 36.4 % E[R]=−0.096 / QQQ n=14 WR 50 % E[R]=−0.023
- **max_duration 40m cap binding** : 61 % des exits sont time_stop — signal ne converge pas assez vite dans la fenêtre allouée. TP 1.5R atteint 22 % (8/36) ≠ pathologie IFVG (TP 3.0R 0.8 %).

### HTF_Bias_15m_BOS (Leg 2.3)

| Métrique | Valeur |
|---|---|
| n | 58 |
| WR | 53.4 % |
| E[R] gross | **−0.0357** |
| PF | 0.615 |
| total_R | −2.07R |
| peak_r p50/p60/p80 | 1.056 / 1.267 / 1.522 |
| mae_r p20 | −1.065 |
| Exits | SL 54 (93.1 %) / eod 4 (6.9 %) / **TP1 0** |
| Weeks < −0.5R | **1/12** (jun_w3 seule < −0.5R à −0.83R) |

**Per-week** : aug_w1 +0.13R (WR 60 %), aug_w3 +0.17R (80 %), nov_w4 +0.51R (75 %) = 3 positives · 9 negatives entre 0 et −0.83R.

**Splits** :
- LONG n=42 WR 47.6 % E[R]=−0.070 / SHORT n=16 WR **68.8 %** E[R]=**+0.055**
- SPY n=17 WR 41.2 % E[R]=−0.099 / QQQ n=41 WR **58.5 %** E[R]=−0.009 (quasi-BE)
- **QQQ SHORT subset** : n=13, WR **76.9 %**, E[R]=**+0.076**, total_R=+0.99R
- **TP 3.0R inatteignable** : peak_r p80=1.522 → 0 TP hits sur 58 trades malgré p60=1.267 (le plus haut des 3). SL 1R atteint sur 54/58 (93.1 %).

---

## Bloc 3 — Lecture structurelle

### 3.1 Volume de signaux (hypothèse "besoin data pas calibration" validée ?)

Per dossier pièce B : l'hypothèse Leg 2 était que le corpus 4w (n=3-11 des quarantines CLAUDE.md) était trop étroit pour conclure — 12 semaines = ~3×.

**Résultat** :
- IFVG volume-rich (n=118, ~10/semaine) — hypothèse "data-starved" **réfutée** : le signal existe, l'edge est absent.
- VWAP sparse (n=36, ~3/semaine) — signal **structurellement rare** malgré time_windows larges.
- HTF intermédiaire (n=58, ~5/semaine) — densité raisonnable, signal existe.

Le problème n'est pas le manque de data. C'est **l'edge absent** (IFVG, HTF) OR **le signal trop rare pour être calibré** (VWAP).

### 3.2 Pathologie commune : TP fixed RR vs peak_r distribution

**3 / 3 playbooks** ont TP fixed RR **≥ 1.5×** peak_r p80 — même problème que Engulfing Leg 1.1, Morning_Trap Leg 1.2, Aplus_03 R.3, Aplus_04 Option B, et tous playbooks 4w précédents :

| Playbook | TP1 (RR) | peak_r p80 | Ratio | TP hit % |
|---|---:|---:|---:|---:|
| IFVG_5m_Sweep | 3.0 | 1.12 | 2.7× | 0.8 % |
| VWAP_Bounce_5m | 1.5 | 1.51 | 0.99× | 22.2 % |
| HTF_Bias_15m_BOS | 3.0 | 1.52 | 1.97× | 0 % |

VWAP est le seul où TP1 ≈ p80 — et c'est le seul qui capture TP (22 %). Mais sans edge positif quand même (TP capture ≠ E[R] positif, parce que losers absorbent 1R plein).

### 3.3 Asymétrie 2025 uptrend — cross-playbook consistant

**5e confirmation cross-playbook** du pattern LONG-toxique / SHORT-meilleur (précédents : Engulfing Leg 1.1, Morning_Trap Leg 1.2, Aplus_04_v1 Option B, Aplus_03_v1) :

| Playbook | LONG E[R] | SHORT E[R] | Δ |
|---|---:|---:|---:|
| IFVG_5m_Sweep | −0.090 | −0.021 | +0.069 |
| VWAP_Bounce_5m | −0.088 | +0.056 (n=5) | +0.144 |
| HTF_Bias_15m_BOS | −0.070 | **+0.055** | +0.125 |
| Engulfing (rappel) | −0.116 | −0.059 | +0.057 |
| Morning_Trap (rappel) | **−0.462** | +0.002 | +0.464 |

**Et QQQ SHORT subset** est systématiquement le meilleur quadrant :

| Playbook | QQQ SHORT E[R] | n | WR |
|---|---:|---:|---:|
| Engulfing Leg 1.1 | −0.008 | 14 | 71.4 % |
| Morning_Trap Leg 1.2 | +0.016 | 16 | 56.2 % |
| HTF_Bias_15m_BOS Leg 2.3 | **+0.076** | 13 | **76.9 %** |

HTF QQQ SHORT est le meilleur subset de **toute la campagne 2026-04-22** (±0.08R, WR 77 %). **Mais** :
- **n=13 < n≥15 gate Stage 1** §0.6 (E[R] > 0.05R + **n ≥ 15** + peak_R p60 > 0.5R + 0 weeks < -0.5R + split régime)
- Post-hoc filtering single-direction × single-instrument = tuning sur subset = violation §10 règle 11 "pas de réouverture sauf hypothèse structurellement nouvelle" — cf Engulfing Leg 1.1 QQQ SHORT rejeté explicite
- E[R]=+0.076 < user bar promotion +0.10R (feedback_real_results_bar.md)

→ **QQQ SHORT pattern est réel mais non-promotable unitairement**. Pourrait justifier une **hypothèse cross-playbook portfolio** (cohort QQQ-SHORT-only des 3 playbooks Leg 2) comme piste future — mais hors scope Leg 2.

### 3.4 Case §20 classification

| Playbook | Cas dominant | Justification |
|---|---|---|
| IFVG_5m_Sweep | **Cas C** (edge absent) | n=118 volume ok, 1 TP hit / 118, 6/12 weeks < -0.5R, LONG catastrophique |
| VWAP_Bounce_5m | **Cas B** (sous-exercé) + **C secondaire** | n=36 sur 12w sparse, 61 % time_stop = signal ne converge pas, n insuffisant pour conclure edge. Bar n≥20 atteint mais étroit. |
| HTF_Bias_15m_BOS | **Cas C** (edge absent) | n=58 ok, 0 TP hits / 58, SL 93 %, LONG toxique |

---

## Bloc 4 — Décision

### Gate Leg 2 cohort (plan §0.5)

> **Kill rule cohort** : si 0/3 passe E[R] > 0.05R gross + n ≥ 20 → Leg 3

| Playbook | E[R] gross | n | Pass ? |
|---|---:|---:|:---:|
| IFVG_5m_Sweep | −0.071 | 118 | ❌ E[R] < 0 |
| VWAP_Bounce_5m | −0.068 | 36 | ❌ E[R] < 0 |
| HTF_Bias_15m_BOS | −0.036 | 58 | ❌ E[R] < 0 |

**0/3 passe → Gate cohort FAIL → progression automatique Leg 3 Aplus_02 Family F Premarket** per §0.5.

### Par playbook

- **IFVG_5m_Sweep** → **ARCHIVED** (2/3 kill rules individuelles atteintes : n<20 ❌ mais n=118, E[R]≤0 ✓, Cas C)
- **VWAP_Bounce_5m** → **ARCHIVED** (E[R]≤0 ✓, Cas B+C secondaire)
- **HTF_Bias_15m_BOS** → **ARCHIVED** (E[R]≤0 ✓, Cas C ; QQQ SHORT subset noted mais non-promotable)

### Gate Stage 1 (§0.6) — non-atteint pour aucun

| Gate | IFVG | VWAP | HTF |
|---|:---:|:---:|:---:|
| E[R]_net > 0.05R | ❌ | ❌ | ❌ |
| n ≥ 15 | ✓ | ✓ | ✓ |
| peak_R p60 > 0.5R | ✓ | ❌ (0.46) | ✓ |
| 0 weeks < −0.5R | ❌ (6) | ❌ (4) | ❌ (1) |
| Split régime cohérent | ❌ LONG toxique | ❌ direction biais | ❌ LONG toxique |

---

## Bloc 5 — Why

### Pourquoi ARCHIVED, pas itération

1. **Hypothèse "besoin data pas calibration" réfutée** : le corpus 12w ne change pas le signe de l'edge. IFVG volume-rich, HTF volume-moyen — E[R] reste négatif. Le signal + la TP structure actuels ne portent pas d'edge.
2. **TP fixed RR plafonne 2/3 playbooks** : IFVG TP 3.0R vs p80=1.12 et HTF TP 3.0R vs p80=1.52 = unreachable. Mais "fixer le TP" = calibration = exactement ce qui a déjà été testé sur Engulfing/Morning_Trap → 2/2 ARCHIVED. Pas itération ici.
3. **Cross-playbook asymétrie LONG/SHORT 2025** confirmée 5e fois. Le marché 2025 tendu hausse rend contrarian LONG toxique. Pas un paramètre tunable — paramètre de régime qui demande un filtre structurel (regime_classifier §0.7, tech debt parallèle).
4. **§10 règle 11 "réouverture branche morte interdite"** : tuner single-direction + single-instrument (QQQ SHORT subset HTF) = post-hoc. Rejeté systématiquement (Engulfing QQQ SHORT déjà rejeté même pattern).

### Pourquoi Leg 3 (pas Leg 4 stat-arb v2 / Leg 5 escalade)

Per §0.5 arbre, Leg 2 FAIL → Leg 3 Aplus_02 Family F Premarket (6e et dernière MASTER family non-testée). C'est la route fixe, pas de skip. Leg 4 (non-MASTER quant v2) vient après si Leg 3 FAIL.

### Pourquoi on ne tue pas les 3 playbooks (DENYLIST au lieu d'ARCHIVED)

ARCHIVED = verdict + dossier préservés, YAML intact dans `playbooks.yml`. Pas de DENYLIST-level interdiction. Ils pourraient re-devenir pertinents si :
- Un filtre régime §0.4-bis futur isole un subset (ex VIX band, day-of-week) structurellement différent
- Un overlay cross-playbook cohort (QQQ-SHORT-only pattern observé) démontre edge sur cohort
- Hypothèse structurellement nouvelle (nouvelle TP logic basée sur peak_r distribution, nouveau timeframe confirmation)

Ces options restent ouvertes comme **§10 réouverture légale** seulement si l'hypothèse nouvelle est formelle et différente de "baseline re-tuned".

### Pourquoi on ne promeut aucun subset (QQQ SHORT HTF)

- n=13 < n≥15 gate Stage 1 (technique)
- E[R]=+0.076 < user bar +0.10R promotion (feedback_real_results_bar.md)
- Post-hoc filtering violation §10 règle 11
- Pattern cross-playbook (3 playbooks confirment) = meta-hypothèse régime, pas single-playbook edge

**Note pour référence future** : le pattern QQQ SHORT 2025 est une **méta-observation** worth documenting. Si Leg 5 escalade user, ce pattern pourrait justifier une nouvelle hypothèse portfolio-level (ex "QQQ-SHORT-cohort-only avec regime_classifier §0.7 VIX band filter"). Mais ce n'est pas un playbook Leg 2.

---

## Suite — §0.5 Leg 3 Aplus_02 Family F Premarket

Progression automatique. Per §0.5 :

> Leg 3 — Aplus_02 Family F Premarket (coût ~1-2j infra + smoke)
> 6e et dernière MASTER family non-testée.
> **3.1** Session YAML 04:00-09:30 ET — extension [session_range.py](backend/engines/session_range.py) + alias [playbook_loader.py:611](backend/engines/playbook_loader.py#L611)
> **3.2** Dossier §18 (8 pièces obligatoire avant code)
> **3.3** Briques réutilisables Sprint 1 — confluence_zone + pressure_confirm + Aplus01Tracker template preserved
> **3.4** YAML playbook — schéma α'' (liquidity_draw swing_k3 significant + structure_alignment k3)
> **3.5** Tests §18 pièce F (unitaires + intégration)
> **3.6** Smoke premarket nov_w4

**Kill rules template Sprint 1** : n<10 OR peak_R p80 < 1R OR E[R] gross ≤ 0 → ARCHIVED + Leg 4.

Infra partielle présente (confluence_zone + pressure_confirm + Aplus01Tracker Sprint 1 preserved). Chantier principal : session YAML premarket 04:00-09:30 ET (extension session_range.py).
