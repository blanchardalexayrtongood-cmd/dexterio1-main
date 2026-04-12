# Campagne WF — core-3 playbooks, OOS agrégé ~3 mois (2025)

**Identifiant** : `wf_core3_oos_jun_nov2025`  
**Git SHA des runs** : `b8e7bcf9035914668eff7ff78329eb7343c07820` (voir `mini_lab_summary_*.json`)

---

## PREUVE CODE

- **Portefeuille IN (YAML campagne, hors canon)** : `backend/knowledge/campaigns/campaign_wf_core3_jun_nov2025.yml`  
  - `NY_Open_Reversal`, `FVG_Fill_Scalp`, `Session_Open_Scalp` uniquement (copie des blocs depuis `knowledge/playbooks.yml`).
- **Moteur / paper / NY code / NF canon / Wave2 / front / Rust** : **non modifiés** pour cette passe.
- **Risk** : `respect_allowlists=true`, `bypass_lss_quarantine=false` (LSS hors YAML + pas de bypass quarantaine).
- **Comportement moteur existant** : le backtest enregistre encore **5** playbooks « CORE+A+ » (`DAY_Aplus_*`, `SCALP_Aplus_*` chargés ailleurs) ; le **funnel** montre **0 trade** sur News_Fade / LSS (absents du YAML). Les A+ listés sont en **DENYLIST** → pas de trades attendus via risk pour eux. **Limite de preuve** : ce n’est pas un isolat « 3 playbooks enregistrés » au sens strict du compteur `playbooks_registered_count`.

---

## PREUVE RUN

### 1) Préflight données (fenêtre alignée aux parquet 1m)

```bash
cd backend
.venv/bin/python scripts/backtest_data_preflight.py \
  --start 2025-06-03 --end 2025-11-27 \
  --symbols SPY,QQQ --warmup-days 30 --ignore-warmup-check \
  --json > results/campaigns/wf_core3_oos_jun_nov2025/preflight_2025-06-03_2025-11-27.json
```

- **Résultat** : `ok=true` ; **warnings** warmup HTF (première barre 2025-06-02 08:00 UTC > `warmup_start` théorique). Pas d’`errors`.

### 2) Walk-forward mini-lab (2 fenêtres **test** = OOS, `--fail-fast`)

**Important** : ne **pas** passer un `--` seul avant les flags de `run_mini_lab_week` (sinon argparse rejette `--`). Les arguments après `run_walk_forward_mini_lab.py` sont déjà relayés via `parse_known_args`.

```bash
cd backend
.venv/bin/python scripts/run_walk_forward_mini_lab.py \
  --start 2025-06-03 --end 2025-11-27 \
  --output-parent wf_core3_oos_jun_nov2025 \
  --fail-fast \
  --symbols SPY,QQQ \
  --playbooks-yaml knowledge/campaigns/campaign_wf_core3_jun_nov2025.yml \
  --no-bypass-lss-quarantine \
  --strict-manifest-coverage
```

- **Plan calendaire** : `WalkForwardLightV0` sur **[2025-06-03, 2025-11-27]** (178 jours).
- **OOS (test uniquement)** :
  - Split 0 test : **2025-08-30 → 2025-10-12** (44 j calendaires)
  - Split 1 test : **2025-10-13 → 2025-11-27** (46 j calendaires)  
  → **90 jours calendaires OOS cumulés** sur 2 volets (≈ 3 mois agrégés, pas une seule fenêtre continue).

### 3) Gate, audit, rollup

```bash
cd backend
OUT=results/campaigns/wf_core3_oos_jun_nov2025
.venv/bin/python scripts/campaign_gate_verdict.py \
  results/labs/mini_week/wf_core3_oos_jun_nov2025/wf_s0_test/mini_lab_summary_wf_s0_test.json \
  --manifest results/labs/mini_week/wf_core3_oos_jun_nov2025/wf_s0_test/run_manifest.json \
  --require-manifest-coverage --require-trade-metrics \
  --out "$OUT/gate_verdict_wf_s0_test.json"

.venv/bin/python scripts/campaign_gate_verdict.py \
  results/labs/mini_week/wf_core3_oos_jun_nov2025/wf_s1_test/mini_lab_summary_wf_s1_test.json \
  --manifest results/labs/mini_week/wf_core3_oos_jun_nov2025/wf_s1_test/run_manifest.json \
  --require-manifest-coverage --require-trade-metrics \
  --out "$OUT/gate_verdict_wf_s1_test.json"

.venv/bin/python scripts/audit_campaign_output_parent.py \
  --output-parent wf_core3_oos_jun_nov2025 --out "$OUT/audit_output_parent.json"

.venv/bin/python scripts/rollup_campaign_summaries.py \
  --output-parent wf_core3_oos_jun_nov2025 --out "$OUT/rollup_campaign_summaries.json"
```

- **Exit codes** : `0` pour l’ensemble (WF `max_returncode=0`, audit `overall_ok=true`).

---

## PREUVE TEST

- Aucun nouveau test unitaire ajouté (hors périmètre demandé). La non-régression outils reste couverte par `scripts/backtest_campaign_smoke.py` / CI existante.

---

## PREUVE ARTEFACT

| Artefact | Chemin |
|----------|--------|
| Préflight JSON | `backend/results/campaigns/wf_core3_oos_jun_nov2025/preflight_2025-06-03_2025-11-27.json` |
| Plan + méta WF | `backend/results/labs/mini_week/wf_core3_oos_jun_nov2025/walk_forward_campaign.json` |
| Summary / manifest / parquet trades (×2) | `backend/results/labs/mini_week/wf_core3_oos_jun_nov2025/wf_s0_test/`, `.../wf_s1_test/` |
| Gate (×2) | `gate_verdict_wf_s0_test.json`, `gate_verdict_wf_s1_test.json` |
| Audit parent | `audit_output_parent.json` |
| Rollup | `rollup_campaign_summaries.json` |

---

## ANALYSE

- **Traçabilité** : `data_coverage_ok=true` partout ; `trade_metrics_parquet` présent (schéma `MiniLabTradeMetricsParquetV0`) sur les deux plis.
- **Portefeuille demandé** : seuls NY / FVG / Session ont des **trades** ; NF et LSS à 0 sur le funnel (conforme au YAML + risk).
- **PnL agrégé (rollup, pondéré trades)** : `expectancy_r_weighted_by_trades ≈ -0.0267` ; `sum_pnl_dollars_tracked ≈ -76 583 $` sur **2780** trades (les deux OOS). **Aucun signal de robustesse positive** sur cette plage avec ce sous-ensemble.
- **Verdict outil `campaign_gate_verdict`** (les deux plis) : **`BACKTEST_READY_BUT_NOT_PAPER_READY`** — motif : `playbooks_yaml` non null (campagne dérivée, hors noyau canon paper standard).
- **Capital final dans les summaries** : valeurs très différentes entre plis (`final_capital` ~1.2k vs ~22k) — **ne pas utiliser comme preuve financière** sans audit ; les métriques parquet / rollup sont la base quantitative retenue ici.
- **Données** : préflight `ok` mais **warmup HTF incomplet** vs fenêtre théorique → pour monter à **6 mois / 1 an** avec exigence stricte, traiter soit l’historique avant juin 2025, soit une politique manifest explicite (déjà documentée côté ladder).

---

## DÉCISION

1. **Verdict outil (officiel dans le repo)** : **`BACKTEST_READY_BUT_NOT_PAPER_READY`** sur chaque pli OOS (YAML dérivé + checks data/metrics OK).

2. **Verdict campagne (taxonomie demandée)** : **`BACKTEST_OK_3M_OOS_LIMITED_SCOPE`**  
   - **OK** : chaîne complète exécutée sur **vraies données** SPY/QQQ 1m, **2 plis OOS** totalisant **~90 jours calendaires**, artefacts auditables (manifest, summary, parquet, WF JSON, rollup).  
   - **LIMITED_SCOPE** : playbook set restreint via YAML campagne + exclusion NF / Trend_FVG / LSS / MTR par conception ; pas équivalent à un run « AGGRESSIVE canon full YAML ».

3. **Pas** : `NOT_READY` (checks stricts passent), `BACKTEST_NEEDS_DATA_FIX` (bloquant non déclenché — mais **avertissement** warmup), `BACKTEST_NEEDS_PLAYBOOK_SCOPE_REDUCTION` (le scope est déjà réduit ; le problème observé est **performance négative**, pas l’absence de scope).

---

## NEXT STEP

1. **Données** : étendre l’historique **avant** 2025-06-02 ou accepter explicitement `--manifest-ignore-warmup-check` + le documenter pour toute exigence « 6m / 1a » stricte.
2. **Comparabilité** : rejouer une variante **sans** `playbooks_yaml` (canon seul) **uniquement** si la policy permet d’exclure NF/LSS/MTR autrement — sinon garder YAML campagne et assumer `BACKTEST_READY_BUT_NOT_PAPER_READY` jusqu’à alignement produit.
3. **Edge** : investiguer **pourquoi** E[R] ≈ -0.027 sur 2780 trades (coûts, sur-fréquence, régimes) — hors scope « preuve de chaîne » mais **bloque** toute montée de ladder **produit**.
4. **Doc runner** : corriger l’exemple `--` dans le docstring de `run_walk_forward_mini_lab.py` lors d’une future passe doc (mini-diff, pas bloquant pour la validité des commandes ci-dessus).
