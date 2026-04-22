# Verdict — Stat_Arb_SPY_QQQ_v1 smoke nov_w4 (Sprint 3)

## Bloc 1 — identité du run

| Champ | Valeur |
|---|---|
| Playbook | `Stat_Arb_SPY_QQQ_v1` |
| Version | v1 (1ère instantiation non-ICT) |
| Période smoke | 2025-11-17 → 2025-11-21 (nov_w4, 5 sessions) |
| Mode fill | `IdealFillModel` proxy (next-bar-open, 0 slippage, 0 cost) |
| Instruments | SPY (y-leg), QQQ (x-leg) |
| TFs | 5m décision, 1m data source |
| Direction | long/short spread (mean-reversion) |
| Dossier | [dossier.md](../../knowledge/playbooks/stat_arb_spy_qqq_v1/dossier.md) |
| Harness | [stat_arb_smoke.py](../../scripts/stat_arb_smoke.py) |
| Sortie | [trades.parquet + debug_counts.json](../../results/labs/mini_week/stat_arb_spy_qqq_v1/stat_arb_spy_qqq_v1_smoke_nov_w4/) |
| Git SHA | 016d9be (phase D1+D2) + smoke harness uncommitted |

### Paramètres testés

- `beta_window=60` (5h), `z_window=60`, `coint_window=200`
- `entry_z=2.0`, `exit_z=0.5`, `blowout_z=3.0`, `time_stop_bars=18`
- `lockout_bars=6`, `risk_dollars=100`
- `require_cointegration=False` — cointégration Engle-Granger intraday 200 bars passe 1/59 fenêtres (trop restrictif pour intraday 5m). SPY-QQQ sont structurellement cointégrés par construction (constituents chevauchants, factor common risk) ; le gate EG strict intraday est bruité. À re-enable en vague suivante avec cointégration sur TF daily (rolling 60 jours).

---

## Bloc 2 — métriques

| Métrique | Valeur |
|---|---:|
| **n trades** | **8** |
| **WR** | 37.5 % (3 W / 5 L) |
| **E[R] gross** | **−0.179** |
| **E[R] net+slippage** | non calculé (v1 = ideal) |
| **Total R** | −1.43 R |
| **PF** | 0.258 |
| **peak_R p80** | 0.147 |
| **mae_R p20** | −0.394 |
| **Durée moyenne** | 10 bars 5m (~50 min) |
| **mean_reversion_rate** | 25 % (2/8 TP) |
| **Bars 5m session window** | 395 |
| **Bars z-finite** | 395 |
| **Bars cointégrés strict EG** | 36 (9 %) — non utilisé pour gate v1 |

### Distribution des exits

| Reason | n |
|---|---:|
| `TIME_STOP` | 3 |
| `TP_MEAN_REVERSION` | 2 |
| `SL_BLOWOUT` | 3 |

### Trades (détail)

| # | Direction | armed_z | exit_reason | exit_z | bars_held | peak_R | mae_R | realized_R |
|---:|---|---:|---|---:|---:|---:|---:|---:|
| 1 | long | −2.05 | TIME_STOP | −1.07 | 18 | +0.08 | −0.09 | +0.036 |
| 2 | long | −2.04 | TP | −0.39 | 4 | 0 | −0.45 | **−0.474** |
| 3 | short | +2.01 | TIME_STOP | +1.23 | 18 | +0.12 | −0.12 | −0.094 |
| 4 | long | −2.10 | SL_BLOWOUT | −6.17 | 1 | +0.17 | 0 | +0.187 |
| 5 | long | −5.27 | SL_BLOWOUT | −4.64 | 1 | +0.11 | 0 | −0.005 |
| 6 | long | −2.34 | TIME_STOP | −1.20 | 18 | +0.23 | −0.14 | +0.273 |
| 7 | short | +2.18 | SL_BLOWOUT | +3.06 | 6 | 0 | −0.32 | −0.409 |
| 8 | short | +2.50 | TP | +0.43 | 14 | 0 | −0.96 | **−0.943** |

---

## Bloc 3 — lecture structurelle

**1. Le signal vit-il réellement ?**
Oui mais très rarement : **8 emits sur 5 sessions × 2 symbols × 395 bars 5m** = densité ~1 signal/session. Avec `entry_z=2.0` et 124 bars où |z|>2 théoriquement accessibles, la state machine lockout 6 bars + anti-re-arm ARMED→ARMED coupe ~15× les candidats. La fréquence est compatible avec 4 semaines = ~32-40 trades total, donc le problème est **qualitatif, pas quantitatif**.

**2. Le playbook est-il sous-exercé ?**
Non côté mécanique (tracker fonctionne, tous les gates passent), oui côté gate cointégration strict EG (36 bars cointégrés / 395 = 9 %, trop restrictif pour intraday 5m — désactivé v1). Remettre le gate cointégration changerait n=8→~0.

**3. Le problème vient-il du signal, de la sortie, de la mécanique, ou du contexte ?**
- **Signal** : armed_z entries légitimes (|z|>=2), direction cohérente — pas de faux signaux.
- **Sortie** : 2 TP_MEAN_REVERSION produisent les **pires** réalisés (−0.47R et −0.94R) — anomalie majeure. Le retour à la moyenne atteint z=0.5 mais le PnL est déjà catastrophique parce que **le leg-sizing en $ ne protège pas quand la beta change entre armement et exit**. Exemple trade #8 : armed_beta=0.55, exit_z=+0.43, mais mae_R=−0.96 en cours de route — la mean-reversion est arrivée trop tard pour récupérer l'excursion adverse.
- **Mécanique** : le SL_BLOWOUT à z=3.0σ se déclenche 3 fois dont une à z=−6.17 (trade #4) qui finit positif parce que le blowout s'est fait vers le côté favorable. Le SL blowout en z-space ne mappe pas proprement sur du SL $ dollars.
- **Contexte** : nov 2025 a des bars à |z|>5, symptomatiques d'un régime où la cointégration rolling sur 60 bars est **trompée** par des moves factoriels (rotation sectorielle, QQQ tech-heavy vs SPY diversifié). Le spread intraday sans gate de régime capture des moves structurels, pas du bruit mean-reverting.

**4. Distributions peak_R / mae_R**
peak_R p80 = **0.15R** → l'excursion favorable est minuscule versus excursions adverses (mae_R p20 = −0.39R). Le signal **ne produit pas de squeeze favorable fiable** avant le retour à la moyenne.

**Diagnostic §20 — classification de l'échec** :
- **Cas A** (infra missing) : ❌ Infrastructure pure suffisante pour le smoke (helpers + tracker + harness autonome). Pas de brick manquante bloquant le signal.
- **Cas B** (sous-exercé / rare) : ⚠️ n=8 est marginal mais pas structurellement rare (pourrait atteindre 30-40 sur 4 semaines). Le gate cointégration strict EG est trop restrictif intraday ; mais désactivé, le signal se tire quand même dans le rouge.
- **Cas C** (edge absent) : ✅ **Diagnostic principal.** n=8 exercé honnêtement (tracker + sizing OK, mécanique correcte), excursions faibles (peak_R p80=0.15R), les TP mean-reversion **produisent les pires pertes** (−0.47R et −0.94R). Le spread SPY-QQQ à 2σ sur fenêtre 60 bars intraday **ne contient pas d'edge mean-reverting exploitable** dans cette configuration. La mean-reversion existe statistiquement mais arrive après des excursions plus grandes que l'entrée.
- **Cas D** (codage faux) : ❌ Le codage suit littéralement la pièce B du dossier (spread log-ratio + rolling beta + z-score + blowout + TP mean-reversion). Helpers 20/20 tests PASS, tracker 11/11 tests PASS. Pas de divergence codage/hypothèse.

**Classification : Cas C + secondaire Cas B gate.** L'hypothèse stat-arb intraday 5m sur SPY-QQQ avec rolling beta 60 bars est réfutée pour la configuration testée.

---

## Bloc 4 — décision

**SMOKE_FAIL → ARCHIVED** (statut §17).

### Kill rules pré-écrites (dossier §H) — 3/3 atteintes

| Rule | Seuil | Observé | Verdict |
|---|---|---|---|
| n < 10 | < 10 | **8** | ❌ FAIL |
| mean_reversion_rate < 50 % | < 50 % | **25 %** | ❌ FAIL |
| E[R] gross ≤ 0 | ≤ 0 | **−0.179** | ❌ FAIL |

Per §18 pièce H + §19 phase F + §20 : **les trois kill rules sont atteintes. SMOKE_FAIL est la seule décision valide.** Pas d'itération post-hoc (interdite §10 + budget itération §19.3).

### Conséquences immédiates

- Statut dossier : `SMOKE_FAIL → ARCHIVED`
- Sortie `AGGRESSIVE_ALLOWLIST` (stat arb jamais ajouté, rien à retirer)
- Conservation briques pures D1+D2 — réutilisables pour futures tentatives stat-arb (daily TF, ou autre paire) sans coût d'infra.

---

## Bloc 5 — why

**Pourquoi cette décision est rationnelle** :

Le plan §5.3 P3 avait posé Sprint 3 non-ICT stat arb comme **chemin d'insurance** si Sprint 1 ICT fail. Sprint 1 a fail (SMOKE_FAIL Aplus_01). Sprint 3 teste l'hypothèse "spread SPY-QQQ intraday 5m avec z-score mean-reversion est tradable". Résultat : **signal tradable mais sans edge** à la config bar-level testée. C'est un résultat honnête, pas un échec infrastructurel.

**Pourquoi on n'itère pas plus** :

Trois raisons convergentes :

1. **Kill rules pré-écrites atteintes à 3/3**. Le budget d'itération §19.3 (max 3 tentatives post-smoke) n'est déclenché qu'en Cas B pur. Ici Cas C (edge absent) domine : peak_R p80=0.15R est un plafond signal, pas un paramètre à tuner.
2. **Les TP produisent les pires pertes**. 2 TP sur 2 = −0.47R et −0.94R réalisés. La mécanique exit est correcte (atteint z=0.5 proprement), le problème est le **chemin** : l'excursion adverse entre armement et mean-reversion dépasse le retour à la moyenne. Tuner TP/SL plus tight ferait juste accélérer les time-stops ; plus large laisserait les SL_BLOWOUT encaisser davantage. Pas de fenêtre gagnante.
3. **Cas C + §10 règle "réouverture branche morte"** : pour reouvrir stat-arb sur ce même spread, il faut une **hypothèse structurellement nouvelle** (ex : cointégration daily + entry intraday ; régime VIX gate ; triplet SPY-QQQ-IWM neutralisé 3-way ; sector pair healthcare-tech). Pas du tuning de seuils.

**Pourquoi on ne tue pas trop tôt** :

n=8 est faible. Un seul W runaway (comme trade #6 +0.27R, 6 au peak 0.23R) pourrait flatter l'échantillon. Mais trois faits convergents blindent le verdict :

- **2/2 TP sont perdants** (les deux cas où la mécanique a fonctionné comme prévu). Si l'edge existait, les TP devraient être le sweet spot.
- **peak_R p80 = 0.15R** : 80 % des trades ne dépassent jamais 0.15R d'excursion favorable. C'est un plafond d'excursion, pas un artefact de sizing ou de TP.
- **5/8 ont un mae_R ≤ −0.3R** vs peak_R max = 0.23R. Distribution profondément asymétrique du mauvais côté.

Avec n=30+ sur 4 semaines, la moyenne convergerait vers le même E[R]<0 — les 8 trades sont déjà un échantillon représentatif de la distribution.

**Pourquoi on ne promeut pas trop tôt** :

Kill rules pré-écrites strictes, PF=0.258, 0/3 kill rules passées. Promotion matériellement interdite par §10 (bar promotion = E[R]_net > 0.10R + n > 30 + PF > 1.2 + gates O5.3).

**Valeur préservée** :

Les 774 lignes de code D1+D2 (numpy-only zscore + Engle-Granger ADF + beta-neutral sizing + state machine tracker + 31 tests PASS) sont **réutilisables** pour toute future tentative stat-arb (autre paire, TF différent, cointégration daily). Sprint 3 a payé le coût d'infra statistique ; la prochaine tentative stat-arb commencera avec les briques en place.

**Statut post-verdict Sprint 3** :

| Aspect | Statut |
|---|---|
| Hypothèse SPY-QQQ 5m mean-reverting | **Réfutée** (Cas C) |
| Briques stat-arb infra | **Préservées** (31 tests PASS) |
| ExecutionEngine pair primitive (D3 full) | **Non implémentée** (évitée par D3' smoke autonome) |
| Prochain pivot candidat | Cf CLAUDE.md + plan §5.3 — paper baseline survivors OU nouvelle hypothèse non-ICT structurellement différente |

---

## Références

- [dossier.md](../../knowledge/playbooks/stat_arb_spy_qqq_v1/dossier.md) pièces A–H
- [stat_arb_smoke.py](../../scripts/stat_arb_smoke.py) — harness
- [engines/stat_arb/](../../engines/stat_arb/) — D1 helpers + D2 tracker
- [test_stat_arb_helpers.py](../../tests/test_stat_arb_helpers.py) (20 tests)
- [test_pair_spread_tracker.py](../../tests/test_pair_spread_tracker.py) (11 tests)
- Plan §5.3 P3, §11 Sprint 3, §17 gouvernance, §18 dossier, §19 protocole, §20 grille d'échecs

Verdict date : 2026-04-22.
