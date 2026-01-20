# BASELINE REPRODUCTIBLE - COMMANDES

## Source

La baseline P2 Phase 0 est basée sur le run **`rolling_2025-06`** existant dans le repo.

## Commande Exacte (Reproduction)

```bash
cd /app
python backend/backtest/run_rolling_30d.py --month 2025-06
```

## Configuration

- **Période:** Juin 2025 (2025-06-02 → 2025-06-30)
- **Symboles:** SPY, QQQ
- **Mode:** AGGRESSIVE
- **Trade Types:** DAILY + SCALP
- **Capital Initial:** $50,000
- **Risk:** 2% base (mode AGGRESSIVE)
- **Slippage:** 0.02% (défaut config)
- **Fees:** 0 (paper trading)

## Dataset

- **Fichiers:**
  - `/app/data/historical/1m/SPY.parquet`
  - `/app/data/historical/1m/QQQ.parquet`
- **Timeframe:** 1m (minute bars)
- **Total bars processed:** ~216,574 (combiné SPY+QQQ juin 2025)

## Métriques Produites

Extrait de `/app/backend/results/summary_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.json`:

```json
{
  "total_trades": 12,
  "wins": 8,
  "losses": 4,
  "winrate": 66.67,
  "total_pnl_r": 21.176,
  "profit_factor": 6.754,
  "expectancy_r": 1.765,
  "max_drawdown_r": 1.0,
  "avg_win_r": 3.147,
  "avg_loss_r": -1.0
}
```

## Artefacts Générés

1. **Summary:**
   `/app/backend/results/summary_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.json`

2. **Trades:**
   - Parquet: `/app/backend/results/trades_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.parquet`
   - CSV: `/app/backend/results/trades_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.csv`

3. **Equity:**
   `/app/backend/results/equity_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.parquet`

4. **Playbook Stats:**
   `/app/backend/results/playbook_match_stats_2025-06.json`

## Baseline Reference (Copie Stable)

Pour P2, ces fichiers ont été copiés dans des noms stables:

```bash
cp backend/results/trades_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.parquet \
   backend/results/baseline_trades_reference.parquet

cp backend/results/equity_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.parquet \
   backend/results/baseline_equity_reference.parquet

# Summary consolidé dans:
backend/results/baseline_reference.json
```

## Formules Métriques (metrics.py)

- **total_R:** `sum(pnl_R_account)` pour tous les trades
- **profit_factor:** `gross_profit_R / abs(gross_loss_R)` (BE exclu)
- **expectancy_R:** `mean(r_multiple)` (BE inclus)
- **max_drawdown_R:** `max(peak_equity_R - trough_equity_R)`
- **r_multiple:** `pnl_dollars / risk_dollars` (par trade)
- **pnl_R_account:** `pnl_dollars / base_r_unit` où `base_r_unit = capital * 0.02`

Formules verrouillées dans `/app/backend/backtest/metrics.py` (ligne 1-413).

## Reproductibilité

### Étape 1: Vérifier Data

```bash
ls -lh data/historical/1m/SPY.parquet
ls -lh data/historical/1m/QQQ.parquet
```

### Étape 2: Run

```bash
python backend/backtest/run_rolling_30d.py --month 2025-06
```

### Étape 3: Vérifier Output

```bash
cat backend/results/summary_rolling_2025-06_AGGRESSIVE_DAILY_SCALP.json | python -m json.tool
```

### Étape 4: Comparer

Comparer les métriques avec baseline_reference.json:
- `total_trades` doit être identique (12)
- `total_pnl_r` doit être identique (~21.176)
- `profit_factor` doit être identique (~6.754)

## Seed / Déterminisme

⚠️ **Note:** Le backtest engine utilise des timestamps réels et un ordre chronologique strict, donc les résultats sont **déterministes** pour une période/dataset donné. Pas de random seed nécessaire.

## Notes Importantes

1. **Mode AGGRESSIVE:**
   - Bypass candlestick patterns (tracé via `relaxed_bypasses`)
   - Relaxation sur certaines règles de confluence
   - Risk tier peut varier (1R → 2R selon conditions)

2. **Funnel:**
   - 34,767 setups générés totaux (raw)
   - 16 playbook matches (après gating/scoring/risk)
   - 12 trades exécutés (certains setups rejetés par risk engine)

3. **Rejections principales:**
   - `candlestick_patterns_missing`: 16 (bypass actif)
   - `volatility_insufficient`: 11
   - `news_events_day_type_mismatch`: 2

---

**Dernière validation:** 2025-12-27  
**Run ID:** rolling_2025-06  
**Status:** Baseline stable ✅
