# day_type Root Cause Analysis

**Date:** 2025-12-27  
**Issue:** 100% day_type = 'unknown' in backtest  
**Status:** ✅ ROOT CAUSE IDENTIFIED

---

## Executive Summary

**Primary Cause:** **Insufficient HTF warmup period**

**Chain:**
1. Date slicing cuts data at `start_date` 08:00 (market open)
2. Daily candles build progressively during the day (0 → 1 candle)
3. `detect_structure()` requires **≥ 20 candles** minimum
4. 0 daily candles at start → `daily_structure = 'unknown'`
5. `daily_structure = 'unknown'` → `day_type = 'unknown'`
6. News_Fade requires specific day_types → **rejected**

---

## Investigation Path

### Step 1: Verify HTF Candles Loading ✅

**File:** `backend/backtest/engine.py`

**Lines 508-509:**
```python
candles_4h = self.tf_aggregator.get_candles(symbol, "4h")
candles_1d = self.tf_aggregator.get_candles(symbol, "1d")
```

**Lines 678-679 (v2 path):**
```python
candles_4h = [c for c in mtf.get("4h", []) if c.timestamp <= current_time][-ROLLING_WINDOW["4h"]:]
candles_1d = [c for c in mtf.get("1d", []) if c.timestamp <= current_time][-ROLLING_WINDOW["1d"]:]
```

✅ **Verdict:** HTF candles ARE loaded and passed to MarketStateEngine

---

### Step 2: Verify MarketStateEngine Processing ✅

**File:** `backend/engines/market_state.py`

**Lines 27-31 (analyze_htf_structure):**
```python
if daily:
    structures['daily_structure'] = detect_structure([
        {'high': c.high, 'low': c.low, 'close': c.close, 'timestamp': c.timestamp}
        for c in daily
    ])
```

**Lines 171-210 (calculate_day_type):**
```python
def calculate_day_type(self, daily_structure: str, ict_patterns: List) -> str:
    # ...
    # Rule 4: Unknown structure → unknown day_type
    if daily_structure == 'unknown':
        return 'unknown'
```

✅ **Verdict:** MarketStateEngine correctly uses `daily_structure` to compute `day_type`

**Dependency confirmed:** `day_type` **depends on** `daily_structure`

---

### Step 3: Verify detect_structure() Logic 🔴

**File:** `backend/utils/indicators.py`

**Lines 1-10:**
```python
def detect_structure(candles: List[Dict[str, Any]]) -> str:
    """
    Détermine la structure du marché (uptrend, downtrend, range)
    
    Returns:
        'uptrend', 'downtrend', ou 'range'
    """
    if len(candles) < 20:  # 🔴 CRITICAL THRESHOLD
        return 'unknown'
```

🔴 **ROOT CAUSE:** `detect_structure()` requires **≥ 20 candles minimum**

---

### Step 4: Verify Actual Candles Count at Backtest Start 🔴

**Test Run (2025-06-03 start):**
```python
# At 08:00 (market open), after load_data()
candles_1d count: 0
multi_tf_candles: {}  # Empty dict

# Reason: Date slicing cuts at start_date 08:00
# No previous days loaded → no 1D candle history
```

**Proof:**
```bash
$ python -c "
engine.load_data()  # with start_date=2025-06-03
candles_1d = engine.multi_tf_candles.get('SPY', {}).get('1d', [])
print(f'candles_1d count: {len(candles_1d)}')
"
# Output: candles_1d count: 0
```

🔴 **ROOT CAUSE CONFIRMED:**
- Date slicing starts at 08:00 (day N)
- Daily candles need full day to form (close at 16:00)
- At 08:00-16:00: 0 daily candles available
- After 16:00: Only 1 daily candle (current day)
- **Never reaches 20 daily candles** during 1D backtest

---

## Causal Chain (Complete)

```
Date Slicing (2025-06-03 08:00)
    ↓
No Previous Days Data
    ↓
candles_1d = [] (0 candles)
    ↓
detect_structure(candles_1d) with len=0
    ↓
len(candles) < 20 → return 'unknown'
    ↓
daily_structure = 'unknown'
    ↓
calculate_day_type(daily_structure='unknown')
    ↓
day_type = 'unknown'
    ↓
News_Fade gating: requires day_type in ['manipulation_reversal', 'range']
    ↓
day_type='unknown' → REJECTED
    ↓
News_Fade never matches (0 trades)
```

---

## Why This Happens

### Design Issue

1. **Multi-TF Aggregator builds progressively:**
   - 1m candles: Available immediately (loaded from parquet)
   - 5m candles: Build after 5 minutes
   - 1h candles: Build after 1 hour
   - 1d candles: **Build after FULL DAY (16:00 close)**

2. **Date slicing cuts context:**
   - `start_date=2025-06-03` loads data from `2025-06-03 08:00:00`
   - No previous days → no historical 1d candles
   - First 1d candle only available at EOD (end of day)

3. **detect_structure() requires lookback:**
   - Minimum: 20 candles
   - For daily TF: **20 trading days ≈ 1 month**
   - Without warmup: **impossible to satisfy**

---

## Fix Strategies

### Option A: HTF Warmup Period (RECOMMENDED) ✅

**Concept:** Load HTF data (1d, 4h, 1h) from **before** `start_date`

**Implementation:**
1. Add `htf_warmup_days` parameter to `BacktestConfig` (default: 30)
2. When loading data:
   - Load 1m data: `start_date` → `end_date` (unchanged)
   - Load HTF aggregates: `start_date - 30 days` → `end_date`
3. Multi-TF aggregator pre-fills HTF candles from warmup period
4. At backtest start (08:00), daily candles already available (20+)

**Pros:**
- ✅ Minimal code change
- ✅ No change to trading logic
- ✅ Maintains date slicing speed benefits (1m still sliced)
- ✅ Provides realistic HTF context

**Cons:**
- Requires loading ~30 days 1d/4h/1h data (lightweight)

**Estimated Lines Changed:** ~50 lines (load_data + aggregator init)

---

### Option B: Pre-compute Daily Structure (ALTERNATIVE)

**Concept:** Run a separate daily analysis pass before 1m backtest

**Pros:**
- Separates HTF analysis from 1m execution

**Cons:**
- ❌ More complex
- ❌ Requires two-pass architecture
- ❌ Doesn't solve H4/H1 structure issues

**Verdict:** NOT RECOMMENDED (over-engineering)

---

### Option C: Reason Code (IF NO FIX POSSIBLE)

**If warmup impossible:**

Add reason code: `"htf_warmup_insufficient"`

Update News_Fade gating to explicitly state:
```python
if day_type == 'unknown':
    reason = "day_type_unknown_htf_warmup_needed"
    return False, reason
```

**Pros:**
- ✅ Honest about limitation
- ✅ No false positives

**Cons:**
- ❌ News_Fade remains blocked
- ❌ Doesn't increase volume

**Verdict:** Fallback only if Option A fails

---

## Recommended Fix: Option A (HTF Warmup)

### Implementation Plan

**File:** `backend/models/backtest.py`
```python
class BacktestConfig(BaseModel):
    # ...
    start_date: Optional[str] = None
    end_date: Optional[str] = None
    htf_warmup_days: int = 30  # NEW: HTF lookback period
```

**File:** `backend/backtest/engine.py` (load_data method)
```python
def load_data(self):
    # Calculate warmup start date
    if self.config.start_date and self.config.htf_warmup_days > 0:
        warmup_start = (
            pd.to_datetime(self.config.start_date) 
            - pd.Timedelta(days=self.config.htf_warmup_days)
        ).strftime('%Y-%m-%d')
        
        # Load 1m data: normal range (speed preserved)
        # Load HTF aggregates: extended range (warmup)
        # Pre-fill multi_tf_candles with warmup period
```

**Expected Result:**
- At backtest start (08:00), `candles_1d` has 20+ candles
- `detect_structure()` returns valid structure (uptrend/downtrend/range)
- `day_type` computed correctly
- News_Fade gating can pass

---

## Validation Plan (After Fix)

### Test 1: Warmup Verification
```python
engine.load_data()  # with htf_warmup_days=30
candles_1d = engine.multi_tf_candles.get('SPY', {}).get('1d', [])
assert len(candles_1d) >= 20, "Warmup failed"
```

### Test 2: day_type Distribution
```bash
python backend/tools/audit_day_type_instrumented.py
# Expected: day_type unknown < 30%
```

### Test 3: News_Fade Matches
```bash
# Before: 0 matches
# After: ≥ 1 match (if conditions met)
```

---

## Summary

| Component | Status | Finding |
|-----------|--------|---------|
| HTF Loading | ✅ Works | Candles loaded correctly |
| MarketStateEngine | ✅ Works | Uses daily_structure correctly |
| detect_structure | ✅ Works | Requires ≥20 candles (by design) |
| **Root Cause** | 🔴 **FOUND** | **No HTF warmup → 0 daily candles** |
| **Fix** | ✅ **CLEAR** | **Add htf_warmup_days parameter** |

---

**Primary Cause:** Date slicing without HTF warmup  
**Impact:** 100% day_type = 'unknown' → News_Fade blocked  
**Fix:** Add 30-day HTF warmup period (minimal patch)  
**Risk:** Low (no logic change, only data loading)  

---

**Delivered:** 2025-12-27  
**Next:** P2-2.B B3 (Patch Implementation)
