# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

**Smart Structure & Order Flow [SSOF]** is a TradingView Pine Script v6 indicator for market structure analysis and trade identification. The indicator identifies swing points, break of structure (BOS) events, supply/demand zones, and provides entry signals based on Fibonacci retracements.

**Primary Trading Context:** Designed for XAG/USD (Silver) trading on 1H timeframe with 5-minute to 8-hour hold times, but adaptable to other instruments and timeframes.

## Development Commands

### Testing Changes
```bash
# 1. Copy ssof.pine content
# 2. Open TradingView Pine Editor (https://www.tradingview.com/pine-editor/)
# 3. Paste code and click "Add to Chart"
# 4. Test on instruments: XAG/USD (1H), XAU/USD (1H), ZEC/USDT (15m), SOL/USDT (4H/Daily)
```

### Version Management
```bash
# View recent changes
git log --oneline -10

# Commit pattern (always include co-author)
git commit -m "feat: description

Co-Authored-By: Warp <agent@warp.dev>"
```

### Validation Checklist After Changes
1. No Pine Script compilation errors
2. Dashboard displays correctly (7 rows, compact format)
3. BOS labels appear at correct swing points
4. Zig-zag lines alternate H-L-H-L properly
5. Zones created at structure breaks
6. Pullback fib zones draw after BOS
7. Color coding matches direction (green=bull, red/orange=bear)

## Code Architecture

### Single-File Structure
The entire indicator is in `ssof.pine` (~1500 lines) organized as:

```
├── INPUTS (grouped settings)
│   ├── Auto-Timeframe Settings
│   ├── Structure Settings  
│   ├── Consolidation Detection
│   ├── Displacement Strength (A6)
│   ├── Zone Settings (A7)
│   ├── Pullback Entry (A8)
│   └── Dashboard Settings
├── TYPE DEFINITIONS
│   ├── Zone (supply/demand)
│   └── PullbackZone (fib entry)
├── ARRAYS (state management)
│   ├── zigZagLines
│   ├── activeZones
│   ├── historicalConsolidationBoxes
│   ├── recentSwingHighs/Lows
├── HELPER FUNCTIONS
├── PIVOT DETECTION (ta.pivothigh/pivotlow)
├── MARKET STRUCTURE LOGIC
│   ├── BOS Detection
│   ├── Protected Levels (A4)
│   ├── Internal Structure Filter (A5)
│   └── Consolidation Detection (A1)
├── VISUALIZATION
│   ├── Zig-Zag Lines
│   ├── Swing Labels
│   ├── BOS Labels & Lines
│   ├── Supply/Demand Zones
│   ├── Pullback Entry System
│   └── Dashboard
└── ALERTS
```

### Core State Variables

**Structure State** (`structureState: int`)
- `1` = Bullish (making HH/HL)
- `-1` = Bearish (making LH/LL)
- `0` = Consolidating (range-bound)

**Swing Tracking**
- `confirmedSwingHigh/Low`: Current confirmed pivot levels
- `prevSwingHigh/Low`: Previous pivots (for HH/LL detection)
- `confirmedHighBar/LowBar`: Bar indices of pivots

**Protected Levels (A4 Rule)**
- `protectedLow`: In bullish structure, breaking this invalidates trend
- `protectedHigh`: In bearish structure, breaking this invalidates trend

**Consolidation State (A1 Rule)**
- `wasConsolidating: bool`: Current consolidation state
- `consolidationRangeHigh/Low`: Box boundaries
- `recentSwingHighs/Lows: array`: For swing clustering detection

**Pullback Entry (A8 Rule)**
- `pullbackDirection: int`: 1 = bullish setup, -1 = bearish
- `inGoldenZone: bool`: Price within 0.5-0.618 retracement
- `pullbackConfirmed: bool`: Entry confirmation triggered

### Auto-Timeframe Mapping

When Auto-TF is enabled, swing length adjusts automatically:

| Timeframe | Swing Length |
|-----------|--------------|
| 1m        | 3            |
| 5m        | 5            |
| 15m       | 7            |
| 30m       | 5            |
| 1H        | 10           |
| 4H        | 15           |
| Daily     | 20           |
| Weekly    | 25           |

## Feature Rules (A1-A8)

The indicator implements a specific trading methodology with numbered rules:

- **A1: Consolidation Detection** - Range-bound market identification using swing clustering
- **A4: Protected Levels** - Structure anchors that invalidate trend when broken
- **A5: Internal Structure Filter** - Warns of counter-trend breaks within impulse range (potential traps)
- **A6: Displacement Strength** - Grades BOS as Strong (S) or Weak (W) based on candle body/ATR ratio
- **A7: Zone Direction Filtering** - Only shows demand zones in bullish structure, supply in bearish
- **A8: Pullback Entry** - Fib golden zone (0.5-0.618) with confirmation modes

**Not Yet Implemented:**
- A2: Multi-Timeframe Overlay (HTF structure on LTF chart)
- A3: Liquidity Pools (resting orders at swing points)

## Known Issues

### Consolidation Detection (HIGH PRIORITY)
**Problem:** Algorithm still triggers in wrong places or misses obvious consolidation zones.

**Current Algorithm (V4.7+):**
```
Consolidation = TRUE when ALL:
  1. Range < consolidationRangeATR * ATR (default 2.0)
  2. At least minSwingTouches (2+) swing highs AND lows exist
  3. Swing highs clustered within swingClusterATR * ATR
  4. Swing lows clustered within swingClusterATR * ATR
```

**Attempted Solutions:**
- V4.1: Bars since BOS + directional move → Failed (impulse moves labeled as consolidation)
- V4.4: ta.highest/lowest over lookback → Failed (captured impulse candles)
- V4.5: Swing point arrays → Improved but issues remain
- V4.7: Swing clustering + range tightness → Current approach

**Potential Solutions:**
- Add time clustering requirement (swings within X bars of each other)
- Detect actual "tests" of levels (wick touches without breaks)
- Use ATR compression as filter
- Verify alternating H-L-H-L swing sequence

### Zone Overlap
Supply/demand zones occasionally stack. Mitigation removes zones on price pass-through.

## Versioning Strategy

- **Major upgrades** (+1.0): New features, significant changes → V5, V6, V7
- **Minor updates** (+0.1): Bug fixes, algorithm changes → V4.1, V4.2, V4.3
- **Small tweaks** (+0.0.1): Dashboard size, colors, labels → V4.9.1, V4.9.2

Examples:
- New consolidation algorithm: V4.7 → V4.8
- Fix compilation error: V4.8 → V4.9
- Change dashboard text size: V4.9 → V4.9.1
- Add multi-timeframe overlay: V4.9 → V5.0

**Current Version:** V4.8 (check `indicator()` call in ssof.pine)

## Design Rationale

### Direction-Specific Colors
- **Bullish BOS:** Green shades (strong: bright, weak: dim)
- **Bearish BOS:** Red shades (strong: bright, weak: dim)
- **Bullish Pullback Zone:** Green box
- **Bearish Pullback Zone:** Orange box

Rationale: V4.0 used uniform colors which were visually confusing for quick direction identification.

### Compact Dashboard (7 rows)
```
┌──────────┬─────────────┐
│ BULL     │ 60 | 10     │  Structure + TF/SwingLen
│ H 73.77  │ L 70.07     │  Swing levels
│ Prot     │ 70.86       │  Protected level
│ BOS      │ S | 22b     │  Strength + bars since
│ S/D      │ 8           │  Active zones
│ Entry    │ WAIT▲       │  Pullback status
│ TF       │ Auto        │  Auto-TF mode
└──────────┴─────────────┘
```

Rationale: User trades on iPhone 15 Pro Max. Original 11-row dashboard was too large for mobile. V4.8 reduced to 7 rows with combined fields. V4.9 adjusted text size for desktop readability.

### Swing Clustering for Consolidation
True consolidation requires multiple tests at similar levels (support/resistance forming). Range measurement alone isn't sufficient - need to verify price oscillates between defined levels.

### Historical Consolidation Boxes
Broken consolidation zones preserved as dimmed gray boxes can act as future support/resistance for re-tests.

## Common Modification Patterns

### Adding a New Setting
```pine
grpNewFeature = "🆕 New Feature"
newSetting = input.bool(true, "Enable?", group=grpNewFeature, tooltip="Description")
newValue = input.float(1.0, "Value", minval=0.1, maxval=10.0, group=grpNewFeature)
```

### Adding a Dashboard Row
```pine
// Increase row count: table.new(dashY, 2, 7, ...) → table.new(dashY, 2, 8, ...)
table.cell(dashboard, 0, 7, "Label", text_color=color.gray, text_size=size.normal)
table.cell(dashboard, 1, 7, valueText, text_color=valueColor, text_size=size.normal)
```

### Adding an Alert
```pine
alertcondition(condition, title="Alert Title", message="Message on {{ticker}} {{interval}}")
```

## References

- **Repository:** https://github.com/D1360tx/tradingview-ssof-indicator
- **TradingView Pine Docs:** https://www.tradingview.com/pine-script-docs/
- **Pine Script v6 Reference:** https://www.tradingview.com/pine-script-reference/v6/
- **Development Context:** See CLAUDE_CONTEXT.md for detailed technical background
- **Change History:** See CHANGELOG.md for version history

## Development Workflow

When user provides feedback:
1. Review annotated screenshots with numbered reference points
2. Identify specific issues from visuals
3. Locate relevant code section in ssof.pine
4. Implement fix/feature
5. Update version number in `indicator()` call
6. Update CHANGELOG.md with changes
7. Update CLAUDE_CONTEXT.md if architecture changes
8. Test on primary instruments (XAG/USD, XAU/USD, etc.)
