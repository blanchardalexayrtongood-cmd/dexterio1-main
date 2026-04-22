# Prochaines étapes — décision post-M+R.3 (2026-04-21)

## État actuel (honnête)

### Ce qu'on a prouvé
- Phase M (10 candidats sous S1, 4w) : **0/10 net E[R] > 0**.
- R.3 (Aplus_03 + TP calib 2.0→0.70R) : E[R] -0.047 → -0.055. Le signal IFVG isolé **n'a pas d'edge à 1:1**.
- Phase D.2 + D.1 : 0/6 MASTER families vraiment implémentées. Les 7 "faithful" empruntent du vocabulaire.
- Engine sanity 33/33 PASS + slippage budget mesuré (-0.065R/trade).

### Ce qui reste **non testé**
- **Family A full** (Aplus_01 Sweep+IFVG+**Breaker**) — jamais instanciée.
- **Family B** (Aplus_04 D/4H bias + 15m BOS + entry confluence) — jamais instanciée.
- **Family F** (Aplus_02 Premarket Sweep + 5m continuation) — jamais instanciée.

### Bloquants schema confirmés (grep-backed)
| Feature MASTER | Présence code | Impact |
|---|---|---|
| `setup_tf: 15m` | ✅ **Supporté** ([playbook_loader.py:125, 427, 891](backend/engines/playbook_loader.py#L125)) | Aplus_04 buildable avec schema actuel — mais TP reste fixed RR (même pathologie R.3). |
| `tp_logic: liquidity_draw` | ❌ **Absent partout** (0 hit dans `backend/engines/`, seul champ TP : `take_profit_logic` RR-based) | Aplus_01/02/04 **ne peuvent pas** exprimer un TP MASTER (session high/low). R.3 démontre : TP fixed RR est fundamentally mal-adapté au signal IFVG. |
| Premarket session context | ❌ Sessions chargées : NY, LONDON, ASIA seulement | Aplus_02 (Family F) bloqué. |
| 1m confirm-in-zone state machine | ❌ Gate actuelle = `candles_1m[-1].close` filter, pas queue setups → wait → 1m tick in zone → execute | Tous les Aplus MASTER attendent ça. S1 le simule de façon minimale et R.3 prouve que ce simple filter ne suffit pas. |

## Réponse à la question de fond (user)

> *« Est-ce que notre architecture YAML actuelle n'est pas suffisante pour exprimer un vrai setup MASTER ? »*

**OUI, confirmé repo-backed.** Les 4 extensions schema ci-dessus sont bloquantes. Sans elles, les 3 Aplus MASTER manquants auront la **même pathologie que Aplus_03 v1/R.3** : TP fixed inadapté à peak_R empirique, entry sans vrai pipeline 5m→1m, et certains (Aplus_02) même pas chargeables dans le pipeline session.

## 3 options (cost/benefit)

### Option A — C-new.1 schema extension (1-2 jours engine work)
**Portée** :
1. `tp_logic: liquidity_draw` + pipeline lookup (daily_liquidity_pools → setup pairing).
2. Premarket session context (Aplus_02).
3. (Optionnel) 1m confirm-in-zone state machine.

**Gate** : test unitaire `tp_logic: liquidity_draw` produit TP = nearest session pool above/below entry.

**Puis** : Aplus_01/02/04 chacun 1 run 4w. Si ≥1 product-grade → 1er vrai MASTER validé. Si 0/3 → bear case Family A/B/F renforcé.

**Risque** : 1-2 jours + 3 × 0.5j tests = ~3j pour potentiellement 0 product-grade. Mais **seule option qui teste vraiment MASTER**.

### Option B — Aplus_04 minimum viable (Family B, buildable en l'état)
**Portée** : créer `Aplus_04_HTF_15m_BOS` avec `setup_tf: 15m` + `require_htf_alignment: D` + TP fixed 1.0R (calibré bas comme R.3).

**Gate** : 1 run 4w.

**Puis** :
- Si net E[R] > 0 → 1er positif.
- Si négatif → confirme que le signal 15m BOS seul, même aligné, n'a pas d'edge à TP fixed. → Schema extension devient urgente.

**Risque** : probablement même pathologie que R.3 (TP trop bas = winners < losers). Mais **0 jour engine work**, juste YAML.

### Option C — Pivot terminal + Polygon 18m data
**Portée** : accepter que le corpus 4 semaines × 4 mois = trop court pour détecter edge faible. Commander Polygon, re-charger 18 mois.

**Puis** : re-run fair audit 4-semaines parallèles sur 18 mois → bien plus de stat-power. Peut-être qu'un playbook actuel (Engulfing ?) a edge ≥ 0 sur un corpus plus large.

**Risque** : paie pour data + plusieurs jours d'integration sans schema extension → les **3 MASTER families restent non testées**.

## Recommandation

**Option B d'abord (coût ~2h), puis Option A.**

Raison : Aplus_04 en l'état teste si le pipeline `setup_tf: 15m → HTF alignment → execute` produit quelque chose de différent du bruit sans 1 ligne de code. Si **ne passe pas** (probable) → bear case Family B renforcé, et **Option A devient justifiée repo-backed** (on aura testé toutes les Families buildables sans schema extension → schema est vraiment le bloquant).

Si Aplus_04 **passe** (improbable mais possible) → 1er product-grade sans engine work, et on sait qu'un signal HTF-aligné 15m BOS a edge. Immense valeur de signal pour choisir quoi builder ensuite.

## Règles absolues maintenues

- Ne pas empiler de leviers sur un signal asymptoté. Liquidity_Sweep_Scalp & Aplus_03 ont eu 3-4 leviers chacun, null. **STOP sur ces 2.**
- Ne pas créer Aplus_01/02 sans schema `tp_logic: liquidity_draw` (preuve R.3 : TP fixed ≠ MASTER).
- Ne pas promouvoir un playbook sans n ≥ 30 ET cross-weeks ≥ 3/4 positives.

## Points décision user

1. **Go/No-Go Option B** (Aplus_04 minimal, ~2h).
2. **Si Option B fail** : go/no-go **Option A** (1-2 jours engine work pour `tp_logic: liquidity_draw`).
3. **Ordre** : Option B avant A (learning cheap first), ou sauter directement A (deep refactor) ?
