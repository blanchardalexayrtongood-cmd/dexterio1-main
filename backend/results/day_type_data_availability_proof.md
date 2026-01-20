# day_type Data Availability Proof

**Date:** 2025-12-27  
**Objectif:** Prouver où day_type existe et comment l'auditer sans rerun lourd

---

## 1) Où day_type est-il stocké/persisté ?

### Code (Définitions)

#### MarketState Model (`backend/models/market_data.py`, ligne 68)
```python
class MarketState(BaseModel):
    # ...
    day_type: str = 'unknown'  # New field for playbook filtering
```

✅ **day_type EXISTE dans MarketState**

#### MarketStateEngine (`backend/engines/market_state.py`, ligne 171)
```python
def calculate_day_type(self, daily_structure: str, ict_patterns: List) -> str:
    """
    Calculate day_type for playbook filtering (P1 implementation)
    
    Returns:
        'trend', 'manipulation_reversal', 'range', or 'unknown'
    """
    # Logic:
    # - range structure → 'range'
    # - sweep + bos → 'manipulation_reversal'
    # - uptrend/downtrend + bos → 'trend'
    # - unknown structure → 'unknown'
```

✅ **day_type EST CALCULÉ** (logique implémentée)

### Persistence (Artefacts)

#### Trades Parquet (`trades_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.parquet`)
```python
# Columns available:
['trade_id', 'timestamp_entry', 'timestamp_exit', 'date', 'month', 
 'symbol', 'playbook', 'direction', 'trade_type', 'quality', 
 'entry_price', 'exit_price', 'stop_loss', 'take_profit_1', 
 'position_size', 'duration_minutes', 'risk_pct', 'pnl_dollars', 
 'risk_dollars', 'r_multiple', 'risk_tier', 'pnl_R_account', 
 'cumulative_R', 'outcome', 'exit_reason']
```

❌ **day_type NOT in trades parquet**

#### Setup Parquet
```bash
$ ls backend/results/*setup*.parquet
# No files found
```

❌ **Setup parquet n'existe pas** (setups non persistés)

#### JSON Summaries
```bash
$ ls backend/results/*.json | grep -E "market|state|setup"
# No files found
```

❌ **Pas de market_state JSON** dans artifacts

---

## 2) day_type dans artefacts existants (rolling_2025-06)

### Vérification Factuelle

```bash
# Trades
$ python -c "import pandas as pd; df = pd.read_parquet('backend/results/trades_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.parquet'); print('day_type' in df.columns)"
False
```

```bash
# Summary JSON
$ grep -i "day_type" backend/results/summary_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.json
# No matches
```

```bash
# Playbook stats
$ grep -i "day_type" backend/results/playbook_match_stats_2025-06.json
# No matches (only rejection counts, not field values)
```

### Verdict

**❌ day_type N'EST PAS PERSISTÉ** dans les artefacts rolling_2025-06

**Raison:**
- day_type est calculé dans `MarketState` (runtime)
- MarketState est passé à `SetupEngine` et `PlaybookEvaluator`
- Mais ni `Setup` ni `Trade` ne sérialisent `day_type` dans parquet/JSON

**Conséquence:**
- On ne peut PAS extraire day_type depuis artefacts existants
- Audit nécessite un nouveau run avec instrumentation

---

## 3) Comment auditer day_type sans rerun lourd ?

### ❌ Branche A (Extraction) - NON APPLICABLE

day_type n'est pas dans artefacts existants → **extraction impossible**

### ✅ Branche B (Instrumentation Minimale) - APPLICABLE

**Stratégie:**
1. Ajouter export day_type dans un parquet léger (`market_state_stream.parquet`)
2. Run 1D ou 5D avec date slicing
3. Pas de modification logique trading
4. Audit depuis le stream

#### Implementation

**Step 1: Créer export stream dans BacktestEngine**

Ajouter dans `backend/backtest/engine.py` (après setup generation):

```python
# Export market_state pour audit (P2-2.B instrumentation)
if self.config.export_market_state:  # Flag optionnel
    market_state_records.append({
        'timestamp': current_ts,
        'symbol': symbol,
        'day_type': market_state.day_type,
        'daily_structure': market_state.daily_structure,
        'bias': market_state.bias,
    })

# À la fin du run:
if market_state_records:
    df = pd.DataFrame(market_state_records)
    df.to_parquet(f'{output_dir}/market_state_stream_{run_name}.parquet')
```

**Step 2: Run 1D avec export**

```python
config = BacktestConfig(
    start_date="2025-06-03",
    end_date="2025-06-03",  # 1 jour = rapide
    export_market_state=True,  # Enable stream
    # ... reste identique
)
```

**Step 3: Audit depuis stream**

```python
import pandas as pd
df = pd.read_parquet('market_state_stream_audit_1d.parquet')

# Distribution day_type
print(df['day_type'].value_counts(normalize=True))

# Unknown rate par symbole
print(df.groupby('symbol')['day_type'].apply(lambda x: (x=='unknown').mean()))
```

---

## Root Cause: Pourquoi day_type = unknown ?

### Code Inspection (`backend/engines/market_state.py:171`)

```python
def calculate_day_type(self, daily_structure: str, ict_patterns: List) -> str:
    # Rule 4: Unknown structure → unknown day_type
    if daily_structure == 'unknown':
        return 'unknown'
```

**Dépendance:** day_type dépend de `daily_structure`

**Question:** Pourquoi daily_structure = unknown ?

### Upstream: daily_structure calculation

```python
# backend/engines/market_state.py:28
structures['daily_structure'] = detect_structure([
    {'high': c.high, 'low': c.low, 'close': c.close, 'timestamp': c.timestamp}
    for c in daily  # ← List[Candle] from daily timeframe
])
```

**Dépendance:** daily_structure dépend de `daily` candles (HTF data)

**Question:** Est-ce que daily candles sont fournis au MarketStateEngine ?

### Wiring Check

À vérifier dans `backend/backtest/engine.py`:
1. Est-ce que daily/H4/H1 bars sont chargées ?
2. Est-ce que MarketStateEngine reçoit ces bars ?
3. Est-ce qu'il y a un "warmup" HTF (lookback avant start_date) ?

**Hypothèse probable:**
- Date slicing coupe les données 1m MAIS
- Pas assez de contexte daily/H4/H1 pour calculer structure
- Résultat: daily_structure = 'unknown' → day_type = 'unknown'

---

## Prochaines Étapes (P2-2.B)

### B1 - Audit avec Instrumentation (Branche B)

**Script:** `backend/tools/audit_day_type_instrumented.py`

1. Ajouter flag `export_market_state` à `BacktestConfig`
2. Instrumenter `BacktestEngine` pour exporter market_state stream
3. Run 1D avec export
4. Générer:
   - `day_type_distribution_1d_sample.json`
   - `news_fade_rejection_audit_1d_sample.json`

**Durée estimée:** ~1-2 min (1D avec instrumentation légère)

### B2 - Root Cause (Code Inspection)

**Document:** `backend/results/day_type_root_cause.md`

Questions à répondre:
1. Est-ce que daily/H4/H1 sont chargées dans BacktestEngine ?
2. Est-ce que MarketStateEngine reçoit ces HTF bars ?
3. Est-ce qu'il y a un warmup period HTF ?
4. Pourquoi daily_structure = 'unknown' ?

**Fichiers à inspecter:**
- `backend/backtest/engine.py` (HTF loading)
- `backend/engines/market_state.py` (calculate_day_type)
- `backend/utils/indicators.py` (detect_structure)

### B3 - Patch (Si wiring manquant)

**Cas 1:** HTF bars pas chargées
→ Ajouter loading daily/H4/H1 dans engine

**Cas 2:** Pas de warmup HTF
→ Ajouter lookback 30 jours daily (pour structure calculation)

**Cas 3:** detect_structure broken
→ Fix logic ou reason_code si non défini dans spec

---

## Conclusion

### Réponses aux 3 Questions

**1. Où day_type est stocké/persisté ?**
- ✅ Défini dans `MarketState` model
- ✅ Calculé dans `MarketStateEngine.calculate_day_type()`
- ❌ Pas persisté dans trades parquet
- ❌ Pas persisté dans JSON summaries
- ❌ Pas de setup parquet

**2. day_type présent dans artefacts rolling_2025-06 ?**
- ❌ **NON** - Aucune trace dans artefacts existants

**3. Comment auditer sans rerun lourd ?**
- ✅ **Instrumentation minimale** (Branche B)
- Ajouter export market_state stream
- Run 1D avec date slicing (rapide)
- Audit depuis stream parquet

---

**Status:** ✅ INVESTIGATION COMPLETE  
**Next:** B1 - Instrumentation + Audit 1D  
**Fichier:** `backend/results/day_type_data_availability_proof.md`
