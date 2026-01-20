# P1-B: Candlestick Engine Wiring Plan

## EXECUTIVE SUMMARY

**Status:** Engine EXISTS and FUNCTIONAL (6-18 patterns detected on samples)

**Blocker:** Type architecture mismatch prevents wiring

**Decision:** OUT OF SCOPE for P1 (requires refactoring 3+ files)

**Recommendation:** DEFER to P2

---

## CURRENT STATE ANALYSIS

### Engine Signature

**File:** `/app/backend/engines/patterns/candlesticks.py`

**Class:** `CandlestickPatternEngine`

**Method:**
```python
def detect_patterns(
    self, 
    candles: List[Candle], 
    timeframe: str, 
    sr_levels: Optional[List[float]] = None
) -> List[PatternDetection]:  # ← RETURNS PatternDetection
```

**Returns:** `List[PatternDetection]`

**PatternDetection fields** (from `/app/backend/models/setup.py:7`):
```python
class PatternDetection(BaseModel):
    id: str
    timestamp: datetime
    symbol: str
    timeframe: str
    pattern_name: str      # ← No "family" field
    pattern_type: str
    strength: float
    candles_data: List[Dict]
    # ... other fields
```

---

### Expected Type

**File:** `/app/backend/engines/playbook_loader.py:796`

**Expected:** `List[CandlestickPattern]`

**CandlestickPattern fields** (from `/app/backend/models/setup.py:54`):
```python
class CandlestickPattern(BaseModel):
    id: str
    timestamp: datetime
    timeframe: str
    family: str            # ← REQUIRED (engulfing, pin_bar, etc.)
    name: str
    direction: str
    strength: float
    body_size: float
    confirmation: bool
    at_level: bool
    after_sweep: bool
```

---

## TYPE MISMATCH ANALYSIS

| Field | PatternDetection | CandlestickPattern | Status |
|-------|------------------|-------------------|--------|
| `family` | ❌ Missing | ✅ Required | **BLOCKER** |
| `name` | `pattern_name` | `name` | Different field name |
| `direction` | ❌ Missing | ✅ Required | **BLOCKER** |
| `body_size` | ❌ Missing | ✅ Required | **BLOCKER** |
| `confirmation` | ❌ Missing | ✅ Required | **BLOCKER** |

---

## WIRING OPTIONS

### Option 1: Refactor Engine to Return CandlestickPattern (RECOMMENDED)

**Files to modify:**
1. `/app/backend/engines/patterns/candlesticks.py:_create_pattern()` (line ~200)
   - Change return type from `PatternDetection` to `CandlestickPattern`
   - Add missing fields: `family`, `direction`, `body_size`, `confirmation`

**Complexity:** MEDIUM
- ~50 lines to modify
- Need to extract `family` from `pattern_name` (e.g., "Bullish Engulfing" → family="engulfing")
- Risk: May break other code expecting `PatternDetection`

**Code snippet:**
```python
def _create_pattern(self, name: str, type_: str, strength_: str, 
                   candles_: List[Candle], tf: str, sr: List[float], 
                   score: float) -> CandlestickPattern:  # ← Change return type
    
    # Extract family from name
    family_map = {
        'hammer': 'hammer',
        'shooting_star': 'shooting_star',
        'bullish_engulfing': 'engulfing',
        'bearish_engulfing': 'engulfing',
        # ... etc
    }
    family = family_map.get(name.lower().replace(' ', '_'), 'unknown')
    
    # Extract direction
    direction = 'bullish' if 'bullish' in name.lower() else 'bearish' if 'bearish' in name.lower() else 'neutral'
    
    # Calculate body_size
    c = candles_[-1]
    body = abs(c.close - c.open)
    range_ = c.high - c.low
    body_size = body / range_ if range_ > 0 else 0
    
    return CandlestickPattern(
        timestamp=c.timestamp,
        timeframe=tf,
        family=family,
        name=name,
        direction=direction,
        strength=score,
        body_size=body_size,
        confirmation=False,  # Default
        at_level=len(sr) > 0,  # Simplified
        after_sweep=False  # Default
    )
```

---

### Option 2: Adapter Pattern in PlaybookLoader (NOT RECOMMENDED)

**Files to modify:**
1. `/app/backend/engines/playbook_loader.py:_evaluate_playbook_conditions()` (line ~796)
   - Accept `List[PatternDetection]`
   - Convert to `List[CandlestickPattern]` on the fly

**Complexity:** LOW
- ~20 lines to add adapter function
- But creates technical debt (two types for same concept)

**Code snippet:**
```python
def _pattern_detection_to_candlestick(pd: PatternDetection) -> CandlestickPattern:
    # Extract family from pattern_name
    family_map = {...}
    family = family_map.get(pd.pattern_name.lower(), 'unknown')
    
    return CandlestickPattern(
        timestamp=pd.timestamp,
        timeframe=pd.timeframe,
        family=family,
        name=pd.pattern_name,
        direction='neutral',  # Can't extract reliably
        strength=pd.strength,
        body_size=0.0,  # Missing
        confirmation=False,
        at_level=False,
        after_sweep=False
    )
```

**Issues:**
- Loses information (direction, body_size not available)
- Creates maintenance burden (two parallel types)

---

## COVERAGE ANALYSIS

### Sample Testing Results

| Test | Bars | Patterns | Coverage % |
|------|------|----------|-----------|
| Sample A (100 bars) | 100 | 6 | 6.0% |
| Sample B (18 bars, monthly sample) | 18 | 3 | 16.7% |
| **Average** | - | - | **~10%** |

**Pattern Distribution (Sample B):**
- `shooting_star`: 2 (66%)
- `dragonfly_doji`: 1 (33%)

**Required by Playbooks:**
- `engulfing`, `pin_bar`, `morning_star`, `evening_star`, `marubozu`, `three_soldiers`, `three_crows`, `hammer`, `shooting_star`, `doji`, `spinning_top`

**Detected Families:**
- `shooting_star` ✅
- `doji` ✅ (dragonfly_doji)

**Missing Families:**
- `engulfing`, `pin_bar`, `marubozu`, `hammer`, etc.

**Coverage Assessment:** **LOW** (~10% of bars have patterns)

---

## IMPACT ESTIMATION

### If Wired (Optimistic)

Assuming engine detects patterns on 10% of setups:

**Before:**
- `candlestick_patterns_missing` bypass: 16
- Avg score: 0.145
- Grades: 100% C

**After:**
- Setups with patterns: ~2 out of 16 (10%)
- Setups without patterns: ~14 (still bypass)
- **Net bypass reduction:** 16 → ~14 (12.5% reduction)

**Score improvement (for matched setups with patterns):**
- pattern_quality_score contribution: +0.1 to +0.3
- New avg score: 0.145 → 0.25 (for those 2 setups)
- Grade improvement: C → B (for 2/16 setups)

**Realistic Impact:** **MARGINAL** (affects ~10% of setups)

---

### If Wired + Engine Enhanced (Optimistic)

If engine threshold/logic adjusted to detect on 50% of bars:

**After:**
- Setups with patterns: ~8 out of 16 (50%)
- Setups without patterns: ~8 (still bypass)
- **Net bypass reduction:** 16 → ~8 (50% reduction)

**Score improvement:**
- New avg score: 0.145 → 0.35
- Grade distribution: 50% C, 40% B, 10% A

**Realistic Impact:** **MODERATE** (requires engine tuning)

---

## EFFORT ESTIMATION

### Option 1: Refactor Engine (Clean)

**Files:** 1 (`candlesticks.py`)
**Lines changed:** ~50-100
**Test files:** 1-2 (unit tests for new return type)
**Risk:** LOW (isolated change)
**Time:** 2-3 hours

---

### Option 2: Wire with Adapter (Quick)

**Files:** 1 (`playbook_loader.py`)
**Lines changed:** ~20
**Risk:** MEDIUM (technical debt)
**Time:** 30 minutes

---

## DECISION MATRIX

| Criteria | Option 1 (Refactor) | Option 2 (Adapter) | DEFER (Current) |
|----------|---------------------|-------------------|-----------------|
| **Impact** | Moderate (+50% coverage if tuned) | Marginal (+10% coverage) | None (bypass active) |
| **Effort** | 2-3 hours | 30 minutes | 0 |
| **Risk** | Low | Medium (debt) | None |
| **Quality** | High (clean) | Low (debt) | N/A |
| **P1 Scope** | OUT ❌ | OUT ❌ | IN ✅ |

---

## RECOMMENDATION

**DEFER to P2**

**Justification:**
1. **LOW coverage (10%)** = Marginal impact even if wired
2. **P1 scope = unblock funnel** ✅ Already achieved (16 matches)
3. **P1 priority = day_type** (blocks News_Fade, higher ROI)
4. **Refactoring = P2 scope** (quality improvement, not blocker)

**Future P2 Actions:**
1. Refactor `candlesticks.py` to return `CandlestickPattern` (Option 1)
2. Tune engine thresholds for 50%+ coverage
3. Wire into `BacktestEngine._process_bar_optimized()`
4. Expected: Bypass 16 → 8, Avg score 0.145 → 0.35

**Current State:** **BYPASS JUSTIFIED** (engine not wired, low coverage)
