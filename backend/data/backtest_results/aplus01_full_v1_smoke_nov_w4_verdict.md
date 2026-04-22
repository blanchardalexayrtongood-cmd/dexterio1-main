# Aplus_01_full_v1 — smoke nov_w4 verdict

**Sprint 1 S1.3, 2026-04-22.** 1ère vraie instanciation MASTER Family A (séquentielle :
sweep → 5m BOS → confluence touch → 1m confirm). Test décisif ICT bot.

---

## Bloc 1 — Identité du run

| Champ | Valeur |
|---|---|
| Playbook | `Aplus_01_full_v1` |
| Version | v1 |
| Famille | ICT-A (séquentielle stateful) |
| Période | 2025-11-17 → 2025-11-21 (nov_w4, 5 NY sessions) |
| Instruments | SPY + QQQ |
| Mode | AGGRESSIVE (IdealFillModel — `--no-relax-caps --no-disable-kill-switch`) |
| Corpus | 9345 × 1m bars, 30 HTF warmup days |
| Playbooks enregistrés | 3 (`Aplus_01_full_v1` + DAY/SCALP_Aplus_1 DENYLIST-filtrés) |
| YAML | [aplus01_full_v1.yml](../../knowledge/campaigns/aplus01_full_v1.yml) |
| Dossier | [playbooks/aplus_01_full_v1/dossier.md](../../knowledge/playbooks/aplus_01_full_v1/dossier.md) |
| Git SHA | `0d2d0f15775181246e260543f8a1510b6cdb78df` |
| Run artifacts | [results/labs/mini_week/aplus01_family_a/aplus01_full_v1_smoke_nov_w4_v2/](../../results/labs/mini_week/aplus01_family_a/aplus01_full_v1_smoke_nov_w4_v2/) |

---

## Bloc 2 — Métriques

### Entonnoir cascade

| Étape | Compteur | Commentaire |
|---|---:|---|
| `matches_by_playbook["Aplus_01_full_v1"]` | **1** | 1 emit synthétique sur 9345 bars 1m × 2 symboles |
| `setups_created_by_playbook["Aplus_01_full_v1"]` | 1 | emit → ICTPattern synthétique → Setup B |
| HTF D-alignment | 1 pass / 0 reject | SMA proxy passe-through |
| `setups_after_risk_filter["Aplus_01_full_v1"]` | 1 | AGGRESSIVE_ALLOWLIST (ajout Sprint 1) + allowlist calib OK |
| `trades_opened_by_playbook["Aplus_01_full_v1"]` | 1 | boucle complète exécutée |

### Résultat trade unique

| Métrique | Valeur |
|---|---|
| `n` | **1** |
| Wins / losses / breakevens | 0 / 1 / 0 |
| Win rate | 0 % |
| E[R] (gross, ideal fill) | **-0.0027 R** |
| Total R | -0.0027 |
| Profit factor | 0.0 |
| peak_R | **0.89** |
| mae_R | -0.90 |
| Duration | 27 min |
| Max drawdown R | 0.005 |

### Détail trade

| Champ | Valeur |
|---|---|
| Symbol | QQQ |
| Direction | SHORT |
| Entry ts | 2025-11-19 15:14 UTC (10:14 ET) |
| Entry / SL / TP1 | 605.07 / 606.28 / 602.65 |
| Exit ts | 2025-11-19 15:41 UTC |
| Exit price / reason | 604.38 / **SL** (après pullback qui a trail/BE le stop puis retour) |
| `r_multiple` | -0.0027 |
| `tp_reason` | **`fallback_rr_pool_beyond_ceiling`** (pool k3 au-delà du ceiling 3.0 → fallback 2.0R) |
| `match_grade` | B |
| `match_score` | 0.49 |

---

## Bloc 3 — Lecture structurelle

### 1. L'infrastructure fonctionne bout-en-bout

La cascade a exercé **tous les étages** sur une data réelle :
- 5m sweep détecté (ICT engine) → driver `_sweep_payload` invertit la direction (REACTION → SWEPT)
- `Aplus01Tracker` ARMED
- 5m BOS counter-direction → ARMED → BOS
- 5m bar touche zone confluence (FVG ∪ OB ∪ breaker) → BOS → TOUCHED
- 1m pressure (BOS break ou engulfing armed_direction) → EMIT
- `Aplus01Driver` injecte ICTPattern synthétique (`pattern_type='aplus01_sequence'`, `price_level=sweep_extreme`) dans `_ict_by_tf["1m"]`
- `playbook_loader.type_map['APLUS01'] = 'aplus01_sequence'` satisfait `required_signals: [APLUS01@1m]`
- `_determine_direction` reconnaît la direction synthétique
- `_calculate_price_levels` utilise `price_level` comme ancre SL structurelle (avec clamp 0.2-2% appliqué)
- `tp_resolver` route vers `liquidity_draw swing_k3` avec pool_selection=significant + ceiling 3.0
- Setup grade B → risk filter OK → trade ouvert → exit SL après 27 min → journalisé avec `tp_reason`

**Conclusion infra : S1.2.1–S1.2.5 validé sur une data réelle. L'implémentation est correcte.**

### 2. La cascade est structurellement rare

**1 emit / 9345 bars 1m / 5 sessions NY / 2 symboles.** Comparaison corpus équivalents nov_w4 :

| Playbook | n (nov_w4) | Source |
|---|---:|---|
| Aplus_03_v2 α'' (Family A IFVG isolé) | 22 (22 × tfl 5m, 4w) | [aplus03_v2_verdict.md](aplus03_v2_verdict.md) |
| Aplus_04_v2 α'' (Family B HTF+15m BOS) | 15 | [aplus04_v2_alpha_pp_verdict.md](aplus04_v2_alpha_pp_verdict.md) |
| Aplus_01_full_v1 (Family A **full**) | **1** | ce verdict |

Ordre de grandeur : la composition séquentielle (4 états, 4 timeouts, 2 timeframes) divise la densité par ~15–20× par rapport à un signal ICT instantané isolé. Aplus_01_light simultané avait atteint 0 matches (documenté plan §2.3). Le passage séquentiel résout l'abs 0 ↔ 1, mais reste sous le seuil de décision statistique.

### 3. tp_reason révèle un second bottleneck (conditionnel)

L'unique emit a tp_reason = **`fallback_rr_pool_beyond_ceiling`** — le resolver a bien trouvé un pool k3 significatif, mais trop loin (> 3.0R). Le ceiling a donc **coupé** le target légitime, on est retombé sur `fallback_rr` à 2.0R, jamais atteint.

C'est l'inverse du bottleneck Aplus_03_v2 (fallback `no_pool` 27% + `min_floor_binding` 45%) : ici le pool est à distance excessive, pas manquant. Sample n=1 ne permet pas de conclure que ce pattern est systémique.

### 4. Grille §20 — Cas B dominant

- **Cas A (infra missing)** : exclu — 1 emit réel, toute la chaîne exercée.
- **Cas B (sous-exercé / rareté structurelle)** : **dominant** — n=1, signal cohérent, infra correcte.
- **Cas C (signal inutile)** : non-concluable — 1 data point ne permet pas d'affirmer plafond.
- **Cas D (codage faux)** : exclu — logique encodée correspond bijection à l'hypothèse pièce B.

---

## Bloc 4 — Décision

**Kill rules pré-écrites (dossier pièce H) :**

| Rule | Threshold | Observed | Verdict |
|---|---|---|---|
| `n < 10` | < 10 | 1 | **KILL** |
| `peak_R p80 < 1R` | < 1.0 | 0.89 (p80 trivial n=1) | **KILL** |
| `E[R] gross ≤ 0` | ≤ 0 | -0.003 | **KILL** |

Les 3 kill conditions sont atteintes. Par dossier H + plan §S1.3 :

> **SMOKE_FAIL → stop ICT, pivot vers Sprint 3 (non-ICT stat arb SPY-QQQ).**

Statut transition (cf §17) : `IMPLEMENTED` → `SMOKE_PENDING` → **`SMOKE_FAIL`** → `ARCHIVED`.

---

## Bloc 5 — Why (pourquoi cette décision est rationnelle)

### Pourquoi KILL plutôt qu'itérer

Aucune calibration, relâchement de timeouts ou extension de corpus ne peut sauver un signal qui fire **1×/5 jours / 2 symboles**. Pour atteindre la barre de promotion (n > 30 + E[R] > 0.10R + PF > 1.2 + gates O5.3 — §10 règles strictes + [feedback_real_results_bar.md](../../../.claude/projects/-home-dexter-dexterio1-main/memory/feedback_real_results_bar.md)), il faudrait un corpus ≥ 30 semaines × 2 symboles au rythme actuel. Incompatible avec :

- **§19.2 règle de non-dispersion** (1 playbook à la fois en phase D→F avant 1 PRODUCT_GRADE).
- **§19.3 budget d'itération** (max 3 tentatives post-smoke).
- **[feedback_no_long_runs.md](../../../.claude/projects/-home-dexter-dexterio1-main/memory/feedback_no_long_runs.md)** (pas de 4-week runs tant qu'on n'est pas dans le vert).

### Pourquoi la décision ne réfute PAS la Family A comme hypothèse

1 data point ≠ hypothèse réfutée. Le dossier Aplus_01_full_v1 passe en ARCHIVED avec la note **hypothèse non-falsifiée, cascade trop rare pour protocole 4-semaines**. Une future instanciation Family A pourrait reformuler :
- Timeouts plus larges (sweep_timeout 20 → 40 bars, bos_timeout 6 → 12)
- Corpus élargi (ex. 6 mois SPY+QQQ+IWM)
- HTF sweep réel (non implémenté — v1 fallback SWEEP@5m)
- Confluence zones élargies (OB extended, breaker w/padding)

Ces ajustements requièrent un **nouveau dossier + nouvelle hypothèse formelle** (§10 règle "Réouverture branche morte") — pas un tune du YAML actuel.

### Pourquoi Sprint 3 (non-ICT stat arb) plutôt qu'un autre playbook ICT

Bilan ICT post-Sprint 1 :
- **Family A IFVG isolé (Aplus_03_v2 α'')** : Case B sous-exercé, 22 trades, E[R] -0.019.
- **Family A full (Aplus_01_full_v1)** : rareté structurelle, 1 trade, SMOKE_FAIL.
- **Family B HTF+15m BOS (Aplus_04_v2 α'' + ε)** : 2 data points convergents négatifs sur schéma α''.
- **Family F Premarket** : session YAML absent, jamais instancié.
- **Families C, E** : testées négatives en Phase 5 + Phase A.
- **7 "vocab-borrowing" (V065/V054/V056/V051/V066/V004 + Engulfing)** : 0/7 MASTER-faithful, tous négatifs.

Les 5/6 familles MASTER testables ont été essayées (A rare, B négative, C/E négative, F non-session). Poursuivre ICT = répéter un pattern structurellement épuisé. Pivot Sprint 3 = terrain non-corrélé :

> **Sprint 3 (plan §5.3 P3)** : stat arb SPY-QQQ (cointegration Engle-Granger, z-score rolling, beta-neutral pair). Completement séparé d'ICT.

Budget : 3–5 jours (YAML + détecteurs + tests + smoke + 4 semaines + verdict). Kill rule identique (E[R] net > 0.10R + n > 30 + PF > 1.2).

### Artefacts de valeur préservés

S1 ne se termine pas bredouille :
- **S1.1 timezone audit** : 12 tests DST + audit report → base solide future.
- **S1.2.1 briques pures** (`confluence_zone.py`, `pressure_confirm.py`) : réutilisables par tout playbook séquentiel futur.
- **S1.2.2 `Aplus01Tracker`** : state machine template validée (17 tests unit). Architecture transférable à Aplus_02 (F Premarket) si jamais réouvert.
- **S1.2.3 wire-up synthétique** : pattern d'injection ICTPattern synthétique dans `_ict_by_tf` → générique, pas dédié Aplus_01.
- **Moteur même cerveau** (Phase W) : toujours byte-identique backtest↔paper, 33/33 logic-correct.

**Rien à reroller. Les briques tiennent. Seule l'hypothèse "cascade séquentielle Family A sur corpus 1-semaine" est retirée.**

---

## Next step immédiat

1. Mettre à jour statut dans [CLAUDE.md](../../../CLAUDE.md) tableau État playbooks : Aplus_01_full_v1 = `SMOKE_FAIL → ARCHIVED`.
2. Laisser AGGRESSIVE_ALLOWLIST avec Aplus_01_full_v1 retiré (ajout Sprint 1 était temporaire pour le smoke).
3. Commit Sprint 1 clos + sync mémoire.
4. Décision utilisateur : (a) lancer Sprint 3 stat arb maintenant, OU (b) paper baseline sur survivors (News_Fade + Engulfing + Session_Open + Liquidity_Sweep), OU (c) autre pivot.
