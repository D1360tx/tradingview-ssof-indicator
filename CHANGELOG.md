# Changelog

All notable changes to the SSOF indicator will be documented in this file.

## [7.0.0-alpha] - 2026-01-08

### 🚨 MAJOR BREAKING CHANGE - Structure Logic Overhaul

**This version fundamentally changes how BOS and structure states are determined. Users may see significantly different BOS labels compared to V6.x.**

#### The 3 Critical Fixes

**1. Dynamic Impulse Container Updates**
- **Before:** Impulse range was set only at BOS moment and never updated
- **After:** Impulse container now extends dynamically on every bar
  - Bullish: `impulseHigh = max(impulseHigh, high)` on every bar
  - Bearish: `impulseLow = min(impulseLow, low)` on every bar
- **Impact:** Internal filter now has accurate bounds for break classification

**2. Symmetric Internal Structure Filter**
- **Before:** Internal check was asymmetric (only checked in opposite structure state)
- **After:** Simple symmetric test: `impulseLow < close < impulseHigh = internal`
- **Impact:** Both bullish AND bearish breaks properly evaluated against same impulse bounds

**3. Protected-Level-Only State Flips**
- **Before:** Structure could flip on any non-internal BOS
- **After:** Reversals (opposite-bias breaks) require protected level breach
  - `BULL → BEAR` only when `close < protectedLow`
  - `BEAR → BULL` only when `close > protectedHigh`
  - Same-bias continuations can extend container without protected level breach
- **Impact:** No more premature trend flips during retracements

#### Added

**Show Internal Structure Zones (Optional)**
- New toggle: "Show Internal Structure Zones?" in Internal Structure Filter group (defaults OFF)
- When enabled, creates supply/demand zones at internal breaks for reference
- Visual distinction from true BOS zones:
  - Dimmer background (95% transparency vs 85%)
  - Dotted border instead of solid
  - Different labels: "iDEMAND" and "iSUPPLY"
  - Lighter text color
- **Why:** V7.0 filters internal breaks correctly, so fewer zones appear by default. This toggle lets you see those internal levels if needed for reference.
- **Recommendation:** Leave OFF for cleanest chart. Enable only if you want to see all structure levels.

#### How Structure Works Now (V7.0 SMC-Aligned)

```
After Bullish BOS at swing high:
  - protectedLow = lowest low in lookback
  - impulseLow = protectedLow (fixed)
  - impulseHigh = high (extends on each new high)
  
  Subsequent breaks:
  - If close > impulseLow AND close < impulseHigh → INTERNAL (no BOS label)
  - If close < protectedLow → BEARISH BOS (structure flips to BEAR)
  - If close > impulseHigh (new high) → extends container, no label needed
```

#### Validation Checklist
- ✅ Green "bullish BOS" cannot print below current impulseHigh when structure = BEAR
- ✅ Red "bearish BOS" cannot print above current impulseLow when structure = BULL
- ✅ Impulse extremes extend on each new extreme while bias is active
- ✅ Bias flips only on protected-level breach; never on internal breaks

#### Migration Notes
- **Breaking change:** BOS labels will appear in different locations than V6.x
- **Why:** V6.x had incorrect logic that promoted internal breaks to full BOS
- **Revert:** If issues arise, checkout tag `v6.2.3` to restore previous behavior:
  ```
  git checkout v6.2.3
  ```

#### Technical Details
- Spec document: `STRUCTURE_LOGIC_FIX_SPEC.md`
- Plan document: `Structure_Logic_Plan.md`
- Key code sections modified:
  - Lines 320-330: Internal filter (symmetric container test)
  - Lines 346-366: BOS determination (protected-level gating)
  - Lines 454-465: Dynamic impulse updates (new section)

---

## [6.2.3] - 2026-01-08

### Changed
- **Pullback golden zone colors updated**
  - **Bullish zones:** Changed from green to yellow/gold (`rgb(255, 215, 0)`)
  - **Bearish zones:** Remain orange (`rgb(255, 140, 0)`)
  - **Border colors:** Updated to match zone colors (gold for bullish, orange for bearish)
  - **Highlight state:** When price enters bullish golden zone, highlights in yellow instead of lime
  - **Rationale:** Better visual distinction between structure colors (green/red) and entry zones (gold/orange)

## [6.2.2] - 2026-01-07

### Changed
- **Consolidation detection temporarily disabled**
  - **Reason:** Consolidation logic proving more challenging than expected
  - **Action:** Commented out entire consolidation detection block
  - **Result:** Indicator focuses on core functionality (BOS, swing levels, zones, pullbacks)
  - **Status:** Will revisit consolidation logic in future version

### What Still Works
✅ Break of Structure (BOS) detection and labeling
✅ Swing high/low identification with HH/HL/LH/LL labels  
✅ Protected levels (structure anchors)
✅ Supply/demand zones
✅ Pullback entry zones with Fibonacci levels
✅ Dashboard showing trend state
✅ All alerts and notifications

### Temporarily Disabled
❌ Consolidation/ranging boxes

This allows us to focus on perfecting the core Smart Money Concepts features before tackling the complex consolidation detection logic.

## [6.2.1] - 2026-01-07

### Fixed
- **Syntax error: Extra closing parenthesis**
  - **Problem:** Lines 498-500 had duplicated `border_width` and `border_style` parameters
  - **Error:** "Syntax error: Extra closing parenthesis"
  - **Solution:** Removed duplicated box.new() parameters
  - **Result:** Indicator compiles without syntax errors

## [6.2.0] - 2026-01-07

### MAJOR REWRITE - Post-BOS Consolidation Logic

**Completely replaced mathematical "tight range" detection with proper Smart Money Concepts consolidation logic.**

#### What Changed
- **REMOVED:** Mathematical tight range detection (ATR-based, oscillation checks, etc.)
- **REMOVED:** Complex swing clustering algorithms
- **ADDED:** Post-BOS consolidation detection based on market structure
- **ADDED:** Proper range definition using broken levels and next swings

#### New Logic (Aligned with SMC)
```pine
After Bullish BOS:
1. Broken swing high = top of consolidation range
2. Next swing low = bottom of consolidation range
3. Box extends until price breaks above high OR below low

After Bearish BOS:
1. Broken swing low = bottom of consolidation range  
2. Next swing high = top of consolidation range
3. Box extends until price breaks above high OR below low
```

#### Why This Is Better
**Previous Approach (Failed):**
- Tried to detect "tight ranges" mathematically
- Used ATR ratios and oscillation patterns
- Result: Boxes stretching across entire moves

**New Approach (SMC-Based):**
- Consolidation = range between structure break and next swing
- Clearly defined entry/exit criteria
- Aligned with actual market structure concepts

#### Expected Results
- Consolidation boxes appear AFTER BOS events
- Range = broken level to next opposite swing
- Clean, precise boxes that match actual market behavior
- Multiple boxes on chart showing progression through trend

#### Breaking Change
This is a fundamental change in how consolidation is detected. Previous versions may have shown different consolidation areas. The new approach is more accurate to Smart Money Concepts.

## [6.1.6] - 2026-01-07

### Fixed
- **Consolidation box vertical stretching**
  - **Problem:** Boxes were stretching from 129 to 139 (10+ points) instead of tight consolidation range
  - **Root Cause:** When `barsInTightRange` became large (30+ bars), the range calculation used ALL those bars with `ta.highest(high, lookbackToStart)`, capturing the entire price movement instead of just the current tight range
  - **Solution:** 
    ```pine
    // Before: Used full duration for range calculation
    consolidationRangeHigh := ta.highest(high, lookbackToStart)  // Could be 30+ bars
    
    // After: Use only recent bars for tight range
    rangeLookback = math.min(consolidationBars, 10)  // Max 10 bars
    consolidationRangeHigh := ta.highest(high, rangeLookback)
    ```
  - **Result:** Boxes now stay tight around actual consolidation levels (~2-3 points) instead of stretching across entire move

### Logic Separation
- **Box Width:** Still uses `barsInTightRange` for time duration (how long consolidation lasted)
- **Box Height:** Now uses `consolidationBars` (max 10) for price range (actual tight range)

This separates the "how long has it been consolidating" (time) from "what's the actual price range" (height), preventing the vertical stretching issue.

## [6.1.5] - 2026-01-07

### Fixed
- **Ultra-conservative historical buffer protection**
  - **Deep Research Findings:**
    - Error "offset (85) beyond limit (84)" on bar 12999 indicates TradingView uses different indexing than expected
    - Previous fix used `bar_index + 1` which caused overflow when `bar_index = 84`
  - **Root Cause:** Misunderstanding of TradingView's historical buffer indexing
  - **Conservative Solution:** 
    ```pine
    safeBarsFromOrigin = math.max(1, math.min(math.min(barsFromOrigin + 1, bar_index), 50))
    ```
  - **Triple Protection:**
    1. Minimum 1 bar (prevents zero-length)
    2. Maximum available bars (prevents buffer overflow) 
    3. **Maximum 50 bars (ultra-conservative cap)**
  - **Trade-off:** May not capture full impulse range in very long moves, but ensures stability

### Technical Note
The 50-bar cap is intentionally conservative. In practice, most significant price moves complete within 50 bars, and this prevents any possibility of buffer overflow while maintaining functionality.

## [6.1.4] - 2026-01-07

### Fixed
- **Final historical buffer fix with null checks**
  - **Problem:** Dynamic fib updates still caused buffer overflows when `originBar` was `na` or invalid
  - **Solution:** Added null and validity checks before calculating `barsFromOrigin`:
    ```pine
    if not na(originBar) and originBar >= 0
        barsFromOrigin = bar_index - originBar
        safeBarsFromOrigin = math.max(1, math.min(barsFromOrigin + 1, bar_index + 1))
        result = ta.highest/lowest(price, safeBarsFromOrigin)
    ```
  - **Fixed:** Proper indentation for nested logic blocks
  - **Result:** All historical buffer errors eliminated

## [6.1.3] - 2026-01-07

### Fixed
- **Complete historical buffer overflow fix**
  - **Problem:** Multiple locations in pullback zone code were causing buffer overflows
  - **Locations Fixed:**
    - Lines 1156, 1163: Bullish BOS fib calculation
    - Lines 1192, 1199: Bearish BOS fib calculation  
    - Lines 1241, 1256: Dynamic fib extreme updates
    - Line 510: Consolidation range calculation
  - **Solution:** Added `safeBarsFromOrigin = math.max(1, math.min(barsFromOrigin + 1, bar_index + 1))` to all `ta.highest/lowest` calls
  - **Result:** Indicator works correctly regardless of available historical data

### Technical Details
All historical lookbacks now use the pattern:
```pine
barsFromOrigin = bar_index - originBar
safeBarsFromOrigin = math.max(1, math.min(barsFromOrigin + 1, bar_index + 1))
result = ta.highest/lowest(price, safeBarsFromOrigin)
```

This ensures we never request more historical data than is available.

## [6.1.2] - 2026-01-07

### Fixed
- **Historical buffer overflow error**
  - **Problem:** When `barsInTightRange` was large (e.g., 85), the oscillation check loop tried to access bars beyond available history
  - **Error:** "The requested historical offset (85) is beyond the historical buffer's limit (84)"
  - **Solution:** Added `math.min(barsInTightRange, bar_index)` to limit lookback to available bars
  - **Result:** Indicator works correctly even with long consolidation periods

## [6.1.1] - 2026-01-07

### Fixed
- **Runtime error on line 468**
  - **Problem:** `ta.highest()` function received invalid length argument (0) when `barsInTightRange` was 0
  - **Solution:** Added `math.max(1, ...)` to ensure lookback is always at least 1 bar
  - **Result:** Indicator now loads without errors

## [6.1.0] - 2026-01-07

### MAJOR IMPROVEMENT - Swing-Based Consolidation Detection

**Completely rewritten consolidation algorithm to detect true sideways ranging behavior.**

#### What Changed
- **REMOVED:** 8-bar BOS delay requirement (was too restrictive)
- **REMOVED:** Simple 3-bar minimum (insufficient filtering)
- **ADDED:** Oscillation detection - price must move both up AND down within range
- **ADDED:** Configurable minimum duration (default 6 bars)
- **ADDED:** Midpoint oscillation check to ensure sideways action
- **IMPROVED:** Historical box retention - only deletes if completely engulfed

#### New Detection Logic
```pine
// Must satisfy ALL:
1. Range tight (< 2.5 ATR)
2. Duration >= 6 bars (configurable)
3. Oscillating: Price visiting both upper AND lower portions of range
   - Not just trending in one direction
   - Must have bars near top AND bottom of range
```

#### Why This Works Better
**Pullback vs Consolidation:**
- **Pullback:** Directional bias, moves toward fib levels, part of trend
- **Consolidation:** Sideways oscillation, tests both highs/lows, indecision

The oscillation check is the key differentiator - it ensures we only mark true ranging periods, not directional pullbacks.

#### New Settings
- "Minimum Duration (bars)" - default 6, range 3-20
- Controls how long price must stay in range to qualify

#### Historical Boxes
- Now only deletes if new box completely engulfs old box
- Allows multiple consolidation boxes to coexist on chart
- Shows progression of consolidations through trend

#### Migration from V6.0.x
This is a significant improvement. You may see more consolidation boxes now, which is correct - previous versions were missing legitimate consolidations due to the 8-bar BOS delay.

## [6.0.3] - 2026-01-07

### Fixed
- **Consolidation box now starts at correct position**
  - **Problem:** Box started at detection point instead of when range actually began
  - **Solution:** Changed `lookbackToStart` to use `barsInTightRange` (actual duration) instead of fixed 4 bars
  - **Result:** Box now extends backwards to capture the full consolidation period

- **Removed remaining box height updates**
  - **Problem:** Lines 545-546 were still calling `box.set_top/bottom` every bar
  - **Solution:** Removed these calls - box top/bottom are now truly locked at initial detection
  - **Result:** Box height stays fixed, no more vertical stretching

### Technical Details
- Box start: `bar_index - barsInTightRange` (dynamic based on consolidation duration)
- Box height: Set once at detection, never updated
- Box width: Only `set_right()` is called to extend forward

## [6.0.2] - 2026-01-07

### Fixed
- **Consolidation box over-expansion bug**
  - **Problem:** Consolidation boxes were stretching vertically to capture entire price range instead of staying tight
  - **Root Cause:** Dynamic expansion logic (lines 469-472) allowed box to grow when price made new highs/lows
  - **Solution:** 
    - Removed dynamic expansion - consolidation range now stays locked to initial detection size
    - Added minimum bars after BOS requirement (8 bars) before consolidation can be detected
    - Added `barsSinceImpulse` counter increment on every bar
  - **Result:** Consolidation boxes now remain compact and accurately represent the ranging price action

### Technical Changes
- Removed expansion logic that updated `consolidationRangeHigh/Low` during consolidation
- Added `sufficientTimeSinceBOS` check to prevent false consolidation signals during pullbacks
- Added proper `barsSinceImpulse += 1` increment (was missing)

## [6.0.1] - 2026-01-06

### Fixed
- **CRITICAL FIX: Structure state (BULL/BEAR) now persists correctly during consolidation**
  - **Problem:** Consolidation detection was incorrectly setting `structureState := 0`, causing background color to change from green/red to gray even without a BOS
  - **Solution:** Removed `structureState` modifications from consolidation entry/exit logic
  - **Result:**
    - Background color (green for BULL, red for BEAR) now stays until actual BOS occurs
    - Dashboard shows correct trend even during tight ranges
    - Consolidation boxes still display, but don't interfere with trend tracking
  - **Technical:** `structureState` now ONLY changes on actual BOS events, never on consolidation detection

## [6.0] - 2026-01-06

### MAJOR REWRITE - Simple Range-Based Detection

**Completely replaced pivot-based consolidation detection with simple range method.**

#### What Changed
- **REMOVED:** Pivot detection and swing array tracking
- **REMOVED:** Complex clustering checks and array management
- **REMOVED:** BOS-based array clearing logic
- **ADDED:** Simple `ta.highest()/ta.lowest()` range calculation
- **ADDED:** Real-time detection with zero lag

#### New Detection Logic (Much simpler!)
```pine
rangeHigh = ta.highest(high, 6)       // Highest in last 6 bars
rangeLow = ta.lowest(low, 6)          // Lowest in last 6 bars
rangeSize = rangeHigh - rangeLow

isRangeTight = rangeSize <= (atr * 2.5)  // Range < 2.5 ATR?

// Count bars in tight range
if isRangeTight then barsInRange++ else barsInRange = 0

isConsolidating = isRangeTight and barsInRange >= 3
```

#### Why This Works
- **Zero lag** - No waiting for pivot confirmation
- **Real-time** - Boxes appear as consolidation forms
- **Simple** - No arrays, no complex state management
- **Reliable** - Direct range measurement, can't fail
- **Tunable** - Adjust range threshold for tighter/looser detection

#### Default Settings
- Lookback: 6 bars (SHORT for tight detection)
- Max Range: 2.5 ATR (TIGHT for compact zones)
- Min Bars: 3 consecutive bars in range

#### Migration from V5.x
The pivot-based approach had fundamental timing issues that couldn't be fixed. This new approach is architecturally superior - simpler, faster, and more reliable.

## [5.7] - 2026-01-05

### Changed
- **Golden zone historical persistence**
  - Golden zones now remain visible until replaced by next zone
  - Provides historical reference without cluttering chart
  - Old zone automatically removed when new BOS creates new zone
  - No more disappearing zones - always have one visible for context

- **Dashboard text size increased**
  - All dashboard rows now use `size.normal` (previously `size.small`)
  - Better readability on all screen sizes
  - Header and data rows now consistent size

### How It Works
**Previous behavior:**
- Zone deleted when invalidated → screen goes blank
- No reference until next BOS

**New behavior:**
- Zone persists as historical reference
- Next BOS creates new zone and removes old one
- Always one zone visible for reference
- Clean chart (no clutter from multiple old zones)

## [5.6] - 2026-01-05

### Fixed
- **Golden zone premature invalidation**
  - Removed overly strict invalidation rule that deleted golden zone when price closed below Fibonacci origin
  - Valid pullbacks can go to 0.786 or even 1.0 Fibonacci levels without invalidating the setup
  - Golden zone now persists until structure actually flips (protected level broken)
  - This allows deeper pullbacks to be recognized and traded

### Why This Was Wrong
Previous versions invalidated the golden zone if price closed even 1 tick below the swing low origin. This was too restrictive because:
- Deep pullbacks (0.786, 1.0 fib) are valid entry opportunities
- Price can temporarily dip below origin without breaking structure
- Only structure flips (protected level breaks) should invalidate the zone

### Now
- Golden zone remains active during deep pullbacks
- Only invalidates when structure state changes (bearish breaks protected low, or vice versa)
- Allows traders to catch entries at deeper discount levels

## [5.5] - 2026-01-05

### Added
- **Fibonacci Origin Selection Toggle**
  - New setting: "Use Protected Low/High for Fib Origin?" (default: OFF)
  - Gives users choice between two Fibonacci calculation methods
  - Default mode: Uses swing pivot approach (traditional, backward compatible)
  - Protected mode: Uses absolute lowest/highest point in impulse (maximum accuracy)

### How It Works
**Default Mode (OFF) - Swing Pivot:**
- Bullish: Uses `prevSwingLow` (the swing low before the high that broke)
- Bearish: Uses `prevSwingHigh` (the swing high before the low that broke)
- Traditional swing-based Fibonacci approach
- Familiar to most traders

**Protected Mode (ON) - Absolute Extreme:**
- Bullish: Uses `protectedLow` (absolute lowest low in lookback range)
- Bearish: Uses `protectedHigh` (absolute highest high in lookback range)
- Captures true structural origin of the impulse
- More accurate measurement of complete impulse range

### Why This Matters
When multiple swing lows exist at different levels before BOS, the swing pivot approach may not use the **lowest** swing low. Protected mode ensures Fibonacci measures from the true impulse origin.

**Example:**
- Swing Low #1 at 4309 (true impulse origin)
- Swing Low #2 at 4370 (pullback during rally)
- BOS triggers
- **Default:** Fib from 4370 (recent swing)
- **Protected:** Fib from 4309 (true origin) ✓

### Changed
- Dynamic Fib tracking now uses correct origin bar for both modes
- Both bullish and bearish calculations support toggle

## [5.4] - 2026-01-05

### Fixed
- **Critical: Fibonacci now uses correct impulse origin**
  - Bullish: Uses `prevSwingLow` (the swing low BEFORE the high that broke)
  - Bearish: Uses `prevSwingHigh` (the swing high BEFORE the low that broke)
  - V5.3 incorrectly used `confirmedSwingLow/High` (most recent swing, not impulse origin)

### Why This Matters
- **Correct:** When price breaks a swing high, the impulse started at the swing low that came BEFORE that high
- **Previous versions** were using the wrong swing as the origin (most recent instead of impulse origin)
- Golden zone now properly measures retracement of the actual impulse that created the BOS

### Example
- Swing Low #1 at 100 (impulse origin)
- Price rallies to Swing High at 110
- Swing Low #2 at 105 (pullback during rally)
- Price breaks 110 (BOS)
- **V5.3 wrong:** Measured from Low #2 (105) - most recent
- **V5.4 correct:** Measures from Low #1 (100) - impulse origin

## [5.3] - 2026-01-05

### Changed
- **Dynamic Fibonacci updating until pullback begins**
  - Fibonacci extreme now updates continuously as price makes new highs/lows
  - Tracks the true impulse extreme even if price continues after BOS
  - Updates stop once pullback begins (enters golden zone)
  - V5.2 calculated Fib only at BOS moment (static)

### How It Works
- Bullish: 100% level updates to highest high as long as price keeps rallying
- Bearish: 100% level updates to lowest low as long as price keeps falling
- Golden zone adjusts dynamically to represent true impulse retracement
- Once price enters golden zone, Fib levels freeze (pullback has begun)

### Technical Details
- On every bar, recalculates `ta.highest/ta.lowest` from swing to current bar
- If new extreme found, updates `currentFibExtreme` and recalculates all fib levels
- Ensures golden zone always measures retracement of the complete impulse move

## [5.2] - 2026-01-05

### Changed
- **Corrected Fibonacci calculation to measure true impulse range**
  - Now uses highest high since swing low (bullish) / lowest low since swing high (bearish)
  - V5.1 incorrectly used the swing level that was broken (not the impulse extreme)
  - Aligns with traditional Fibonacci retracement methodology
  - Measures the actual impulse move from start to finish

### Technical Details
- Bullish: 0% = swing low, 100% = `ta.highest(high)` from swing low bar to current bar
- Bearish: 0% = swing high, 100% = `ta.lowest(low)` from swing high bar to current bar
- Golden zone now represents proper retracement of the full impulse move
- More accurate and consistent with standard Fib tools

## [5.1] - 2026-01-05

### Changed
- **Improved Fibonacci pullback calculation accuracy**
  - Golden zone now measures between structural swing points (swing low to swing high)
  - Previously used confirmation candle's high/low (inconsistent and arbitrary)
  - Fibonacci levels are now stable and consistent across all BOS events
  - Better alignment with traditional Fibonacci retracement methodology

### Technical Details
- Bullish: 0% = swing low, 100% = swing high (that was broken)
- Bearish: 0% = swing high, 100% = swing low (that was broken)
- Eliminates variability from confirmation candle wicks/bodies

## [5.0.1] - 2026-01-05

### Fixed
- **Consolidation detection not triggering**
  - Made V5.0 filters (time clustering & ATR compression) OPTIONAL
  - Both default to OFF for backward compatibility with V4.7 behavior
  - Users can now enable stricter filters individually

### Added
- New setting: "Require Time Clustering? (V5.0)" - defaults to OFF
- New setting: "Require ATR Compression? (V5.0)" - defaults to OFF
- New setting: "ATR Compression Ratio" - configurable threshold (default 0.90)

### Changed
- V5.0 filters are now opt-in rather than always-on
- Core consolidation detection works like V4.7 by default
- Users can progressively enable stricter filters as needed

## [5.0] - 2026-01-05

### Fixed
- **bosColor variable redeclaration bug**
  - Removed `var` keyword from bosColor declarations in BOS blocks
  - Eliminates potential compilation issues

### Changed
- **Significantly enhanced consolidation detection algorithm**
  - Added TIME clustering: swings must occur within 1.5x lookback window
  - Added ATR compression filter: requires volatility < 85% of 20-bar average
  - Added minimum bars requirement: 10 bars minimum in range before detection
  - Improved swing array management: keeps last 2 swings on BOS for context
  - Consolidation now checks 7 criteria (up from 4):
    1. Range tightness (existing)
    2. Minimum swing touches (existing)
    3. Price clustering - highs (existing)
    4. Price clustering - lows (existing)
    5. **NEW:** Time clustering - highs
    6. **NEW:** Time clustering - lows
    7. **NEW:** ATR compression
  - Should eliminate false positives on impulse moves
  - Should prevent detection when swings are too far apart in time

### Added
- New helper functions: `isTimesClustered()`, `isATRCompressed()`
- Bar index tracking arrays for swing points

### Verified
- Supply/Demand zone creation and mitigation logic confirmed working correctly

## [4.9] - 2026-01-04

### Changed
- **Dashboard text size increase for desktop readability**
  - Structure state row: size.small → size.normal
  - Timeframe row: size.small → size.normal
  - All other rows: size.tiny → size.small
  - Better balance between mobile and desktop viewing

## [4.8] - 2026-01-02

### Changed
- **Compact mobile-friendly dashboard**
  - Reduced from 11 rows to 7 rows
  - Changed text size from normal to small/tiny
  - Combined fields: TF+SwingLen, BOS+BarsSince, SwingH+SwingL
  - Shortened labels: BULLISH→BULL, WAITING▲→WAIT▲, etc.
  - Better balance for iPhone 15 Pro Max and desktop viewing
  - Renamed "Zones" to "S/D" for clarity

## [4.7] - 2026-01-02

### Changed
- **Improved consolidation detection with range tightness + swing clustering**
  - New algorithm checks: tight range + clustered swing highs + clustered swing lows
  - Range must be < X ATR (default 2.0) to qualify as consolidation
  - Swing highs must be within tolerance of each other (clustered at resistance)
  - Swing lows must be within tolerance of each other (clustered at support)
  - Exits consolidation on breakout above/below the range
  - Eliminates false consolidation detection during impulse moves

### Added
- New settings: "Max Range (ATR)", "Swing Cluster Tolerance", "Min Touches"

## [4.6] - 2026-01-02

### Changed
- **Direction-specific golden zone colors**
  - Bullish golden zone now displays in green
  - Bearish golden zone now displays in orange
  - Deep zone borders also match direction (green/red)

### Added
- Separate color settings for bull/bear zones

## [4.5] - 2026-01-02

### Changed
- **Smarter consolidation range using swing point clustering**
  - Consolidation range now calculated from actual swing highs/lows
  - Tracks last 10 swing points in arrays
  - Range = max(swing highs) to min(swing lows)
  - Requires minimum 2 swing highs AND 2 swing lows
  - Swing arrays cleared on new BOS for fresh calculation
  - Eliminates impulse moves from inflating the range

## [4.4] - 2026-01-02

### Fixed
- **Consolidation range calculation**
  - Changed from impulse range to actual recent price action
  - Uses ta.highest/ta.lowest over consolidation period
  - Box starts from lookback period, not old impulse bar

## [4.3] - 2026-01-02

### Fixed
- **Overlapping consolidation boxes**
  - Added overlap detection before creating new consolidation box
  - Removes historical boxes that overlap in price range
  - Prevents stacked boxes in same price area

## [4.2] - 2026-01-02

### Added
- **Historical consolidation zones**
  - Broken consolidation boxes now preserved as dimmed gray
  - New setting: "Show Historical Consolidation?" toggle
  - New setting: "Max Historical Boxes" limit
  - Historical boxes useful for identifying re-test zones

## [4.1] - 2026-01-02

### Fixed
- **Consolidation detection logic**
  - Added directional move check (price movement > X ATR)
  - Consolidation only triggers if price hasn't moved significantly
  - Prevents impulse moves from being labeled as consolidation

### Added
- New setting: "Max Range (ATR)" threshold

## [4.0] - 2026-01-02

### Added
- **Initial release with full rule set (A1-A8)**
  - A1: Consolidation Detection with range box
  - A4: Protected Levels (structure anchors)
  - A5: Internal Structure Filter (trap warnings)
  - A6: Displacement Strength grading (Strong/Weak BOS)
  - A7: Zone Direction Filtering
  - A8: Pullback Entry with Fib Golden Zone
  - Developing swing level lines (SH/SL before BOS)
  - Direction-specific BOS colors (green/red)
  - Dashboard with structure state and pullback status
  - Comprehensive alert system
