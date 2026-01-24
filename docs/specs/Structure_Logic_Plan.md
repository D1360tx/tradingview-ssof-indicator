# SSOF Structure Logic Overhaul — Implementation Plan

**Date:** 2026-01-08  
**Version:** 1.0  
**Status:** ✅ Phase 1-3 IMPLEMENTED - Phase 4 (Validation) In Progress

---

## Problem Statement

Current BOS and structure logic occasionally promotes internal breaks to full BOS, fails to maintain/update the active impulse container, and flips bias outside of protected-level invalidation. This spec formalizes correct SMC behavior and defines code changes.

## Scope

Covers BOS detection, impulse container maintenance, internal-structure filtering, and state machine (BULL/BEAR/NEUTRAL). No visualization or zone logic changes, except where required to reflect state.

---

## Core Definitions (Authoritative)

### Structural Swings
A swing high/low is a structural pivot (ta.pivothigh/pivotlow with configured lengths). Only these are eligible for BOS.

### BOS (strict)
Bullish BOS: candle close > prior structural swing high. Bearish BOS: candle close < prior structural swing low. Wicks alone never confirm.

### Impulse Range (container)
After a BOS confirms:
- Bullish: protectedLow = higher low leading to BOS; impulseLow = protectedLow; impulseHigh = highest high printed since BOS (must update forward as expansion continues).
- Bearish: protectedHigh = lower high leading to BOS; impulseHigh = protectedHigh; impulseLow = lowest low printed since BOS (must update forward as expansion continues).

### Protected Level (single per bias)
Bullish bias uses protectedLow as invalidation; bearish bias uses protectedHigh. Protected levels are invalidation, not reversal targets.

---

## Current Implementation (Summary)

- Swings: correct.
- BOS condition: uses bodyClosesAbove/Below — correct.
- Impulse container: initialized only at BOS using current bar's high/low, not updated thereafter — incorrect.
- Internal filter: checks only relative to current structure and with asymmetric conditions; can miss internal breaks — incomplete.
- State changes: structure flips on any non-internal BOS; lacks explicit NEUTRAL use during internal-only action — incomplete.

---

## Issues To Fix (Top 3)

### 1. Impulse container not updating dynamically
Causing stale bounds and misclassification.

### 2. Internal-structure filter incomplete
Internal breaks inside the active container can be promoted to BOS.

### 3. State machine permits flips outside protected-level invalidation
Doesn't leverage NEUTRAL when movement is internal-only.

---

## Target Behavior (Spec)

### Single-BOS-Per-Leg Rule
Once a BOS confirms and an impulse container is set, subsequent swing breaks entirely inside [impulseLow, impulseHigh] are internal; do not change trend, protected levels, or emit BOS (label as internal iBOS± if desired).

### Internal Structure Test (exact)
Given a candidate break at price P (the close that broke the swing):
- Internal if impulseLow < P < impulseHigh.
- Otherwise external (eligible to be a BOS in the opposite direction) and will either extend the container (same-bias) or invalidate via protected level (opposite-bias).

### Dynamic Impulse Updates
On every bar after BOS while bias is active:
- Bullish: impulseHigh = max(impulseHigh, high).
- Bearish: impulseLow = min(impulseLow, low).

### State Machine (BULL/BEAR/NEUTRAL)
- States: 1=BULL, -1=BEAR, 0=NEUTRAL.
- Transitions:
  - 1 → -1 only when close < protectedLow.
  - -1 → 1 only when close > protectedHigh.
- While inside the active container AND without protected-level breach, trend remains as-is; optionally set NEUTRAL when repeated internals dominate and there is no extension (configurable). Minimum viable: keep existing bias until invalidation; do not flip on internal breaks.

---

## Algorithm (Pseudocode)

### Inputs / State
- structureState ∈ {1,0,-1}
- confirmedSwingHigh, confirmedSwingLow, prevSwingHigh, prevSwingLow
- protectedLow, protectedHigh, impulseLow, impulseHigh
- useInternalFilter (bool)

#### Detection helpers
bodyClosesAbove(level) and bodyClosesBelow(level)

### Step 1: Raw break evaluation
```
rawBull = !na(confirmedSwingHigh) and bodyClosesAbove(confirmedSwingHigh)
rawBear = !na(confirmedSwingLow) and bodyClosesBelow(confirmedSwingLow)
```

### Step 2: Internal filter
```
isInternal = false
if useInternalFilter and !na(impulseLow) and !na(impulseHigh)
  breakPrice = close
  if breakPrice > impulseLow and breakPrice < impulseHigh
    isInternal := true
internalBull = rawBull and isInternal
internalBear = rawBear and isInternal
```

### Step 3: Valid BOS
```
bullishBOS = rawBull and not isInternal
bearishBOS = rawBear and not isInternal
```

### Step 4: Protected-level invalidation (authoritative flips)
```
protectedBroken = false
if structureState == 1 and !na(protectedLow) and close < protectedLow
  protectedBroken := true
  structureState := -1
  // initialize bearish container
  protectedHigh := highest lookback lower-high (see Step 6);
  impulseHigh := protectedHigh
  impulseLow := low
if structureState == -1 and !na(protectedHigh) and close > protectedHigh
  protectedBroken := true
  structureState := 1
  // initialize bullish container
  protectedLow := lowest lookback higher-low
  impulseLow := protectedLow
  impulseHigh := high
```

### Step 5: Same-bias BOS extends container, not flip
```
if bullishBOS
  structureState := 1
  // set/confirm protectedLow (origin) and initialize/extend container
  if na(protectedLow) then protectedLow := lowest lookback low
  if na(impulseLow) then impulseLow := protectedLow
  impulseHigh := max(impulseHigh, high)
if bearishBOS
  structureState := -1
  if na(protectedHigh) then protectedHigh := highest lookback high
  if na(impulseHigh) then impulseHigh := protectedHigh
  impulseLow := min(impulseLow, low)
```

### Step 6: Determining protected origin (lookback)
Use lookbackBars = swingLength * protectedLookback. For bulls: lowest(low, lookbackBars); for bears: highest(high, lookbackBars). Persist until invalidation.

### Step 7: Continuous updates
```
if structureState == 1 and !na(impulseHigh)
  impulseHigh := max(impulseHigh, high)
if structureState == -1 and !na(impulseLow)
  impulseLow := min(impulseLow, low)
```

---

## UI/Labels

- Show Protected High only in BEAR; Protected Low only in BULL.
- BOS labels: only when not internal. Internal breaks may show iBOS± (optional) and trap warnings.
- Dashboard 'Prot' displays protectedLow (BULL) or protectedHigh (BEAR). Consider optional NEUTRAL display when configured.

---

## Validation Checklist

1. Green "bullish BOS" cannot print below current impulseHigh when structure = BEAR.
2. Red "bearish BOS" cannot print above current impulseLow when structure = BULL.
3. Impulse extremes extend on each new extreme while bias is active.
4. Bias flips only on protected-level breach; never on internal breaks.
5. Regression on example charts (Gold 4H):
   - Active bullish impulse ~4174→~4550: no bearish BOS until close < ~4174.
   - Any break like ~4311 labeled internal (iBOS↑) or suppressed.

---

## Migration Plan (High Level)

### Phase 1: Dynamic Container Updates
1. Add persistent dynamic updates for impulseHigh/impulseLow.
2. Test on multiple timeframes.

### Phase 2: Internal Filter Rewrite
1. Replace structure-conditioned internal checks with container-based symmetric test.
2. Ensure both bull and bear breaks are evaluated against same impulse bounds.

### Phase 3: State Machine Hardening
1. Gate state flips by protected-level breach only; decide on optional NEUTRAL.
2. Update dashboard to reflect changes.

### Phase 4: Validation & Documentation
1. Test on example charts (Gold, Silver, BTC).
2. Update CHANGELOG with breaking changes.
3. Increment version to V7.0 (major rewrite).

---

## Risks & Notes

- Lookback origin selection is sensitive; retain current protectedLookback multiplier but ensure it's evaluated consistently each bar when bias changes.
- Historical buffer constraints: use safeBarsFromOrigin bounds when computing extremes.
- Breaking change: Existing users may see different BOS labels; document in CHANGELOG.
- Performance: Dynamic updates add minimal overhead (simple max/min operations per bar).

---

## Appendix: Minimal Code Diff Pointers (not applied yet)

- Where: Market Structure Logic block (approx lines 313–420 and 1111–1142 for updates).
- Add: continuous container update after BOS and on each bar.
- Replace: internal filter with container-based symmetric check.
- Keep: body-close BOS conditions and protected-level visuals/alerts.

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
**Created By:** Warp Agent + User Research
