# Changelog

All notable changes to the SSOF indicator will be documented in this file.

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
