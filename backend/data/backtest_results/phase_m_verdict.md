# Phase M — Mass-apply S1 recipe — verdict 2026-04-21

## Rappel protocole

- **Overlay** : [mass_s1_v1.yml](backend/knowledge/campaigns/mass_s1_v1.yml) = copie intégrale de `playbooks.yml` avec patch S1 sur **10 candidats** :
  - **4 continuations** (`entry_confirm: require_close_above_trigger=true` + `entry_buffer_bps=2.0` + `require_htf_alignment: D`) : Engulfing_Bar_V056, Aplus_03_IFVG_Flip_5m, FVG_Fill_V065, OB_Retest_V004
  - **6 reversal/range** (`entry_confirm` seulement, pas de HTF gate par design) : Morning_Trap_Reversal, Liquidity_Sweep_Scalp, Liquidity_Raid_V056, Range_FVG_V054, Asia_Sweep_V051, London_Fakeout_V066
- **Conditions** (calib_corpus_v1-like) : 4 semaines (jun_w3 + aug_w3 + oct_w2 + nov_w4), SPY+QQQ, `RISK_EVAL_ALLOW_ALL_PLAYBOOKS=true`, `RISK_EVAL_RELAX_CAPS=false` (caps actives), `RISK_EVAL_DISABLE_KILL_SWITCH=true`, allowlist restreinte aux 10 candidats.
- **Budget slippage** : -0.065 R/trade (reconcile harness [reconcile_paper_vs_backtest_calib_oct_w2.md](backend/data/backtest_results/reconcile_paper_vs_backtest_calib_oct_w2.md)).
- **Total trades** : **125** sur 4 semaines.

## Résultats bruts

### Trade count par playbook × semaine

| Playbook | jun_w3 | aug_w3 | oct_w2 | nov_w4 | Σ | matches 4w |
|---|---:|---:|---:|---:|---:|---:|
| Engulfing_Bar_V056 | 7 | 7 | 8 | 8 | **30** | 1198 |
| Morning_Trap_Reversal | 8 | 10 | 8 | 8 | **34** | 1826 |
| Liquidity_Sweep_Scalp | 10 | 15 | 8 | 13 | **46** | 1543 |
| Aplus_03_IFVG_Flip_5m | 2 | 2 | 2 | 7 | **13** | 173 |
| OB_Retest_V004 | 0 | 0 | 2 | 0 | **2** | 130 |
| **FVG_Fill_V065** | 0 | 0 | 0 | 0 | **0** | 74 (setups passent risk, gate reject) |
| **Range_FVG_V054** | 0 | 0 | 0 | 0 | **0** | 78 |
| **Liquidity_Raid_V056** | 0 | 0 | 0 | 0 | **0** | 2 |
| **Asia_Sweep_V051** | 0 | 0 | 0 | 0 | **0** | 0 |
| **London_Fakeout_V066** | 0 | 0 | 0 | 0 | **0** | 1 |

### Métrique E[R] — playbooks ayant produit des trades

| Playbook | n | WR | E[R] gross | **E[R] net** | Σ R | peak_R p60 | peak_R p80 | mean mae_R |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| OB_Retest_V004 | 2 | 50% | +0.102 | **+0.037** | +0.20 | 1.55 | 1.82 | -0.67 |
| Engulfing_Bar_V056 | 30 | 66.7% | +0.043 | **-0.022** | +1.28 | 0.95 | 1.53 | -0.42 |
| Morning_Trap_Reversal | 34 | 29.4% | +0.003 | **-0.062** | +0.10 | 1.46 | 3.02 | -0.84 |
| Liquidity_Sweep_Scalp | 46 | 32.6% | -0.041 | **-0.106** | -1.88 | 0.36 | 0.88 | -0.29 |
| Aplus_03_IFVG_Flip_5m | 13 | 38.5% | -0.047 | **-0.112** | -0.62 | 0.55 | 1.19 | -0.42 |

### E[R] par semaine (stabilité)

| Playbook | jun | aug | oct | nov |
|---|---:|---:|---:|---:|
| Engulfing_Bar_V056 | -0.139 | +0.230 | -0.068 | +0.149 |
| Morning_Trap_Reversal | -0.029 | -0.006 | -0.247 | +0.296 |
| Liquidity_Sweep_Scalp | +0.034 | -0.030 | -0.086 | -0.082 |
| Aplus_03_IFVG_Flip_5m | -0.081 | +0.054 | -0.101 | -0.052 |
| OB_Retest_V004 | — | — | +0.102 | — |

## Verdict 5-classes par candidat M

### SAVE (0)
Aucun playbook ne passe net E[R] > 0 avec n ≥ 30 sur 4 semaines.
Engulfing_Bar_V056 était la référence S1 pré-M (+0.020 net sur n=38 en calib_corpus_v1). Sous M (caps actives, fenêtre 4w neuve) : **E[R] gross +0.043 / net -0.022**. Proche breakeven mais **ne passe plus net zero**. Le baseline s'est légèrement dégradé quand la recette s'applique sur un corpus non train-on.
→ reclassé **IMPROVE** (garder + tenter TP calib + re-mesurer).

### IMPROVE (3)

| Playbook | Signal confirmé | Problème résiduel | Correctif proposé |
|---|---|---|---|
| **Engulfing_Bar_V056** | WR 66.7% n=30, peak_R p80=1.53R | TP1 placement — 4/30 TP1 hits (13%), 14/30 time_stops | Tester TP1 1.5→1.0R (cible le p50 du peak_R) + TP2 à p80. Si net >0 stable sur 4w → SAVE. |
| **Morning_Trap_Reversal** | peak_R p80=3.02R (MFE énorme), WR 29.4% | TP1 2.0R + fixed SL inatteignables vs mae_r mean -0.84R | 4e levier : TP peak-R calibrated (p60=1.46R) + SL tighter (p70 mae). Si ne passe pas → **KILL** (4 leviers tentés). |
| **Aplus_03_IFVG_Flip_5m** | peak_R p80=1.19R, WR 38.5% | TP fixed 2.0R inatteignable (0/13 TP hits sous S1). Même pathologie qu'en v1. | Apply TP1 2.0→0.70R, BE 1.5→0.40R (calib explicite déjà proposée [aplus03_v1_verdict.md](backend/data/backtest_results/aplus03_v1_verdict.md)). Si pass → premier Family A viable. Si fail → bear case Family A renforcé. |

### REWRITE partial (2) — matches OK mais S1 gate incompatible

| Playbook | Gap | Direction |
|---|---|---|
| **FVG_Fill_V065** | 74 matches/4w, setups passent risk filter, **entry_confirm gate rejette 100%** (0 commits sur 2 setups reach gate). Pas de vrai signal structurel pour S1. TP fixed 2R + pas de liquidity target. | **REWRITE** : bias gate ON + reprendre sans `require_close_above_trigger` (designed pour engulfing, pas FVG fill) + TP peak-R. Nouveau YAML, pas patch. |
| **Range_FVG_V054** | 78 matches/4w, 0 trades. Gate entry_confirm n'a même pas de stats = 0 setups propagés côté execution. Range playbook = pas de signal directionnel clean. | **REWRITE** ou **KILL** — range playbooks n'ont pas d'edge en intraday sans regime confirmation. |

### RECREATE from scratch (2) — detecteur silencieux OU family MASTER mal instanciée

| Playbook | Preuve silence | Decision |
|---|---|---|
| **Asia_Sweep_V051** | 0 matches sur 4 semaines. **SESSION_WINDOW_MISMATCH** confirmé (B0.2) — Asia session ≠ sessions NY chargées dans le corpus US. | **RECREATE** from MASTER Family F (`Aplus_02_Premarket_Sweep_5m`). Schema YAML doit supporter premarket session context (absent aujourd'hui). Cette implémentation actuelle emprunte du vocab sans mécanique. |
| **London_Fakeout_V066** | 1 match / 4 semaines. Détecteur structurellement rare. | **RECREATE** si Family MASTER correspondante identifiée, sinon **KILL**. |

### KILL (3 candidats M + confirmés)

| Playbook | Leviers tentés | Preuve |
|---|---|---|
| **Liquidity_Sweep_Scalp** | C.1 vwap_regime (null effect), C.3 entry_confirm (M, E[R] -0.041), TP recalibration suggérée B1 (peak_R p60=0.36R → TP fixed 2R structurellement unreachable) | **peak_R p80=0.88R = signal non-exploitable en R/R fixe**. 4 leviers indépendants, asymptote confirmée. **KILL définitif**. |
| **Liquidity_Raid_V056** | 2 matches sur 4 semaines | Détecteur cassé ou signal trop rare. **KILL** sauf si test rapide (drop S1 gate) démontre ≥20 matches/semaine. |
| **OB_Retest_V004** | n=2 insuffisant pour verdict | **DEFER** : ré-évaluer sans `require_close_above_trigger` pour avoir n exploitable. Pas KILL, pas SAVE. |

## Familles MASTER : statut réel post-M

| Family | Playbook canonique | Testé sous M ? | Verdict |
|---|---|---|---|
| **Family A** (Sweep + IFVG + Breaker 1m) | Aplus_03 (IFVG only), Aplus_01 (full) | Aplus_03 = IMPROVE candidat ; Aplus_01 jamais instancié | **Partiellement testée**. Aplus_03 signal non-null (peak_R p60=0.55R, WR 38.5%), TP calibration requise. Aplus_01 (full Family A avec Breaker) reste à créer. |
| **Family B** (D/4H bias + 15m BOS) | Aplus_04 | Non | **Jamais testé**. Schema gap : `setup_tf: 15m` utilisé nulle part hors HTF_Bias_15m_BOS legacy. |
| **Family F** (Premarket Sweep + 5m continuation) | Aplus_02 | Non | **Jamais testé**. Schema gap : premarket session context absent du pipeline. |
| **Family C** (FVG Fill) | V065 | Oui (M = REWRITE partial) | Borrowed-vocab confirmé — gate S1 incompatible + TP fixed inatteignable. |
| **Family E** (Liquidity Raid) | V056 | Oui (M = KILL candidat) | Détecteur rare + signal asymptote. |

## Concepts MASTER encore mal compris / mal transcrits

1. **TP = liquidity draw** (non résolu) — Schema YAML toujours fixed RR. Phase M confirme : 4/5 playbooks avec trades ont peak_R p60 ∈ [0.36, 1.55]R, tous les TP sont à 2.0R fixed. **Aucun TP atteint par construction** pour Liquidity_Sweep/Aplus_03 (0/13 et 0/46 TP1 hits). **Blocker structurel** — tant que ce schema n'est pas étendu (`tp_logic: liquidity_draw` + lookup pools), les Families A/B/F qu'on voudrait recréer auront la même pathologie.

2. **1m execution sur 5m setup** — Phase M confirme : la gate `require_close_above_trigger` lit `candles_1m[-1].close` comme un simple filtre post-setup, pas une queue de setups en attente de confirmation 1m dans la zone. Résultat visible : Engulfing 181 setups checked / 32 passed (82% rejected) — le gate comprime arbitrairement le sample sans pipeline réel. **Pour Family A, B, F, cela doit être refait proprement** (state machine : setup 5m pending → wait → 1m tick in zone → execute).

3. **HTF bias enforcement** — `require_htf_alignment: D` fonctionne techniquement (Phase S1 baseline). Sous M : appliqué à 4 continuations. Aucune évidence additionnelle de valeur sur ce run (sample écrasé par les 6 reversals). À re-mesurer proprement.

4. **Family F (Premarket) + Family A (Breaker)** : 0 instantiation à ce jour. Phase C-new doit les créer from scratch.

## Décompte final 10 candidats M

| Classe | Count | Playbooks |
|---|---:|---|
| SAVE | **0** | — |
| IMPROVE | **3** | Engulfing_Bar_V056, Morning_Trap_Reversal, Aplus_03_IFVG_Flip_5m |
| REWRITE partial | **2** | FVG_Fill_V065, Range_FVG_V054 |
| RECREATE from scratch | **2** | Asia_Sweep_V051, London_Fakeout_V066 |
| KILL | **2** + 1 DEFER | Liquidity_Sweep_Scalp, Liquidity_Raid_V056 (DEFER: OB_Retest_V004) |

## Gate M

Plan Gate M : *« ≥1 playbook additionnel franchit net E[R] > 0 OU verdict clair que la recette S1 ne généralise pas »*.

**Verdict** : **S1 ne généralise pas au-delà de son playbook natif (Engulfing)**.
- 0/10 net E[R] > 0 avec n exploitable.
- Engulfing lui-même est passé de +0.020 net (calib train) à -0.022 net (4w M).
- 5/10 candidats silencieux (matches→0 trades), dont 4 MASTER faithful.
- 3 playbooks (Morning_Trap, Liquidity_Sweep, Aplus_03) ont peak_R structure mais sont incompatibles avec TP fixed 2R.

**Réponse explicite à la question de fond** (posée par user, documentée dans [phase_m_pre_diagnostic.md](phase_m_pre_diagnostic.md)) :

> *« Est-ce que certaines stratégies actuelles ne devraient pas être améliorées, mais carrément reconstruites proprement à partir du MASTER / vidéos / logique de marché ? »*

**OUI, confirmé par données** :
- **4 playbooks doivent être réécrits** (FVG_Fill_V065, Range_FVG_V054, Asia_Sweep_V051, London_Fakeout_V066) : vocab MASTER emprunté sans mécanique, détecteurs silencieux ou incompatibles avec gate 1m.
- **2-3 playbooks doivent être créés from scratch** (Aplus_01 Family A full, Aplus_02 Family F, Aplus_04 Family B) : jamais instanciés, schema YAML actuel insuffisant.
- **2 playbooks doivent être tués** (Liquidity_Sweep_Scalp, Liquidity_Raid_V056) : signal asymptote ou détecteur cassé après 3-4 leviers.

**L'architecture YAML actuelle n'est PAS suffisante** pour exprimer un vrai setup MASTER. Extensions bloquantes requises :
1. `tp_logic: liquidity_draw` + pipeline lookup pools session high/low.
2. `setup_tf: 15m` + feed D/4H bias.
3. Premarket session context.
4. State machine 1m-confirm-in-zone (pas gate-on-latest-candle).

## Prochaine étape (per plan)

`M → R → C-new → K → P`

- **R.1 / R.2** : rewrites ciblés BOS_Scalp_5m1m + FVG_Scalp_5m1m (hors-M candidats).
- **R.3** : calib Aplus_03 (TP 2.0→0.70R) — 1 run 4w, verdict final IFVG Family A.
- **C-new** : étendre schema YAML (point 1-4 ci-dessus) avant d'instancier Aplus_01/02/04.
- **K** : confirmer DENYLIST Liquidity_Sweep_Scalp + Liquidity_Raid_V056.
- Decision user requise avant R pour : ordre Aplus_03 calib vs C-new schema extension vs kills.
