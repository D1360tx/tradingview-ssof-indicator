# Smart Structure & Order Flow [SSOF]

A comprehensive TradingView indicator for market structure analysis and high-probability trade identification.

![Pine Script](https://img.shields.io/badge/Pine%20Script-v6-green)
![Version](https://img.shields.io/badge/version-7.0.1--alpha-blue)
![License](https://img.shields.io/badge/license-MPL%202.0-orange)

> **⭐ Current Stable: V7.0.1-alpha** - See [CHANGELOG.md](CHANGELOG.md) for critical SMC fixes to protected levels and reversal detection.

## Features

### Core Structure Analysis
- **Swing High/Low Detection** - Automatically identifies market pivot points
- **Break of Structure (BOS)** - Detects bullish and bearish structure breaks
- **Zig-Zag Visualization** - Connects swing points with directional colored lines
- **Higher Highs/Lows & Lower Highs/Lows** - Labels structure progression

### Advanced Rules (A1-A8)
- **A1: Consolidation Detection** - Identifies range-bound markets using swing clustering
- **A4: Protected Levels** - Tracks structure anchors that invalidate trend
- **A5: Internal Structure Filter** - Warns of potential traps with counter-trend breaks
- **A6: Displacement Strength** - Grades BOS as Strong (S) or Weak (W) based on candle characteristics
- **A7: Zone Direction Filtering** - Only shows demand zones in bullish structure, supply in bearish
- **A8: Pullback Entry (Fib Golden Zone)** - Draws fib retracement zones for entry timing

### Entry System
- **Golden Zone (0.5 - 0.618)** - Optimal pullback entry area
- **Deep Zone (0.618 - 0.786)** - Higher risk/reward discount zone
- **Entry Confirmation** - Displacement candle or Internal BOS confirmation modes
- **Direction-specific colors** - Green for bullish setups, orange for bearish

### Visual Features
- **Supply/Demand Zones** - Auto-generated at BOS events
- **Liquidity Sweeps** - Detects false breakouts
- **Developing Levels** - Shows SH/SL lines before BOS confirmation
- **Compact Dashboard** - Mobile-friendly status display

### Auto-Timeframe Adjustment
Automatically adjusts swing length based on chart timeframe:
- 1m: 3 | 5m: 5 | 15m: 7 | 30m: 5 | 1H: 10 | 4H: 15 | Daily: 20

## Installation

1. Open TradingView and go to Pine Editor
2. Copy the contents of `ssof.pine`
3. Paste into Pine Editor
4. Click "Add to Chart"

## Settings

### Structure Settings
| Setting | Default | Description |
|---------|---------|-------------|
| Swing Length | Auto | Number of bars for pivot detection |
| Show Labels | ON | Display HH/HL/LH/LL labels |
| Show Zig-Zag | ON | Draw lines connecting swings |

### Consolidation Detection
| Setting | Default | Description |
|---------|---------|-------------|
| Lookback Bars | 20 | Period to analyze |
| Max Range (ATR) | 2.0 | Maximum range to qualify as consolidation |
| Swing Cluster Tolerance | 0.5 | How close swings must cluster |
| Min Touches | 2 | Minimum swings on each side |

### Pullback Entry
| Setting | Default | Description |
|---------|---------|-------------|
| Fib Upper Level | 0.5 | Top of golden zone |
| Fib Lower Level | 0.618 | Bottom of golden zone |
| Confirmation Type | Internal BOS | Entry trigger method |

## Dashboard

Compact 7-row display showing:
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

## Alerts

- Strong/Weak Bullish BOS
- Strong/Weak Bearish BOS
- Structure change (Bull → Bear, Bear → Bull)
- Consolidation breakout
- Pullback entry (zone entry + confirmation)

## License

Mozilla Public License 2.0

## Author

© SSOF - Smart Structure & Order Flow
