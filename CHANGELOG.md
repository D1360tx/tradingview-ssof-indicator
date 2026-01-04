# Changelog

All notable changes to the SSOF indicator will be documented in this file.

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
