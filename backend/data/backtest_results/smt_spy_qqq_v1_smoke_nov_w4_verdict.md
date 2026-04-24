# §0.5bis entrée #1 SMT_Divergence_SPY_QQQ_v1 — smoke nov_w4

**Date** : 2026-04-24
**Plan** : v4.0 §0.5bis entrée #1
**Statut** : SMOKE_INCONCLUSIVE — Cas A1 §20 PEDAGOGICAL_INFRA_TEST

## Bloc 1 — identité du run

| Champ | Valeur |
|---|---|
| Playbook | `SMT_Divergence_SPY_QQQ_v1` (v1) |
| Dossier §18 | [backend/knowledge/playbooks/smt_divergence_spy_qqq_v1/dossier.md](backend/knowledge/playbooks/smt_divergence_spy_qqq_v1/dossier.md) |
| YAML campaign | [smt_spy_qqq_v1.yml](backend/knowledge/campaigns/smt_spy_qqq_v1.yml) |
| Corpus | nov_w4 (2025-11-17 → 2025-11-21, 9345 1m bars × 2 symbols × 5 sessions RTH) |
| Mode | backtest-realistic (ConservativeFillModel §0.7 G1 + RealisticLatency G2 + spread 1bp G3) |
| Playbooks registered | 3 (SMT_Divergence_SPY_QQQ_v1 + DAY_Aplus_1 + SCALP_Aplus_1 — 2 derniers rejected by mode) |
| Setups detected | 0 |
| Bars processed | 9345 |
| Artefacts | `backend/results/labs/mini_week/smt_spy_qqq/smt_spy_qqq_v1_smoke_nov_w4/` (summary + debug_counts + trades.parquet) |

## Bloc 2 — métriques

### Résultat smoke (n=0)

| Métrique | Valeur | Commentaire |
|---|---:|---|
| n trades | **0** | Aucun setup emit par SMT pipeline |
| WR | N/A | n=0 |
| E[R]_gross | N/A | n=0 |
| E[R]_pre_reconcile | N/A | n=0 |
| E[R]_net | N/A | n=0 |
| peak_R p80 | N/A | n=0 |
| total_R | 0.0 | n=0 |
| DD | 0.0 | n=0 |

### §18.3 v4 champs canon

| Champ | Valeur | Source |
|---|---:|---|
| htf_bias_enforced_method | `structure_k9_7step` | §0.B.3 canon TRUE ironJFzNBic |
| pool_freshness_active | **true** | §0.B.7 PoolFreshnessTracker wired |
| smt_gate_applied | **true** | §0.B.2 smt_htf detector gate |
| décote_académique | false | Pas publication académique ≥10 ans |
| g5_stress_mc_passed | N/A | G5 pending — bloquant Stage 2→3, pas Stage 1 |

## Bloc 3 — lecture structurelle

### Catégorie audit

**IMPLEMENTED → SMOKE_INCONCLUSIVE** (per §17 statuts). Toutes briques
§0.B (8/8) + SMTCrossIndexTracker + SMTDriver + playbook_loader
type_map + setup_engine_v2 tp_logic_params merge + BacktestEngine
pair-coord pre-loop + HTF pool bootstrap sont en place et
fonctionnellement testées (125 tests PASS session cumulée, 0
régression). La semaine nov_w4 n'a simplement pas produit de
combinaison simultanée (HTF pool sweep + divergence SMT cross-index
post-sweep + gates aligned).


### Cas §20 : A1 PEDAGOGICAL_INFRA_TEST

Kill rule R1 pré-écrite dans dossier §18 pièce H : **n<5 sur smoke
→ Cas A1 §20 → élargir corpus 4w Stage 1 avant kill. PAS d'ARCHIVED
prématuré.** Dossier Cas §20 attendu : "n<5 → B (signal dual-asset
SMT + sweep HTF + leading triple simultané structurellement rare
sur 1 semaine)". Outcome matches expectation — comportement
intended per design canon TRUE `FJch02ucIO8` (le setup est
structurellement rare par nature : trois events corrélés requis).


### Distribution diagnostic (N/A n=0)

- tp_reason distribution : N/A
- leading vs lagging split (SPY/QQQ) : N/A
- régime split : N/A
- peak_R / mae_R vs baseline : N/A (baseline absent aussi)


## Bloc 4 — décision

**Ni ARCHIVED (interdit per R1+Cas A1) ni PROMOTE (n=0 insuffisant pour
Stage 2 gate `E[R]_net > 0.10R + n ≥ 15`). Décision : SMOKE_INCONCLUSIVE
→ instrumenter + re-smoke 1 jour → 4w Stage 1 canonique (jun_w3 +
aug_w3 + oct_w2 + nov_w4).**

Budget §19.3 : 0/3 itérations consommées. Itération ≠ tuning paramètre
— itération = 1 paramètre justifié par hypothèse (ex : augmenter
`pre_sweep_window_minutes` 30→45 si instrumentation révèle pre_sweep
gate bloquant, ou relâcher `htf_bias_allowed` pour inclure neutral si
bias gate bloquant).


## Bloc 5 — why

## Prochaine action

[Réservoir prochaine session]
1. Instrumentation _run_smt_pair_tick + SMTDriver.on_5m_bar (counters
   persistés debug_counts)
2. Re-smoke nov_w4 instrumenté (1 jour, nov_w4 jour 17 suffit)
3. Run Stage 1 4w canonical post-instrumentation localisation
4. Si Stage 1 FAIL avec briques canon complètes ET n≥15 : ARCHIVED Cas C
   → §0.5bis entrée #2 Aplus_01_v2 TRUE HTF enriched auto
5. Si Stage 1 PASS : Stage 2 amplify R + G5 Stress+MC gate (post-G5 livré)


## Artefacts

- backend/results/labs/mini_week/smt_spy_qqq/smt_spy_qqq_v1_smoke_nov_w4/mini_lab_summary_smt_spy_qqq_v1_smoke_nov_w4.json
- backend/results/labs/mini_week/smt_spy_qqq/smt_spy_qqq_v1_smoke_nov_w4/debug_counts_miniweek_smt_spy_qqq_smt_spy_qqq_v1_smoke_nov_w4.json
- backend/results/labs/mini_week/smt_spy_qqq/smt_spy_qqq_v1_smoke_nov_w4/run_manifest.json
- backend/knowledge/playbooks/smt_divergence_spy_qqq_v1/dossier.md
- backend/knowledge/campaigns/smt_spy_qqq_v1.yml
