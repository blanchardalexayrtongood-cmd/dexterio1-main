# Phase M — diagnostic honnête pré-run (2026-04-21)

## Question de fond (posée par user)

> Est-ce que certaines stratégies actuelles ne devraient pas être améliorées, mais carrément reconstruites proprement à partir du MASTER / vidéos / logique de marché, parce que leur version actuelle est trop déformée ou trop compressée ?

**Réponse : OUI, explicitement.** Preuves repo-backées :

- **Phase D.2** ([tf_faithfulness_audit_v1_verdict.md](tf_faithfulness_audit_v1_verdict.md)) : 0/7 "MASTER faithful" (V065/V056/V054/V051/V066/V004 + Engulfing) n'enforcent **réellement** la mécanique MASTER. 0/7 enforcent D/4H bias, 0/7 utilisent liquidity-targeting TPs, 0/7 utilisent 1m confirmation. Du vocabulaire emprunté, pas de la mécanique.
- **3/6 familles MASTER (A, B, F) ne sont jamais instanciées** — Aplus_01/02/04 sont `research_only`. Aplus_03 v1 (premier test fidèle Family A) : E[R] -0.074 avec 0/47 TPs hit à 2.0R alors que peak_R p80=1.06R. Signal non-null, TP inatteignable.
- **Phase C.1** ([c1_vwap_verdict.md](c1_vwap_verdict.md), [c1_lsweep_verdict.md](c1_lsweep_verdict.md)) : stacker des filtres post-entrée plafonne à E[R]<0. Signal asymptote. Un filtre supplémentaire coûte du sample sans bouger la math.

## Phase M = falsification test, pas save attempt

Recette S1 (`require_close_above_trigger: true` + `entry_buffer_bps: 2.0` + `require_htf_alignment: D` pour continuations) vient de la SEULE victoire product-grade : Engulfing_Bar_V056 (net +0.020R, n=38, WR 57.9%). C'est un stack validé empiriquement **sur un signal structurel propre** (engulfing bar = pattern objectif bien défini).

**Ce que S1 va probablement faire** sur les 10 candidats :
- Cut 30-60% des trades → augmenter WR de 3-8 points
- Ne PAS réparer : TPs fixed RR inatteignables (peak_R < TP), détecteurs structurellement défectueux, session windows cassées, signaux sans edge structurel
- Révéler **quels playbooks ont un signal latent** (S1 les pousse sur E[R]>0) vs **quels ont besoin de REWRITE/RECREATE** (S1 les laisse <0)

## Pré-diagnostic 5-classes par candidat M

| # | Playbook | Classe actuelle | Prédiction post-S1 | Si S1 échoue → |
|---|---|---|---|---|
| 1 | Engulfing_Bar_V056 | SAVE (contrôle) | E[R] ≈ +0.02 (baseline) | Regression alert W.5 |
| 2 | Aplus_03_IFVG_Flip_5m | IMPROVE | Marginal — bloqué par TP 2.0R inatteignable (peak_R p80=1.06R). S1 peut ajouter +0.02-0.03R mais pas suffisant. | **IMPROVE** : recalibrer TP1 2.0→0.70R avant de juger |
| 3 | Morning_Trap_Reversal | IMPROVE (reversal, entry_confirm only) | Très marginal — déjà asymptoté à -0.081 post-C.1. S1 est un 2e filtre sur le même signal. | **REWRITE ou KILL** : 3 leviers déjà tentés |
| 4 | Liquidity_Sweep_Scalp | IMPROVE (reversal) | Très marginal — vwap_regime effet nul, cap session=3 binding. | **KILL** probable : signal asymptote confirmé |
| 5 | FVG_Fill_V065 | REWRITE | Inconnu — execution_layer était artefact audit. Premier test clean. | **REWRITE** : TP fixed 2R + pas de liquidity-draw |
| 6 | Liquidity_Raid_V056 | REWRITE (reversal) | Inconnu — premier test clean | **REWRITE** : pas de bias gate, pas de liquidity target |
| 7 | Range_FVG_V054 | REWRITE (range, entry_confirm only) | Bas — range playbook, pas de edge directionnel | **REWRITE ou KILL** |
| 8 | Asia_Sweep_V051 | REWRITE (reversal) | **S1 ne fix pas le SESSION_WINDOW_MISMATCH** (B0.2 verdict). 0 trades probables. | **RECREATE from scratch** si session window déclarée correctement dans MASTER Aplus_02 |
| 9 | London_Fakeout_V066 | REWRITE (reversal) | n=1 historique → statistiquement indéterminé | **REWRITE** ou **besoin plus de data** |
| 10 | OB_Retest_V004 | REWRITE (continuation) | Inconnu — OB detector fix v2 appliqué. Premier vrai test. | Dépend du résultat |

**Confiance en prédictions** : moyenne. Phase M est précisément là pour **falsifier ou confirmer** ces prédictions avec data, pas pour les valider par supposition.

## Familles MASTER non-instanciées (RECREATE candidates explicites)

| Family MASTER | Statut actuel | Bloquant pour implémentation |
|---|---|---|
| **Family A** — Sweep + IFVG 5m + Breaker/OB/FVG entry 1m | `research_only` dans `playbooks_Aplus_from_transcripts.yaml`. Aplus_03 v1 = premier test Family A, IFVG-only. Aplus_01 (sweep+IFVG+breaker) jamais instancié. | `tp_logic: liquidity_draw` absent du schema, pipeline 5m→1m réel absent |
| **Family B** — D/4H bias + 15m BOS + FVG/OB entry | Seul HTF_Bias_15m_BOS existe (legacy, survivor n=3, +0.23 — trop petit échantillon). Aplus_04 (version MASTER fidèle) jamais instancié. | `setup_tf: 15m` utilisé nulle part ailleurs, pipeline HTF→15m pas testé end-to-end |
| **Family F** — Premarket sweep + 5m confirm + 5m continuation | Aplus_02 jamais instancié. | Premarket session context absent du pipeline (sessions chargées : NY, LONDON, ASIA) |

**Conséquence pratique** : avant d'instancier Aplus_01/02/04 (Phase C-new du plan), il faut étendre le schema YAML. C'est pourquoi C-new **suit** Phase M dans l'ordre — M donne le signal sur les vocab-borrowing, C-new crée les vrais Aplus.

## Concepts MASTER mal compris / mal transcrits (flag explicite)

1. **TP = liquidity draw** — concept : TP au prochain session high/low (pas RR fixe). YAML actuel ne l'exprime pas. Tous les TPs fixed RR. Phase B1 a mesuré peak_R p60 = 0.4–0.6R → fixed 2.0R inatteignable pour la plupart. **Correctif : étendre schema `tp_logic: liquidity_draw` + lookup des liquidity pools.**

2. **1m execution sur 5m setup** — concept MASTER : détecter setup 5m → patienter → confirmer 1m dans la zone → exécuter. Implémentation actuelle : gate 1m hardcodée sur la bougie courante (`check_entry_confirmation` lit `candles_1m[-1].close`). **Ce n'est pas le pipeline MASTER, c'est un gate simpliste.** Correctif : pipeline réel avec queue de setups en attente + confirmation 1m dans la zone d'entrée.

3. **HTF bias enforcement** — concept : aligner la direction du trade avec D/4H bias. YAML field `require_htf_alignment: D` existe (ajouté Phase S1 via Engulfing) et fonctionne. **Mais** 0/7 "MASTER faithful" ne l'utilise. Phase M étendra à 5-6 d'entre eux.

4. **Post-Judas / Silver Bullet / MMXM / OTE / SMT divergence** — concepts présents dans MASTER_FINAL.txt (71 transcripts), **zéro modélisation dans le moteur**. Hors scope M/R/C-new. À documenter pour famille G+ future.

## Règle absolue pour la lecture des résultats Phase M

- **Un playbook qui échoue S1 n'est pas "sauvé par recette supplémentaire"**. Au moins 3 leviers indépendants tentés = candidat KILL.
- **Un playbook qui dépasse E[R]>0 avec S1 est candidat SAVE** (à préciser avec n ≥ 30 et stabilité cross-weeks).
- **Un playbook avec signal non-null (peak_R p60 ≥ 0.5R ou WR ≥ 40%) mais E[R]<0 est candidat IMPROVE** (TP/SL calibration).
- **Un playbook qui ne passe aucune des 3 catégories ci-dessus** est candidat REWRITE (si concept MASTER justifié) ou RECREATE (si Family MASTER entière manquante) ou KILL.

## Lien avec la suite du plan

Séquence post-M du plan : R (rewrites ciblés BOS_Scalp_5m1m + FVG_Scalp_5m1m) → C-new (Aplus_01/02/04 from MASTER) → K (kills finaux) → P (portfolio). Phase M nourrit R et K avec des verdicts concrets ; C-new est indépendant (Families A/B/F jamais testées, donc bear case Family C/E ne s'applique pas à elles).
