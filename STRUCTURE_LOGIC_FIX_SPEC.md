# SSOF Structure Logic Overhaul — BOS/Impulse/Internal Filter Spec v1

**Date:** 2026-01-08  
**Status:** Analysis Complete - Implementation Pending  
**Severity:** Critical - Affects core structure detection

---

## Problem Statement

Current BOS and structure logic occasionally promotes internal breaks to full BOS, fails to maintain/update the active impulse container, and flips bias outside of protected-level invalidation. This spec formalizes correct SMC behavior and defines code changes.

### Scope
Covers BOS detection, impulse container maintenance, internal-structure filtering, and state machine (BULL/BEAR/NEUTRAL). No visualization or zone logic changes, except where required to reflect state.

---

## Core Definitions (Authoritative)

### Structural Swings
A swing high/low is a structural pivot (`ta.pivothigh`/`pivotlow` with configured lengths). Only these are eligible for BOS.

### BOS (strict)
- **Bullish BOS:** candle close > prior structural swing high
- **Bearish BOS:** candle close < prior structural swing low
- **Wicks alone never confirm**

### Impulse Range (container)
After a BOS confirms:

**Bullish:**
- `protectedLow` = higher low leading to BOS
- `impulseLow` = protectedLow
- `impulseHigh` = highest high printed since BOS (**must update forward as expansion continues**)

**Bearish:**
- `protectedHigh` = lower high leading to BOS
- `impulseHigh` = protectedHigh
- `impulseLow` = lowest low printed since BOS (**must update forward as expansion continues**)

### Protected Level (single per bias)
- Bullish bias uses `protectedLow` as invalidation
- Bearish bias uses `protectedHigh` as invalidation
- **Protected levels are invalidation, not reversal targets**

---

## Current Implementation (Summary)

✅ **Swings:** Correct - using `ta.pivothigh/pivotlow`  
✅ **BOS condition:** Correct - uses `bodyClosesAbove/Below`  
❌ **Impulse container:** Initialized only at BOS using current bar's high/low, not updated thereafter  
❌ **Internal filter:** Checks only relative to current structure with asymmetric conditions; can miss internal breaks  
❌ **State changes:** Structure flips on any non-internal BOS; lacks explicit NEUTRAL use during internal-only action  

---

## Issues To Fix (Top 3)

### Issue #1: Impulse Container Not Updating Dynamically
**Problem:** Impulse range is set ONLY at BOS moment (lines 383-385, 417-419) and never updated as price expands.

**Current Code:**
```pine
if bullishBOS
    impulseLow := protectedLow
    impulseHigh := high  // ❌ Only uses current bar's high
```

**Impact:** Container becomes stale immediately, causing internal breaks to be misclassified as valid BOS.

**Required Fix:**
```pine
// After every bar while in bullish structure
if structureState == 1 and not na(impulseHigh)
    impulseHigh := math.max(impulseHigh, high)
```

---

### Issue #2: Internal Structure Filter Incomplete
**Problem:** Filter only checks in the opposite structure state and uses asymmetric conditions (lines 323-331).

**Current Code:**
```pine
if rawBullishBOS
    // Only checks in BEARISH structure
    if structureState == -1 and close < impulseHigh
        isInsideImpulseRange := true
```

**Impact:** In bullish structure, bearish breaks aren't properly checked if they're internal. Causes premature trend flips.

**Required Fix:**
```pine
// Symmetric container-based test
if rawBullishBOS or rawBearishBOS
    if not na(impulseLow) and not na(impulseHigh)
        if close > impulseLow and close < impulseHigh
            isInsideImpulseRange := true
```

---

### Issue #3: Structure Flips Outside Protected Level Invalidation
**Problem:** Structure state changes on any non-internal BOS, not just protected-level breaks.

**Current Code:**
```pine
if bullishBOS
    structureState := 1  // ❌ Flips immediately
```

**Impact:** Structure flips during internal retracements instead of waiting for protected-level breach.

**Required Fix:**
```pine
// Only flip on protected level breach
if structureState == 1 and close < protectedLow
    structureState := -1
    // Then initialize new bearish container
```

---

## Target Behavior (Spec)

### Single-BOS-Per-Leg Rule
Once a BOS confirms and an impulse container is set, subsequent swing breaks entirely inside `[impulseLow, impulseHigh]` are **internal**:
- Do NOT change trend
- Do NOT update protected levels
- Do NOT emit BOS labels
- Optionally label as internal `iBOS↑/↓` with trap warnings

### Internal Structure Test (exact)
Given a candidate break at price `P` (the close that broke the swing):

**Internal if:**
```
impulseLow < P < impulseHigh
```

**Otherwise external** (eligible to be a BOS in opposite direction) and will either:
- Extend the container (same-bias continuation)
- Invalidate via protected level (opposite-bias reversal)

### Dynamic Impulse Updates
On every bar after BOS while bias is active:
- **Bullish:** `impulseHigh = max(impulseHigh, high)`
- **Bearish:** `impulseLow = min(impulseLow, low)`

### State Machine (BULL/BEAR/NEUTRAL)
**States:** `1=BULL`, `-1=BEAR`, `0=NEUTRAL`

**Transitions:**
- `1 → -1` only when `close < protectedLow`
- `-1 → 1` only when `close > protectedHigh`

While inside the active container AND without protected-level breach:
- Trend remains as-is
- Optionally set NEUTRAL when repeated internals dominate (configurable)
- **Minimum viable:** Keep existing bias until invalidation; do NOT flip on internal breaks

---

## Algorithm (Pseudocode)

### Inputs / State
```pine
var int structureState ∈ {1, 0, -1}
var float confirmedSwingHigh, confirmedSwingLow
var float prevSwingHigh, prevSwingLow
var float protectedLow, protectedHigh
var float impulseLow, impulseHigh
bool useInternalFilter
```

### Step 1: Raw Break Evaluation
```pine
rawBull = not na(confirmedSwingHigh) and bodyClosesAbove(confirmedSwingHigh)
rawBear = not na(confirmedSwingLow) and bodyClosesBelow(confirmedSwingLow)
```

### Step 2: Internal Filter
```pine
isInternal = false
if useInternalFilter and not na(impulseLow) and not na(impulseHigh)
    breakPrice = close
    if breakPrice > impulseLow and breakPrice < impulseHigh
        isInternal := true

internalBull = rawBull and isInternal
internalBear = rawBear and isInternal
```

### Step 3: Valid BOS
```pine
bullishBOS = rawBull and not isInternal
bearishBOS = rawBear and not isInternal
```

### Step 4: Protected-Level Invalidation (Authoritative Flips)
```pine
protectedBroken = false

if structureState == 1 and not na(protectedLow) and close < protectedLow
    protectedBroken := true
    structureState := -1
    // Initialize bearish container
    protectedHigh := highest lookback lower-high
    impulseHigh := protectedHigh
    impulseLow := low

if structureState == -1 and not na(protectedHigh) and close > protectedHigh
    protectedBroken := true
    structureState := 1
    // Initialize bullish container
    protectedLow := lowest lookback higher-low
    impulseLow := protectedLow
    impulseHigh := high
```

### Step 5: Same-Bias BOS Extends Container (Not Flip)
```pine
if bullishBOS
    structureState := 1
    // Set/confirm protectedLow (origin) and initialize/extend container
    if na(protectedLow)
        protectedLow := lowest lookback low
    if na(impulseLow)
        impulseLow := protectedLow
    impulseHigh := math.max(impulseHigh, high)

if bearishBOS
    structureState := -1
    if na(protectedHigh)
        protectedHigh := highest lookback high
    if na(impulseHigh)
        impulseHigh := protectedHigh
    impulseLow := math.min(impulseLow, low)
```

### Step 6: Determining Protected Origin (Lookback)
```pine
lookbackBars = swingLength * protectedLookback

// For bullish: lowest(low, lookbackBars)
// For bearish: highest(high, lookbackBars)
// Persist until invalidation
```

### Step 7: Continuous Updates (Critical!)
```pine
// On EVERY bar while bias is active
if structureState == 1 and not na(impulseHigh)
    impulseHigh := math.max(impulseHigh, high)

if structureState == -1 and not na(impulseLow)
    impulseLow := math.min(impulseLow, low)
```

---

## UI/Labels

- Show **Protected High** only in BEAR structure
- Show **Protected Low** only in BULL structure
- **BOS labels:** Only when NOT internal
- **Internal breaks:** May show `iBOS↑/↓` (optional) and trap warnings `⚠️ TRAP?`
- **Dashboard 'Prot':** Displays `protectedLow` (BULL) or `protectedHigh` (BEAR)
- Consider optional NEUTRAL display when configured

---

## Validation Checklist

1. ✅ Green "bullish BOS" cannot print below current `impulseHigh` when structure = BEAR
2. ✅ Red "bearish BOS" cannot print above current `impulseLow` when structure = BULL
3. ✅ Impulse extremes extend on each new extreme while bias is active
4. ✅ Bias flips only on protected-level breach; never on internal breaks
5. ✅ **Regression on example charts (Gold 4H):**
   - Active bullish impulse ~4174→~4550: no bearish BOS until close < ~4174
   - Any break like ~4311 labeled internal (`iBOS↑`) or suppressed

---

## Real-World Test Case (Gold 4H - User Provided)

**Setup:**
- Prior bullish BOS at 4133, 4259, 4353
- Impulse range: ~4174 (low) → ~4550 (high)
- Structure: BULLISH

**Observed:**
- Green "BOS ↑" label appeared at ~4311 (INCORRECT)
- This is inside the impulse range (4174-4550)

**Expected:**
- No BOS label at 4311
- Should show `iBOS↑` or trap warning
- Structure remains BULL until close < 4174
- Only a close > 4550 (protected high from prior bear) OR < 4174 (protected low) flips structure

---

## Migration Plan (High Level)

### Phase 1: Dynamic Container Updates
1. Add continuous `impulseHigh`/`impulseLow` updates on every bar
2. Test on multiple timeframes

### Phase 2: Internal Filter Rewrite
1. Replace structure-conditioned checks with symmetric container-based test
2. Ensure both bull and bear breaks are evaluated against same impulse bounds

### Phase 3: State Machine Hardening
1. Gate state flips by protected-level breach only
2. Decide on optional NEUTRAL state usage
3. Update dashboard to reflect changes

### Phase 4: Validation & Documentation
1. Test on example charts (Gold, Silver, BTC)
2. Update CHANGELOG with breaking changes
3. Increment version to V7.0 (major rewrite)

---

## Risks & Notes

- **Lookback origin selection** is sensitive; retain current `protectedLookback` multiplier but ensure it's evaluated consistently each bar when bias changes
- **Historical buffer constraints:** Use `safeBarsFromOrigin` bounds when computing extremes
- **Breaking change:** Existing users may see different BOS labels; document in CHANGELOG
- **Performance:** Dynamic updates add minimal overhead (simple max/min operations per bar)

---

## Appendix: Code Location Pointers

### Files to Modify
- `ssof.pine` (main indicator file)

### Approximate Line Numbers
- **Market Structure Logic:** Lines 313-420
- **Impulse Updates:** Need to add after line 455 (bars since impulse tracking)
- **Dynamic Container:** Need continuous updates in main bar loop
- **Internal Filter:** Lines 323-331 (complete rewrite)
- **BOS Processing:** Lines 372-438 (modify state update logic)

### Key Variables
```pine
var int structureState          // Line 285
var float protectedLow/High     // Lines 289-292
var float impulseLow/High       // Lines 295-296
bool isInsideImpulseRange       // Line 322
bool bullishBOS/bearishBOS      // Lines 348-349
```

---

## Research References

### Core SMC Principles Applied
1. **A1-A4:** Structural swings, BOS definition, impulse range, protected levels
2. **B1-B3:** BOS detection rules, single-BOS-per-leg
3. **C1-C2:** Internal structure filter logic
4. **D1-D3:** State machine transitions
5. **E1-E5, F, G:** Gold 4H example analysis and rule verification

### User Research Document
See conversation log 2026-01-08 for full research breakdown including:
- Protected level theory vs practice
- Dashboard flip analysis
- Multi-timeframe validation requirements

---

## Next Steps

1. **Review & Approve:** User validates spec before implementation
2. **Create Feature Branch:** `git checkout -b feature/structure-logic-v7`
3. **Implement Changes:** Follow algorithm pseudocode section
4. **Test Thoroughly:** Validate against checklist and real charts
5. **Update Documentation:** CHANGELOG, WARP.md, CLAUDE_CONTEXT.md
6. **Version Bump:** V6.2.3 → V7.0.0 (major breaking change)

---

**Document Version:** 1.0  
**Last Updated:** 2026-01-08  
**Author:** Warp Agent + User Research
