# Verdict — Stat_Arb_SPY_QQQ_v2 smoke nov_w4 (Leg 4.1 §0.5)

**Date** : 2026-04-22
**Décision** : **SMOKE_FAIL → ARCHIVED** (3/3 kill rules pièce H atteintes) → **Leg 4.2 progression automatique**

---

## Bloc 1 — identité du run

| Champ | Valeur |
|---|---|
| Playbook | Stat_Arb_SPY_QQQ_v2 (Leg 4.1 §0.5 non-MASTER quant hypothesis) |
| Version | v2 (daily cointégration 60j rolling gate + intraday 5m z-score) |
| Période | nov_w4 = 2025-11-17 → 2025-11-21 (5 sessions NY) |
| Instruments | SPY (jambe y) + QQQ (jambe x), beta-neutral |
| Mode | Harness autonome [stat_arb_smoke_v2.py](backend/scripts/stat_arb_smoke_v2.py), IdealFillModel équivalent (no costs) |
| Corpus | 35526 SPY 1m + 36101 QQQ 1m bars, 6287 5m bars joint, 395 5m session bars window smoke |
| Daily corpus | 89 daily closes pour fenêtre rolling 60 trading days (load 120j calendar) |
| Config | beta_window=60, z_window=60, daily_coint_window=60, entry_z=2.0, exit_z=0.5, blowout_z=3.0, time_stop=18 bars 5m (1.5h), lockout=6 bars |
| Dossier | [backend/knowledge/playbooks/stat_arb_spy_qqq_v2/dossier.md](backend/knowledge/playbooks/stat_arb_spy_qqq_v2/dossier.md) |

---

## Bloc 2 — métriques

| Métrique | Valeur |
|---|---:|
| **daily_coint_pass_days** | **5/5 trading days ON** |
| bars_skipped_daily_gate | **0** (gate jamais binding sur nov_w4) |
| Setups emitted | 8 |
| Trades closed | 8 |
| **n** | **8** |
| **WR** | **37.5 %** (3/8) |
| **E[R] gross** | **−0.179** |
| **PF** | **0.258** |
| **peak_R p80** | **0.147** |
| mae_R p20 | −0.394 |
| total_R | −1.430 |
| **mean_reversion_rate** (TP_MEAN_REVERSION / n) | **25.0 %** (2/8) |
| Exit reasons | SL_BLOWOUT = 3 · TIME_STOP = 3 · TP_MEAN_REVERSION = 2 |

**Résultat remarquable** : métriques **identiques à Sprint 3 v1** (n=8, WR=37.5 %, E[R]=−0.179, PF=0.258, peak_R p80=0.15, mean_rev=25 %, exits 3+3+2). Cette identité n'est pas un bug — elle signe un diagnostic structurel décisif (voir Bloc 3).

---

## Bloc 3 — lecture structurelle

### Le daily coint gate fonctionne-t-il ?
**Oui, parfaitement.** 5/5 jours nov_w4 avec `is_coint = True` (ADF stat ≈ −3.4 à −3.55, largement sous critical value 5 %). Le gate régime est **informatif et non-restrictif** : SPY-QQQ sont structurellement coïntégrés en TF daily sur fenêtre 60j, comme prévu par la thèse. Kill Rule 1 PASS (≥1 jour ON requis).

### Le gate change-t-il le comportement vs v1 ?
**Non, dans ce corpus.** `bars_skipped_daily_gate = 0` → le gate autorise toutes les bars 5m de la smoke week. Conséquence : v2 dégénère fonctionnellement en v1 avec `require_cointegration=False` (qui était la config v1 par défaut puisque v1 intraday coint était trop strict, 1/59 PASS). Les 8 setups, 8 trades, exits, PnL sont **byte-identiques** à v1.

### Pourquoi c'est un diagnostic décisif
Le test v1 avait laissé une ambiguïté : "si le gate coint était daily et permissif, le signal intraday aurait-il survécu ?". v2 **tranche cette question empiriquement** : gate daily 5/5 PASS, gate intraday contourné → même n, même E[R], même mean-rev rate. **Le gate n'était pas le bottleneck. Le signal lui-même (|z|>2σ sur spread SPY-QQQ 5m log-return) ne mean-reverse pas suffisamment souvent (25 % < 50 %) et ne produit pas d'amplitude exploitable (peak_R p80 = 0.15 R).**

### Le problème vient-il du signal, de la sortie, de la mécanique, ou du contexte ?
**Du signal.** Diagnostic cross-v1/v2 :
- **Contexte** : daily coint PASS 100 % → régime nominal idéal pour l'hypothèse mean-rev. Pas de regime change, pas de blow-out structurel.
- **Mécanique** : tracker fonctionnel (8 setups émis correctement), fills next-bar-open, lockout post-SL appliqué.
- **Sortie** : 3 TIME_STOP + 3 SL_BLOWOUT + 2 TP. Sur les 2 TP, gains modestes — sur les 2 SL_BLOWOUT, pertes catastrophiques (identiques v1 : un SHORT_SPREAD à −0.94R et un LONG_SPREAD à −0.47R) parce que le chemin prix entre armement et exit visé |z|≤0.5 passe par |z|>3.0 adverses plus souvent qu'il ne tend vers 0.
- **Signal** : l'hypothèse "déviation z>2σ sur spread SPY-QQQ 5m log-return = mean-reverting" est **fausse** dans ce corpus. Mean_rev_rate 25 % signe que la déviation continue dans la même direction au moins 3x plus souvent qu'elle revient.

### Le signal vit-il ? Sous-exercice ?
Signal **vit** (8 armements / 395 session bars = ~2 % density, raisonnable). **Pas sous-exercé.** Kill rule 1 (n<10) atteinte marginalement (8 < 10) mais sous-exercice n'est pas l'explication — mean_rev < 50 % et E[R] < 0 sont dominants.

### Distribution vs baseline
v1 et v2 convergent byte-identiquement → pas de distribution à comparer. L'invariance elle-même est l'information : le gate cointégration (quelle que soit sa forme — intraday 5m ou daily 60j) n'est **pas discriminant** pour la mean-reversion intraday SPY-QQQ. Le signal est autonomement mauvais.

---

## Bloc 4 — décision

### Kill Rules pièce H pré-écrites (AVANT smoke)

| Kill rule | Seuil | Observé | Statut |
|---|---|---|:---:|
| 1. `daily_coint_pass_days` < 1 sur 5 | < 1 | **5/5** | ✅ PASS (gate fonctionnel) |
| 2. `n` < 10 | < 10 | **8** | ❌ **ATTEINTE** |
| 3. `mean_reversion_rate` < 50 % | < 50 % | **25 %** | ❌ **ATTEINTE** |
| 4. `E[R] gross` ≤ 0 | ≤ 0 | **−0.179** | ❌ **ATTEINTE** |
| 5. Gate promotion | n≥10 ET mean_rev≥50% ET E[R]>0 | Non atteint | Non applicable |

**3/5 kill rules atteintes (rules 2, 3, 4). → SMOKE_FAIL → ARCHIVED terminal. → Leg 4.2 progression automatique.**

### Classification §20
**Cas C dominant** (edge absent) : signal exerçable (8 setups émis, gate régime permissif, mean-rev rate 25 % < 50 % = signal ne fait pas ce qu'on prédit). Même diagnostic que v1.

**Cas D secondaire possible** (codage faux au niveau hypothèse) : l'hypothèse v2 "daily coint gate unlocks intraday mean-rev" est réfutée par le fait que le gate daily PASS 100 % n'a rien changé. Mais ce n'est pas un **codage faux** — le gate est correctement implémenté et correctement évalué. C'est l'**hypothèse économique** qui est fausse : la stabilité structurelle daily n'implique **pas** la stationnarité 5m intraday du spread log-return.

### Décision : **ARCHIVED + Leg 4.2 automatique**

---

## Bloc 5 — why

### Pourquoi cette décision est rationnelle

**§H kill rules pré-écrites** : 3 seuils atteints sur 4 évaluables (rules 2, 3, 4). Appliquer la règle sans détour.

**Valeur du test v2** : l'identité v2=v1 fournit une réfutation **plus forte** que v1 seul :
- v1 laissait ouvert "et si le gate était permissif ?"
- v2 prouve empiriquement que le gate n'est pas le bottleneck
- Conclusion : le signal (spread z-score 5m sur SPY-QQQ) est autonomement faible dans ce régime.

### Pourquoi on n'itère pas plus

**§19.3 budget** (3 iter max post-smoke par hypothèse). **§10 règle 11** (réouverture branche morte interdite sauf hypothèse structurellement nouvelle). Itérations v2 possibles mais **non tentées** :

1. **Ajuster seuils z** (entry 1.5/2.5, exit 0.3/0.8, blowout 2.5/4.0) — pur tuning fit-history, violerait §19.3 & §10 r11 (pas structurellement différent).
2. **Changer time_stop / lockout** — idem tuning.
3. **Baisser daily_coint_alpha à 0.01** — déjà 5/5 PASS, ne change rien.
4. **Raise `daily_coint_window` à 90 ou 120 trading days** — le problème n'est pas côté gate.

Aucune de ces options ne réfute ni ne valide l'hypothèse structurelle "spread SPY-QQQ 5m mean-reverting sous gate daily". Le diagnostic est posé : **ce signal ne mean-reverse pas intraday**.

Une **v3 structurellement différente** resterait légale §10 r11 :
- Paire alternative (SPY-IWM, QQQ-DIA, XLK-QQQ) — hypothèse nouvelle.
- TF différent (spread daily avec hold 5-10 jours, pas intraday).
- Gate autre que EG (Johansen multi-var, Ornstein-Uhlenbeck MLE).
- Signal différent (Bollinger bands daily, rolling p95/p5 percentile break).

Mais ces pistes **sortent du scope Leg 4.1** (§0.5 "2-3j par node") et entrent dans Leg 5 (ESCALADE USER post-épuisement).

### Pourquoi on ne tue pas trop tôt

On **ne tue pas** l'hypothèse stat-arb universellement. On tue la **configuration SPY-QQQ 5m intraday** (v1 et v2 convergents). D'autres paires, TF, ou gates restent autorisées per §10 r11.

### Pourquoi on ne promeut pas trop tôt

**N/A** (n=8, E[R]=−0.179, mean_rev=25 %). Même un relâchement de kill rules ne ferait pas franchir le bar promotion (`E[R]>0.10R + n>30 + PF>1.2` — feedback user explicite).

### 9e data point négatif — méta-hypothèse

Cumul cross-playbook 2025 SPY/QQQ :
1. Aplus_03 v1 / 2. Aplus_03_v2 α'' / 3. Aplus_04 v1 / 4. Aplus_04_v2 α'' / 5. Aplus_04_v2 ε (Family A/B ICT)
6. Sprint 1 Aplus_01 Family A full / 7. Leg 3 Aplus_02 Family F Premarket (MASTER ICT bear ferme)
8. Sprint 3 Stat_Arb v1 / 9. Leg 2 cohort IFVG/VWAP/HTF 12w
10. **Leg 4.1 Stat_Arb v2** ← ce data point

**ICT bear ferme** (6/6 MASTER families testées) **+ stat-arb SPY-QQQ 5m bear ferme** (v1 & v2 convergents). Le méta-bear s'étend : la paire SPY-QQQ en TF 5m intraday ne présente pas d'edge exploitable **quel que soit le mode de gate régime**.

### Progression automatique

Per §0.5 Leg 4 arbre :
> "4.1 KILL → 4.2 VIX-regime overlay sur cohort survivor_v1 (fallback) — filtre régime §0.4-bis sur cohort existant News_Fade + Engulfing + Session_Open + Liquidity_Sweep."

**Leg 4.2 — VIX-regime overlay** (coût estimé ≤1j, pas de nouveau playbook) :
- **Input** : cohort survivor_v1 (4 playbooks) déjà auditée (survivor_v1_verdict.md : best 3-pack E[R]=−0.009).
- **Action** : charger série VIX quotidienne aligned au corpus, subset trades où VIX_close prior-day ∈ [15, 25] (mean-rev fertile §0.4-bis).
- **Kill rule** : subset cohort E[R] net ≤ 0.05R → Leg 5 ESCALADE USER.
- **PASS** : subset positif → corpus expansion / Polygon ingest pour Stage 1 gate.

Démarrage automatique Leg 4.2 sans attendre confirmation user (§0.9 CEO autonome, aucun point §0.3 déclenché).

---

**Synthèse** : Stat_Arb_SPY_QQQ_v2 (daily coint 60j gate + intraday 5m z-score) → 8 trades, E[R]=−0.179, mean_rev=25 %, byte-identical à v1. Gate daily 5/5 PASS n'est pas le bottleneck — **le signal 5m est autonomement faible**. Kill rules 2, 3, 4 atteintes. ARCHIVED. 9e data point négatif méta-hypothèse DexterioBOT 2025 SPY/QQQ intraday. Progression Leg 4.2 VIX-regime overlay cohort survivor_v1.
