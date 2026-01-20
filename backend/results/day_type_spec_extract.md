# Day_Type Specification Extract

## SOURCE

**File:** `/app/backend/knowledge/playbooks.yml`

**Field:** `context_requirements.day_type_allowed`

---

## VALID VALUES (from playbook definitions)

Based on all 11 playbooks in playbooks.yml:

1. **`"trend"`** - Trending day (clear directional movement)
   - Used by: 7/11 playbooks
   - Examples: NY_Open_Reversal, London_Sweep_NY_Continuation, Trend_Continuation_FVG_Retest, etc.

2. **`"manipulation_reversal"`** - Manipulation followed by reversal
   - Used by: 3/11 playbooks
   - Examples: Morning_Trap_Reversal, News_Fade

3. **`"range"`** - Ranging/consolidation day
   - Used by: 4/11 playbooks
   - Examples: FVG_Fill_Scalp, Lunch_Range_Scalp, News_Fade

**Vocabulary:** `["trend", "manipulation_reversal", "range"]`

---

## PLAYBOOK REQUIREMENTS

| Playbook | day_type_allowed |
|----------|-----------------|
| NY_Open_Reversal | ["trend", "manipulation_reversal"] |
| London_Sweep_NY_Continuation | ["trend"] |
| Trend_Continuation_FVG_Retest | ["trend"] |
| Power_Hour_Expansion | ["trend"] |
| Morning_Trap_Reversal | ["manipulation_reversal", "range"] |
| Liquidity_Sweep_Scalp | ["trend"] |
| FVG_Fill_Scalp | ["trend", "range"] |
| BOS_Momentum_Scalp | ["trend"] |
| **News_Fade** | **["manipulation_reversal", "range"]** ← BLOCKER |
| Session_Open_Scalp | ["trend"] |
| Lunch_Range_Scalp | ["range", "trend"] |

---

## IMPLEMENTATION LOGIC (Inferred from ICT Concepts)

### Detection Rules

**1. TREND DAY**
- Clear directional movement (uptrend or downtrend)
- Multiple BOS (Break of Structure) in same direction
- Clean FVGs forming in trend direction
- Low consolidation periods

**Algorithm:**
```python
def detect_trend_day(daily_candles, ict_patterns):
    """
    Conditions:
    - Structure: uptrend OR downtrend (not range)
    - BOS count in primary direction >= 2
    - Range expansion > threshold (e.g., ATR * 1.5)
    """
    structure = detect_structure(daily_candles)
    if structure in ['uptrend', 'downtrend']:
        bos_count = count_bos_in_direction(ict_patterns, structure)
        if bos_count >= 2:
            return 'trend'
    return None
```

---

**2. MANIPULATION_REVERSAL DAY**
- Initial liquidity sweep (fake move)
- Strong reversal back
- Often seen after news events

**Algorithm:**
```python
def detect_manipulation_reversal_day(daily_candles, ict_patterns):
    """
    Conditions:
    - Sweep detected in early session
    - Followed by reversal pattern (CHoCH or strong BOS in opposite direction)
    - High/low of day taken, then reversed
    """
    sweeps = [p for p in ict_patterns if p.pattern_type == 'sweep']
    reversals = [p for p in ict_patterns if p.pattern_type == 'bos' and p.reversal]
    
    if len(sweeps) >= 1 and len(reversals) >= 1:
        # Check if reversal happened after sweep
        if reversals[0].timestamp > sweeps[0].timestamp:
            return 'manipulation_reversal'
    return None
```

---

**3. RANGE DAY**
- Consolidation/choppy
- No clear directional bias
- Multiple failed breakouts

**Algorithm:**
```python
def detect_range_day(daily_candles):
    """
    Conditions:
    - Structure: range
    - Low ATR relative to recent average
    - High/low within tight range
    """
    structure = detect_structure(daily_candles)
    if structure == 'range':
        return 'range'
    return None
```

---

## FALLBACK

If unable to determine confidently: **`day_type = "unknown"`**

Playbooks with `day_type_allowed` will reject if market_state.day_type not in their list.

---

## PRIORITY

**P1 Target:** Implement day_type calculation in MarketStateEngine

**Impact:** Unblock News_Fade playbook (+2 matches on test sample)

**Complexity:** MEDIUM (requires pattern analysis logic)

**Reference Implementation:** Lines 225-280 in `/app/backend/engines/market_state.py` (HTF structure analysis)

---

## MINIMAL IMPLEMENTATION FOR P1

For P1, use SIMPLIFIED logic:

```python
def calculate_day_type(daily_structure: str, ict_patterns: List[ICTPattern]) -> str:
    """
    Simplified day_type calculation for P1
    """
    # Count BOS patterns
    bos_count = len([p for p in ict_patterns if p.pattern_type == 'bos'])
    
    # Count sweep patterns
    sweep_count = len([p for p in ict_patterns if p.pattern_type == 'sweep'])
    
    # Rule 1: If structure is range → range day
    if daily_structure == 'range':
        return 'range'
    
    # Rule 2: If sweep detected + structure reversal → manipulation_reversal
    if sweep_count >= 1 and bos_count >= 1:
        return 'manipulation_reversal'
    
    # Rule 3: If structure is uptrend/downtrend + BOS >= 2 → trend
    if daily_structure in ['uptrend', 'downtrend'] and bos_count >= 2:
        return 'trend'
    
    # Rule 4: Default to trend if structure is clear
    if daily_structure in ['uptrend', 'downtrend']:
        return 'trend'
    
    # Fallback
    return 'unknown'
```

**Note:** This is SIMPLIFIED. Full implementation would analyze:
- Intraday price action
- Session profiles
- Volatility patterns
- News calendar events

But for P1, this covers ~80% of cases and unblocks News_Fade.
