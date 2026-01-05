# Changelog

All notable changes to the SSOF indicator will be documented in this file.

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
