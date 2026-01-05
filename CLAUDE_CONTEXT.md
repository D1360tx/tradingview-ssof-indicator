# CLAUDE_CONTEXT.md - Development Reference

> **Purpose:** This document provides technical context for AI assistants (Claude) or developers continuing work on the SSOF indicator. It explains architecture, design decisions, known issues, and development roadmap.

---

## Project Overview

**Smart Structure & Order Flow [SSOF]** is a TradingView Pine Script v6 indicator for market structure analysis. It identifies swing points, break of structure (BOS) events, supply/demand zones, and provides entry signals based on Fibonacci retracements.

**Primary Use Case:** Diego trades Silver/USD (XAG/USD) on 1H timeframe, holding positions 5 minutes to 8 hours.

**Current Version:** V4.9

---

## Code Architecture

### File Structure
```
ssof.pine (single file ~1500 lines)
├── Header & Version
├── INPUTS (grouped by feature)
│   ├── Auto-Timeframe Settings
│   ├── Structure Settings  
│   ├── Consolidation Detection
│   ├── Displacement Strength
│   ├── Zone Settings
│   ├── Pullback Entry (Fib)
│   ├── Confirmation Settings
│   ├── Styling
│   └── Dashboard Settings
├── TYPE DEFINITIONS
│   ├── Zone (supply/demand)
│   └── PullbackZone (fib entry)
├── ARRAYS
│   ├── zigZagLines
│   ├── activeZones
│   ├── historicalConsolidationBoxes
│   ├── recentSwingHighs (for consolidation)
│   └── recentSwingLows (for consolidation)
├── HELPER FUNCTIONS
├── SWING DETECTION (ta.pivothigh/pivotlow)
├── MARKET STRUCTURE LOGIC
│   ├── BOS Detection
│   ├── Protected Levels
│   ├── Internal Structure Filter
│   └── Consolidation Detection
├── VISUALIZATION
│   ├── Protected Level Lines
│   ├── Zig-Zag Lines
│   ├── Swing Labels
│   ├── BOS Labels & Lines
│   ├── Supply/Demand Zones
│   ├── Developing Levels
│   ├── Liquidity Sweeps
│   ├── Pullback Entry System
│   └── Dashboard
├── ALERTS
└── PLOTS
```

### Key Variables

#### Structure State
```pine
structureState: int
  1  = Bullish (making HH/HL)
  -1 = Bearish (making LH/LL)
  0  = Consolidating (range-bound)
```

#### Swing Points
```pine
confirmedSwingHigh / confirmedSwingLow  // Current confirmed pivots
prevSwingHigh / prevSwingLow            // Previous pivots (for HH/LL detection)
confirmedHighBar / confirmedLowBar      // Bar indices of pivots
```

#### Protected Levels (A4)
```pine
protectedLow   // In bullish structure, losing this = structure break
protectedHigh  // In bearish structure, losing this = structure break
```

#### Consolidation
```pine
wasConsolidating: bool           // State flag
consolidationRangeHigh/Low       // Box boundaries
recentSwingHighs/Lows: array     // For clustering detection
```

#### Pullback Entry
```pine
pullbackDirection: int           // 1 = bullish setup, -1 = bearish
inGoldenZone: bool              // Price in 0.5-0.618 zone
pullbackConfirmed: bool         // Entry confirmation triggered
```

---

## Feature Implementation Status

### ✅ Fully Implemented

| Feature | Rule | Notes |
|---------|------|-------|
| Swing Detection | - | Uses `ta.pivothigh/pivotlow` with configurable length |
| BOS Detection | - | Closes above/below confirmed swing |
| Zig-Zag Lines | - | Connects alternating HH-HL or LH-LL |
| HH/HL/LH/LL Labels | - | Color-coded with price |
| Protected Levels | A4 | Tracks structure invalidation points |
| Internal Structure Filter | A5 | Warns of counter-trend breaks (traps) |
| Displacement Strength | A6 | Grades BOS as Strong/Weak by candle body/ATR |
| Zone Direction Filter | A7 | Only shows demand in bull, supply in bear |
| Pullback Entry (Fib) | A8 | Golden zone with confirmation modes |
| Supply/Demand Zones | - | Created at BOS, removed on mitigation |
| Auto-Timeframe | - | Adjusts swing length per TF |
| Dashboard | - | Compact 7-row status display |
| Alerts | - | BOS, structure flip, entries, sweeps |

### ⚠️ Needs Work

| Feature | Issue | Notes |
|---------|-------|-------|
| Consolidation Detection | A1 | Current algorithm still not accurate - see Known Issues |

### ❌ Not Implemented

| Feature | Rule | Notes |
|---------|------|-------|
| Multi-Timeframe Overlay | A2 | Show HTF structure on LTF chart |
| Liquidity Pools | A3 | Identify resting orders at swing points |

---

## Known Issues & Bugs

### 1. Consolidation Detection (HIGH PRIORITY)

**Problem:** The consolidation box often appears in wrong places or doesn't capture obvious consolidation zones.

**Current Algorithm (V4.7+):**
```
Consolidation = TRUE when ALL:
  1. Range < consolidationRangeATR * ATR (default 2.0)
  2. At least minSwingTouches swing highs AND lows exist (default 2)
  3. Swing highs are clustered within swingClusterATR * ATR
  4. Swing lows are clustered within swingClusterATR * ATR
```

**What's Been Tried:**
- V4.1: Bars since BOS + directional move check → Failed (impulse moves labeled as consolidation)
- V4.4: ta.highest/ta.lowest over lookback → Failed (captured impulse candles)
- V4.5: Swing point arrays, max/min of swings → Better but still issues
- V4.7: Swing clustering + range tightness → Improved but not perfect

**What Might Work:**
- Require swings to be within X bars of each other (time clustering, not just price)
- Detect actual "tests" of levels (wicks touching but not breaking)
- Use ATR compression as additional filter
- Track swing sequence (needs alternating H-L-H-L pattern)

### 2. Zone Overlap

Occasionally supply/demand zones can overlap or stack. Current mitigation removes zones on price pass-through.

---

## Design Decisions & Rationale

### Why Separate Bullish/Bearish BOS Colors?
V4.0 used same colors for both. Changed because visually confusing - hard to quickly identify direction.
- Bullish BOS: Green shades (strong: bright, weak: dim)
- Bearish BOS: Red shades (strong: bright, weak: dim)

### Why Direction-Specific Golden Zone Colors?
V4.6 change. Yellow/gold was ambiguous. Now:
- Bullish pullback: Green box
- Bearish pullback: Orange box

### Why Compact Dashboard?
User trades on iPhone 15 Pro Max. Original 11-row dashboard too large on mobile. V4.8 reduced to 7 rows with combined fields. V4.9 increased text size back to `size.normal` for desktop readability.

### Why Track Historical Consolidation Boxes?
V4.2 addition. Broken consolidation zones can act as future S/R for re-tests. Toggle allows users to keep chart clean if preferred.

### Why Swing Clustering for Consolidation?
True consolidation has multiple tests at similar levels (support/resistance forming). Just counting bars or measuring range isn't enough - need to verify price is actually oscillating between defined levels.

---

## Settings Reference

### Auto-Timeframe Defaults
| Timeframe | Swing Length |
|-----------|--------------|
| 1m | 3 |
| 5m | 5 |
| 15m | 7 |
| 30m | 5 |
| 1H | 10 |
| 4H | 15 |
| Daily | 20 |

### Consolidation Settings
| Setting | Default | Purpose |
|---------|---------|---------|
| Lookback Bars | 20 | Period to analyze |
| Max Range (ATR) | 2.0 | Tightness threshold |
| Swing Cluster Tolerance | 0.5 | How close swings must cluster |
| Min Touches | 2 | Minimum swings each side |

### Pullback Entry Settings
| Setting | Default | Purpose |
|---------|---------|---------|
| Fib Upper Level | 0.5 | Top of golden zone |
| Fib Lower Level | 0.618 | Bottom of golden zone |
| Fib Deep Level | 0.786 | Discount zone (optional) |
| Confirmation Type | Internal BOS | Entry trigger |
| Displacement ATR | 0.5 | Candle body threshold |

---

## Versioning Scheme

- **Major upgrades** (new features, significant changes): +1.0 → V5, V6, V7...
- **Minor updates** (bug fixes, algorithm changes): +0.1 → V4.1, V4.2, V4.3...
- **Small tweaks** (dashboard size, colors, labels): +0.0.1 → V4.9.1, V4.9.2...

**Examples:**
| Change Type | Version Bump |
|-------------|--------------|
| New consolidation algorithm | V4.7 → V4.8 |
| Fix compilation error | V4.8 → V4.9 |
| Change dashboard text size | V4.9 → V4.9.1 |
| Change golden zone color | V4.9.1 → V4.9.2 |
| Add multi-timeframe overlay feature | V4.9.2 → V5.0 |

---

## Development Workflow

1. User provides feedback via annotated screenshots with numbered reference points
2. Identify specific issues from screenshot
3. Locate relevant code section
4. Implement fix/feature
5. Update version number in `indicator()` call
6. Update CHANGELOG.md
7. Update this CLAUDE_CONTEXT.md if architecture changes
8. Export to outputs folder

---

## Future Roadmap

### Near-term (V4.x)
- [ ] Fix consolidation detection algorithm
- [ ] Add consolidation breakout alerts
- [ ] Improve zone sizing consistency

### Medium-term (V5.x)
- [ ] Multi-timeframe structure overlay (A2)
- [ ] Liquidity pool identification (A3)
- [ ] Session-based analysis (London/NY/Asia)

### Long-term (V6.x)
- [ ] Order block detection
- [ ] Fair value gap highlighting
- [ ] Backtesting statistics

---

## Testing Notes

**Primary Test Instruments:**
- XAG/USD (Silver) - 1H timeframe
- ZEC/USDT - 15m timeframe
- SOL/USDT - 4H, Daily
- XAU/USD (Gold) - 1H timeframe
- MU (Micron) - 1H timeframe

**What to Check After Changes:**
1. No compilation errors
2. Dashboard displays correctly
3. BOS labels appear at correct locations
4. Zig-zag lines connect properly (alternating H-L-H-L)
5. Zones appear at structure breaks
6. Pullback fib zones draw correctly after BOS
7. Colors match direction (green=bull, red/orange=bear)

---

## Code Snippets for Common Tasks

### Adding a New Setting
```pine
grpNewFeature = "🆕 New Feature"
newSetting = input.bool(true, "Enable?", group=grpNewFeature, tooltip="Description")
newValue = input.float(1.0, "Value", minval=0.1, maxval=10.0, group=grpNewFeature)
```

### Adding a Dashboard Row
```pine
// Increase table rows: table.new(dashY, 2, 7, ...) → table.new(dashY, 2, 8, ...)
table.cell(dashboard, 0, 7, "Label", text_color=color.gray, text_size=size.normal)
table.cell(dashboard, 1, 7, valueText, text_color=valueColor, text_size=size.normal)
```

### Adding an Alert
```pine
alertcondition(condition, title="Alert Title", message="Message on {{ticker}} {{interval}}")
```

---

## Contact & Resources

- **GitHub:** https://github.com/D1360tx/tradingview-ssof-indicator
- **TradingView Docs:** https://www.tradingview.com/pine-script-docs/
- **Pine Script v6 Reference:** https://www.tradingview.com/pine-script-reference/v6/

---

*Last Updated: 2026-01-04 (V4.9)*
