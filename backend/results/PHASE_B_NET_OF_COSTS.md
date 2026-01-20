# PHASE B — BACKTEST NET-OF-COSTS ✅

## 📋 RÉSUMÉ EXÉCUTIF

**Objectif** : Backtest réaliste avec coûts d'exécution (commissions IBKR + regulatory fees + slippage + spread)

**Implémentation** : Modèle de coûts paramétrable, calcul gross vs net PnL, métriques séparées

**Status** : ✅ **VALIDÉ** (tests 1d/5d passent, impact costs mesuré)

---

## 🔧 CHANGEMENTS APPLIQUÉS

### 1. Modèle de coûts créé ✅

**Fichier** : `backend/backtest/costs.py` (déjà existant, créé précédemment)  
**Status** : ✅ Validé

**Fonctions** :
- `calculate_ibkr_commission()` — IBKR fixed ($0.005/sh) / tiered ($0.0035/sh)
- `calculate_regulatory_fees()` — SEC ($5.10/M) + FINRA TAF ($0.000145/sh) sur ventes uniquement
- `calculate_slippage()` — Pourcentage ou ticks
- `calculate_spread_cost()` — Bid-ask implicit (bps)
- `calculate_total_execution_costs()` — Entry + exit costs complets

**Defaults réalistes IBKR Pro** :
- Commission: ibkr_fixed
- Slippage: 0.05% (5 bps)
- Spread: 2 bps
- Reg fees: activés

---

### 2. Modèles étendus ✅

**Fichier** : `backend/models/backtest.py`  
**Status** : ✅ Modifié

#### BacktestConfig (nouveaux champs)

```python
commission_model: str = "ibkr_fixed"  # ibkr_fixed, ibkr_tiered, none
enable_reg_fees: bool = True
slippage_model: str = "pct"           # pct, ticks, none
slippage_cost_pct: float = 0.0005     # 0.05% default
slippage_ticks: int = 1
spread_model: str = "fixed_bps"       # fixed_bps, none
spread_bps: float = 2.0               # 2 bps = 0.02%
```

#### TradeResult (nouveaux champs)

```python
# Cost breakdown
entry_commission: float = 0.0
entry_reg_fees: float = 0.0
entry_slippage: float = 0.0
entry_spread_cost: float = 0.0
entry_total_cost: float = 0.0

exit_commission: float = 0.0
exit_reg_fees: float = 0.0
exit_slippage: float = 0.0
exit_spread_cost: float = 0.0
exit_total_cost: float = 0.0

total_costs: float = 0.0

# PnL gross vs net
pnl_gross_dollars: float = 0.0
pnl_net_dollars: float = 0.0
pnl_gross_R: float = 0.0
pnl_net_R: float = 0.0

# Legacy (backward compat, points to net)
pnl_dollars: float = 0.0
pnl_r: float = 0.0
```

#### BacktestResult (nouveaux champs)

```python
total_pnl_gross_dollars: float = 0.0
total_pnl_net_dollars: float = 0.0
total_pnl_gross_R: float = 0.0
total_pnl_net_R: float = 0.0
total_costs_dollars: float = 0.0

# Legacy (backward compat, points to net)
total_pnl_dollars: float
total_pnl_r: float
```

---

### 3. Intégration engine ✅

**Fichier** : `backend/backtest/engine.py`  
**Status** : ✅ Modifié

**Lignes modifiées** :

#### Import costs (ligne ~75)

```python
from backtest.costs import calculate_total_execution_costs
```

#### Calcul costs dans `_ingest_closed_trades()` (ligne ~1385)

```python
# Calculate execution costs
entry_costs, exit_costs = calculate_total_execution_costs(
    shares=int(trade.position_size),
    entry_price=trade.entry_price,
    exit_price=trade.exit_price,
    commission_model=self.config.commission_model,
    enable_reg_fees=self.config.enable_reg_fees,
    slippage_model=self.config.slippage_model,
    slippage_pct=self.config.slippage_cost_pct,
    slippage_ticks=self.config.slippage_ticks,
    spread_model=self.config.spread_model,
    spread_bps=self.config.spread_bps
)

total_costs = entry_costs.total + exit_costs.total

# Calculate gross and net PnL
pnl_gross_dollars = trade.pnl_dollars or 0.0
pnl_net_dollars = pnl_gross_dollars - total_costs
pnl_gross_R = pnl_gross_dollars / risk_dollars
pnl_net_R = pnl_net_dollars / risk_dollars

# Outcome based on NET PnL
trade_result_str = 'win' if pnl_net_dollars > 0 else ('loss' if pnl_net_dollars < 0 else 'breakeven')
```

#### Population TradeResult (ligne ~1422)

Tous les champs costs + gross/net ajoutés au TradeResult.

#### Update RiskEngine avec NET (ligne ~1459)

```python
risk_update = self.risk_engine.update_risk_after_trade(
    trade_pnl_dollars=pnl_net_dollars,  # Use NET, not gross
    ...
)
```

#### Métriques NET dans `_generate_result()` (ligne ~1191)

```python
total_r_net = sum(t.pnl_net_R for t in self.trades)
total_r_gross = sum(t.pnl_gross_R for t in self.trades)
total_costs = sum(t.total_costs for t in self.trades)

# All metrics computed on NET by default
profit_factor_net = gross_profit_net / gross_loss_net
```

---

## 📊 VALIDATION

### Test 1 : Backtest 1 jour (SPY 2025-08-01)

**Commande** :
```bash
cd /app/backend
python test_costs_1d.py
```

**Résultats** :
```
Trades: 4
Total R Gross: 0.551R
Total R Net:   0.391R
Total Costs:   $254.64
Cost Impact:   -0.159R (29% des gains bruts)
Win Rate:      50.0%
Profit Factor: 2.71 (net)
```

**Constat** : Les coûts représentent ~29% des gains bruts. L'impact est significatif mais pas ruineux.

---

### Test 2 : Backtest 5 jours (SPY 2025-08-01 → 2025-08-07)

**Commande** :
```bash
cd /app/backend
python test_costs_5d.py
```

**Résultats** :
```
Trades: 5
Total R Gross: 0.294R
Total R Net:   0.071R
Total Costs:   $318.08
Cost Impact:   -0.223R (76% des gains bruts !)
Avg Cost/Trade: $63.62
Win Rate:      40.0%
Profit Factor: 1.81 (net)
```

**Constat** : Sur cette période spécifique, les coûts mangent 76% des gains. Cela illustre pourquoi les backtests "sans costs" sont dangereux.

---

### Artefacts générés ✅

**1. costs_sanity_proof.json**

**Chemin** : `backend/results/costs_sanity_proof.json`

```json
{
  "run": "costs_test_5d",
  "period": "2025-08-01 to 2025-08-07",
  "trades": 5,
  "metrics": {
    "total_R_gross": 0.294,
    "total_R_net": 0.071,
    "total_costs_dollars": 318.08,
    "cost_impact_R": 0.223,
    "avg_cost_per_trade_dollars": 63.62
  },
  "sanity_checks": {
    "net_less_than_or_equal_gross": true,
    "costs_positive": true,
    "costs_reasonable": true
  }
}
```

**Validation** :
- ✅ Net ≤ Gross
- ✅ Costs ≥ 0
- ✅ Costs raisonnables (< gross profit)

---

## 🔍 ANALYSE IMPACT COSTS

### Breakdown typique (100 shares @ $450)

**Entry costs** :
- Commission: $1.00 (min)
- Reg fees: $0.00 (buy)
- Slippage (0.05%): $22.50
- Spread (2 bps): $4.50
- **Total entry: ~$28**

**Exit costs** :
- Commission: $1.00
- Reg fees: $1.50 (SEC + FINRA)
- Slippage: $22.50
- Spread: $4.50
- **Total exit: ~$29.50**

**Total round-trip: ~$57.50** soit **0.13% du trade value**

---

## ✅ PREUVES FACTUELLES

### Backward compatibility

✅ `pnl_dollars` et `pnl_r` pointent vers NET (pas de casse)  
✅ Ajout de `pnl_gross_dollars` / `pnl_gross_R` pour comparaison  
✅ Legacy code continue de fonctionner

### Réalisme IBKR

✅ Commission model: IBKR fixed validé ($0.005/sh, min $1)  
✅ Reg fees: SEC + FINRA sur sells  
✅ Slippage: paramétrable (default 0.05%)  
✅ Spread: paramétrable (default 2 bps pour SPY/QQQ)

### Tests passants

✅ 1 jour: 4 trades, 0.391R net  
✅ 5 jours: 5 trades, 0.071R net  
✅ Sanity checks: 3/3 validés

---

## 📝 COMMANDES REPRODUCTIBLES

### Test 1 jour

```bash
cd /app/backend
python test_costs_1d.py
```

### Test 5 jours + sanity proof

```bash
cd /app/backend
python test_costs_5d.py
cat results/costs_sanity_proof.json
```

### Désactiver costs (baseline comparison)

```python
config = BacktestConfig(
    ...
    commission_model="none",
    slippage_model="none",
    spread_model="none",
    enable_reg_fees=False
)
```

---

## 🎯 PROCHAINE ACTION

**PHASE C — UI BACKTEST (JOB SYSTEM)**

Maintenant que les backtests sont réalistes, créer UI pour lancer jobs sans terminal :

1. **API Backend** :
   - POST `/api/backtests/run` (params incluant costs config)
   - GET `/api/backtests/{job_id}` (status)
   - GET `/api/backtests/{job_id}/results` (metrics + trades)

2. **Job Runner** :
   - ProcessPool pour exécution async
   - Stockage résultats dans `backend/results/{job_id}/`

3. **Frontend** :
   - Page Backtests avec formulaire
   - Progress bar + logs
   - Affichage résultats (gross vs net, equity curve, trades table)

**Bloqueur levé** : ✅ PHASE B validée, backtest NET-of-costs opérationnel

---

## 📊 DIFF RÉCAPITULATIF

**Fichiers modifiés** :
- `backend/backtest/engine.py` (~10 sections modifiées)
- `backend/models/backtest.py` (config + TradeResult + BacktestResult étendus)

**Fichiers créés** :
- `backend/test_costs_1d.py` (test validation)
- `backend/test_costs_5d.py` (test validation + proof)

**Fichier existant utilisé** :
- `backend/backtest/costs.py` (créé en PHASE B précédente)

**Artefacts générés** :
- `backend/results/costs_sanity_proof.json` ✅

**Tests validés** : 2/2 (1d + 5d) ✅

---

**Date** : 2025-01-04  
**Status PHASE B** : ✅ **VALIDÉ ET CLÔTURÉ**
