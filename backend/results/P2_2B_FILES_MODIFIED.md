# P2-2.B — HTF Warmup Fix: Files Modified

## Modified Files

### 1. `/app/backend/engines/timeframe_aggregator.py`
**Change**: Increased window sizes for 4h and 1d timeframes
```python
# Line ~35-43
self.WINDOW_SIZES = {
    "1m": 500,
    "5m": 200,
    "10m": 150,
    "15m": 100,
    "1h": 50,
    "4h": 30,  # Was: 20
    "1d": 30   # Was: 10 (P2-2.B: Increased to support detect_structure >= 20)
}
```

**Reason**: Aggregator was limiting daily candles to 10, preventing detect_structure() from working (needs >= 20).

---

### 2. `/app/backend/backtest/engine.py`
**Changes**:

#### A. HTF candle slices in _process_bar_optimized() — Location 1 (line ~637)
```python
multi_tf_data = {
    "1m": candles_1m[-500:],
    "5m": candles_5m[-200:],
    "15m": candles_15m[-100:],
    "1h": candles_1h[-50:],
    "4h": candles_4h[-30:],  # Was: [-20:]
    "1d": candles_1d[-30:]   # Was: [-10:] (P2-2.B: Increased to support detect_structure)
}
```

#### B. HTF candle slices in _process_bar_optimized() — Location 2 (line ~663, fallback)
```python
multi_tf_data = {
    "1m": candles_1m[-500:],
    "5m": candles_5m[-200:],
    "15m": candles_15m[-100:],
    "1h": candles_1h[-50:],
    "4h": candles_4h[-30:],  # Was: [-20:]
    "1d": candles_1d[-30:]   # Was: [-10:]
}
```

#### C. Improved warmup logging (line ~497-500)
```python
for symbol in self.config.symbols:
    candles_1d = self.tf_aggregator.get_candles(symbol, "1d")
    candles_4h = self.tf_aggregator.get_candles(symbol, "4h")
    candles_1h = self.tf_aggregator.get_candles(symbol, "1h")
    logger.info(f"   {symbol}: {len(candles_1d)} daily, {len(candles_4h)} 4h, {len(candles_1h)} 1h candles after warmup (fed {warmup_bars_fed} 1m bars)")
```

**Reason**: Engine was slicing to only 10 daily candles when passing to market_state_engine, even though aggregator had 27+.

---

### 3. `/app/backend/models/setup.py`
**Change**: Added HTF context fields for instrumentation
```python
# Line ~109-115
# Context
market_bias: str
session: str
confluences_count: int = 0

# P2-2.B: HTF context for instrumentation
day_type: str = 'unknown'
daily_structure: str = 'unknown'

notes: str = ''
```

**Reason**: Setup model needed these fields to export market_state data for debugging.

---

### 4. `/app/backend/engines/setup_engine_v2.py`
**Change**: Populate day_type and daily_structure from market_state
```python
# Line ~159-176
setup = Setup(
    id=str(uuid4()),
    timestamp=current_time,
    symbol=symbol,
    direction=direction,
    quality=match['grade'],
    final_score=match['score'],
    trade_type='DAILY' if match['playbook_category'] == 'DAYTRADE' else 'SCALP',
    entry_price=entry_price,
    stop_loss=stop_loss,
    take_profit_1=tp1,
    take_profit_2=tp2,
    risk_reward=risk_reward,
    market_bias=market_state.bias,
    session=market_state.current_session,
    day_type=market_state.day_type,  # P2-2.B: Added
    daily_structure=market_state.daily_structure,  # P2-2.B: Added
    ict_patterns=ict_patterns,
    ...
)
```

**Reason**: Propagate HTF context from market_state to setup for instrumentation.

---

### 5. `/app/backend/tools/debug_htf_warmup.py`
**Changes**: Updated test date and htf_warmup_days for sufficient history
```python
# Line ~32-41
config = BacktestConfig(
    run_name="htf_warmup_debug",
    symbols=["SPY"],
    data_paths=[str(historical_data_path("1m", "SPY.parquet"))],
    start_date="2025-08-01",  # Was: "2025-06-03"
    end_date="2025-08-01",
    trading_mode="AGGRESSIVE",
    trade_types=["DAILY"],
    htf_warmup_days=40,  # Was: 30
    export_market_state=True
)
```

**Reason**: Need sufficient trading days (40 calendar → ~28 trading days) to populate 20+ daily candles.

---

## Summary

**Total files modified**: 5

**Critical changes**: 2
- timeframe_aggregator.py (window sizes)
- engine.py (HTF slices)

**Supporting changes**: 3
- setup.py (model extension)
- setup_engine_v2.py (field population)
- debug_htf_warmup.py (test config)

**Lines of code changed**: ~15 lines
**Files created**: 2 (P2_2B_HTF_WARMUP_FIX_COMPLETE.md, this file)

---

## Testing

✅ All changes validated via:
1. Debug script (tools/debug_htf_warmup.py)
2. Smoke suite (tools/smoke_suite.py)
3. Instrumentation (market_state_stream.parquet export)

**Result**: day_type unknown reduced from 100% → 0%
