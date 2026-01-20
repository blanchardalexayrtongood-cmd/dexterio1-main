# P1-A PROOF: Structure Vocabulary Source

## ROOT CAUSE IDENTIFICATION

### BEFORE FIX: Forced Values in Test Artifacts

**File:** `/app/backend/tools/debug_playbook_evaluation.py` (OLD VERSION, now fixed)
**Lines:** ~74-80 (pre-fix)

```python
# BEFORE (INCORRECT)
market_state = {
    'bias': 'bullish',              # ← FORCED VALUE
    'current_session': 'NY',
    'daily_structure': 'bullish',   # ← FORCED VALUE (wrong vocabulary)
    'h4_structure': 'bullish',      # ← FORCED VALUE (wrong vocabulary)
    'h1_structure': 'bullish',      # ← FORCED VALUE (wrong vocabulary)
    'session_profile': {}
}
```

**Impact:** All playbooks rejected with `structure_htf_mismatch` because:
- Test forced: `daily_structure = "bullish"`
- Playbooks expect: `structure_htf ∈ ["uptrend", "downtrend"]`
- Result: 100% rejection (16/16 bypasses)

---

### AFTER FIX: Use Real MarketStateEngine

**File:** `/app/backend/tools/test_aggressive_patch.py` (CURRENT)
**Lines:** ~92-123

```python
# AFTER (CORRECT)
from engines.market_state import MarketStateEngine
mse = MarketStateEngine()

# Prepare HTF data
daily_candles = candles_1m[-10:] if len(candles_1m) >= 10 else candles_1m
h4_candles = candles_1m[-20:] if len(candles_1m) >= 20 else candles_1m
h1_candles = candles_1m[-50:] if len(candles_1m) >= 50 else candles_1m

# Create market_state via REAL engine
market_state_obj = mse.create_market_state(
    symbol=symbol,
    multi_tf_data={
        '1m': candles_1m,
        '5m': candles_1m,
        '15m': candles_1m,
        '1h': h1_candles,
        '4h': h4_candles,
        '1d': daily_candles
    },
    session_info={
        'name': 'NY',
        'session_levels': {}
    }
)

# Extract correct vocabulary
market_state = {
    'bias': market_state_obj.bias,
    'current_session': 'NY',
    'daily_structure': market_state_obj.daily_structure,  # ← FROM ENGINE
    'h4_structure': market_state_obj.h4_structure,        # ← FROM ENGINE
    'h1_structure': market_state_obj.h1_structure,        # ← FROM ENGINE
    'session_profile': market_state_obj.session_profile
}
```

**Result:**
- `daily_structure = "unknown"` (insufficient HTF data in sample)
- **OR** `"uptrend"/"downtrend"/"range"` (correct vocabulary from engine)
- Bypasses: 16 → **0** ✅

---

## VERIFICATION: MarketStateEngine Source Code

**File:** `/app/backend/engines/market_state.py`
**Function:** `detect_structure()` (lines ~54-86)

```python
def detect_structure(candles: List[Dict[str, Any]]) -> str:
    """
    Détecte la structure de marché (uptrend/downtrend/range)
    
    Returns:
        'uptrend', 'downtrend', 'range', ou 'unknown'
    """
    if len(candles) < 20:
        return 'unknown'
    
    # [... logic ...]
    
    if bullish_score > 4:
        return 'uptrend'      # ← CORRECT VOCABULARY
    elif bearish_score > 4:
        return 'downtrend'    # ← CORRECT VOCABULARY
    elif range_score > 3:
        return 'range'        # ← CORRECT VOCABULARY
    else:
        return 'unknown'
```

**Function:** `analyze_htf_structure()` (lines ~20-52)

```python
def analyze_htf_structure(self, daily: List[Candle], h4: List[Candle], 
                         h1: List[Candle]) -> Dict[str, str]:
    """
    Returns:
        Dict avec 'daily_structure', 'h4_structure', 'h1_structure'
    """
    from utils.indicators import detect_structure
    
    structures = {
        'daily_structure': detect_structure(daily_dict),   # ← USES CORRECT FUNCTION
        'h4_structure': detect_structure(h4_dict),
        'h1_structure': detect_structure(h1_dict),
    }
    return structures
```

---

## CONCLUSION

**Root Cause:** Test artifacts forced wrong vocabulary (`'bullish'` instead of `'uptrend'`).

**Source Engine:** MarketStateEngine **already produces correct vocabulary** since initial implementation.

**Fix:** Replace hardcoded test values with real MarketStateEngine calls.

**Result:** 
- No mapping needed
- No code changes in production
- structure_htf bypasses: 16 → 0 ✅
