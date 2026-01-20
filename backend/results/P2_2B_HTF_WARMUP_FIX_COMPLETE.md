# P2-2.B — HTF Warmup Fix Complete

## 📌 Problème identifié

**day_type retournait 100% "unknown"**, bloquant le playbook News_Fade.

### Cause racine

La chaîne causale complète était :

1. ✅ Le **prefeed HTF warmup** fonctionnait correctement (chargement des données)
2. ✅ L'**agrégateur** stockait correctement les bougies daily
3. ❌ **MAIS** le moteur de backtest ne passait que **10 daily candles** à `market_state_engine`
4. ❌ `detect_structure()` nécessite **≥20 candles** → retournait "unknown"
5. ❌ `calculate_day_type()` dépend de `daily_structure` → retournait "unknown"

### Limites hardcodées trouvées

```python
# engines/timeframe_aggregator.py
WINDOW_SIZES = {
    "1d": 10,  # ❌ Insuffisant pour detect_structure
    "4h": 20,
}

# backtest/engine.py (2 locations)
multi_tf_data = {
    "1d": candles_1d[-10:],  # ❌ Limite hardcodée
    "4h": candles_4h[-20:],
}
```

---

## 🔧 Correctif appliqué

### 1. Augmentation des window sizes (timeframe_aggregator.py)

```python
self.WINDOW_SIZES = {
    ...
    "4h": 30,  # 20 → 30
    "1d": 30   # 10 → 30 (support detect_structure >= 20)
}
```

### 2. Augmentation des slices dans engine.py (2 locations)

**Location A — Recalcul sur événement HTF (ligne ~637)**
```python
multi_tf_data = {
    "1m": candles_1m[-500:],
    "5m": candles_5m[-200:],
    "15m": candles_15m[-100:],
    "1h": candles_1h[-50:],
    "4h": candles_4h[-30:],  # 20 → 30
    "1d": candles_1d[-30:]   # 10 → 30
}
```

**Location B — Fallback si cache manquant (ligne ~663)**
```python
multi_tf_data = {
    "1m": candles_1m[-500:],
    "5m": candles_5m[-200:],
    "15m": candles_15m[-100:],
    "1h": candles_1h[-50:],
    "4h": candles_4h[-30:],  # 20 → 30
    "1d": candles_1d[-30:]   # 10 → 30
}
```

### 3. Ajout des champs HTF au modèle Setup (setup.py)

```python
class Setup(BaseModel):
    ...
    # P2-2.B: HTF context for instrumentation
    day_type: str = 'unknown'
    daily_structure: str = 'unknown'
```

### 4. Population des champs HTF (setup_engine_v2.py)

```python
setup = Setup(
    ...
    day_type=market_state.day_type,
    daily_structure=market_state.daily_structure,
    ...
)
```

### 5. Amélioration des logs warmup (engine.py)

```python
logger.info(f"   {symbol}: {len(candles_1d)} daily, {len(candles_4h)} 4h, {len(candles_1h)} 1h candles after warmup (fed {warmup_bars_fed} 1m bars)")
```

---

## ✅ Validation

### Script de debug exécuté

```bash
cd /app/backend && python tools/debug_htf_warmup.py
```

### Résultats AVANT correctif

```
- day_type unknown: 100.0%
- daily_structure unknown: 100.0%
- HTF candles passed to market_state: 10 daily (< 20 minimum)
```

### Résultats APRÈS correctif

```
✅ day_type unknown: 0.0% (< 100%)
✅ daily_structure unknown: 0.0%
✅ HTF candles passed to market_state: 27+ daily (>= 20 minimum)
✅ Smoke suite: ALL TESTS PASSED
```

### Artefacts de preuve

- `/app/backend/results/htf_warmup_debug_2025-08-01_after.json`
- Smoke suite report: `/app/backend/results/P2_smoke_suite_report.json`

---

## 📊 Impact

### Déblocage immédiat

- ✅ `News_Fade` playbook n'est plus rejeté pour `day_type_mismatch`
- ✅ Structure daily correctement calculée (`uptrend`, `downtrend`, `range`)
- ✅ day_type correctement calculé (`trend`, `manipulation_reversal`, `range`)

### Impact attendu

- **Volume de trades** : Augmentation attendue (News_Fade + autres playbooks dépendant de day_type)
- **TOTAL R** : Progression vers l'objectif "MAX R"

---

## 🧪 Tests de non-régression

✅ **Smoke suite complète passante** (82.9s)

- Syntaxe check
- Unit tests
- Backtest 1d
- Backtest 5d
- Metrics validation

---

## 📝 Commandes reproductibles

### Debug HTF warmup

```bash
cd /app/backend
python tools/debug_htf_warmup.py
```

### Smoke suite

```bash
cd /app/backend
python tools/smoke_suite.py
```

---

## 🎯 Prochaines étapes (P2-2.C)

Volatility Engine :
- Vérifier si une mesure de volatilité est définie dans playbooks.yml
- Si oui → implémenter
- Sinon → neutraliser avec reason_code

---

## ⚠️ Notes importantes

- **Aucune modification de la logique de trading** (strictement wiring HTF)
- **Patch minimal et ciblé**
- **Preuves repo-level obligatoires** fournies
- **Micro-backtests uniquement** (1j, 5j) pour validation technique
- **Backtests mensuels** à effectuer par l'utilisateur en local

---

**Critère de succès P2-2.B : ✅ VALIDÉ**

`day_type_unknown_pct` < 100% → **0.0%** atteint.
