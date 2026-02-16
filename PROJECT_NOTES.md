# FOB Calculator - Project Notes

## Overview
A bi-directional FOB (Free On Board) cost calculator that converts between NZD and Japanese Yen. It applies various GST-exempt deductions (brokerage, freight, compliance, etc.), correctly handles GST in both directions, and uses a live exchange rate with a configurable rate adjustment and a 0.70 customer margin.

Built for Megan starting 2026-02-15.

## Current State: Phase 2 - FOB Calculator (Complete)
- Single HTML file: `CurrencyConverter.html`
- Desktop copy: `D:/Users/Megan/OneDrive/Desktop/CurrencyConverter.html`
- Project folder: `C:/ClaudeProjects/Currency Converter/`
- Spreadsheet reference: `Currency Converter.xlsx` (Megan's original spec, includes Sample Calculation tab)

## App Name
**FOB Calculator** (renamed from "Currency Converter" / "NZD to JPY Calculator")

## Bi-Directional Conversion
The app has a direction toggle at the top to switch between two modes:

### NZD to Yen (default)
1. Input NZD Cost to Customer
2. Subtract ALL applicable fixed costs (all are GST-exempt)
3. Remaining = GST-inclusive base
4. Extract GST (base x 3/23 = 15% NZ GST)
5. NZD base (excl GST) x customer yen rate = Yen Total

### Yen to NZD
1. Input Yen amount
2. Divide by customer yen rate = NZD base (excl GST)
3. Add 15% GST = NZD base (incl GST)
4. Add ALL applicable fixed costs (GST-exempt)
5. = NZD Cost to Customer

## GST Logic (Critical)
**ALL fixed costs are GST-exempt.** GST only applies to the core NZD amount derived from/going to the yen conversion. This was the key insight that makes both directions produce identical results:
- NZD→Yen: subtract costs first, then extract GST from what remains
- Yen→NZD: calculate GST on the base, then add costs on top
- Both directions give the same GST amount because they work on the same base

This was verified against Megan's spreadsheet Sample Calculation tab (Scenario 1 and Scenario 2).

## Fixed Costs / Variables
All costs are GST-exempt and configurable via the Variables panel.

| Item | Default |
|---|---|
| Spot Buy Rate Adjustment | -0.80 |
| Brokerage | $575 |
| Freight | $2,262 |
| Marine Insurance GST | $110 |
| Statement of Compliance | $390 |
| Compliance (European) | $1,900 |
| Compliance (Other) | $1,100 |
| Heat Treatment | $225 |

### Conditional Logic
- **Brokerage, Freight & Marine Insurance GST**: Always applied automatically
- **European?** (Y/N toggle): Controls what options appear
- **Statement of Compliance?**: Only shown if European = Yes. Subtracts $390
- **Compliance?**: If European + Yes → $1,900; if Not European + Yes → $1,100
- **Heat Treatment?**: If Yes → $225

## Variables Panel Features
- **"Variables" button** in top-right header opens/closes the panel
- Clean table layout showing Spot Buy Rate Adjustment + all 7 cost values
- **Auto-saves** to browser localStorage on every edit (persists across sessions)
- **"Reset to Defaults"** button: restores values to saved defaults
- **"Save as New Defaults"** button: saves current values as new defaults
  - Custom defaults stored separately in localStorage (`fobCalcDefaults`)
  - "Reset" will then restore to these custom defaults instead of the original hardcoded ones
- Shows "Saved" / "New defaults saved" confirmation flash

## UI Layout (top to bottom)
1. **Header row**: "FOB Calculator" title + Variables button
2. **Rate info**: Estimated Spot Buy rate and customer rate
3. **Direction toggle**: NZD to Yen / Yen to NZD
4. **Input field**: $ or ¥ prefix changes with direction
5. **Result boxes** (side by side): NZD Cost to Customer + Yen Total
   - Input currency box is dimmed/dashed
   - Output currency box is highlighted/scaled up
   - Swap automatically when direction toggles
6. **Auto deductions**: Brokerage, Freight, and Marine Insurance GST (always on)
7. **Options toggles**: European, SoC, Compliance, Heat Treatment
8. **Breakdown**: Full step-by-step calculation (dynamic, changes with direction)
9. **Status**: Rate update timestamp

## Tech Stack
- Pure HTML / CSS / JavaScript (single file, ~900 lines)
- No frameworks, no dependencies
- API: `https://open.er-api.com/v6/latest/NZD` (free, no key required)
- Rate refreshes every 30 minutes, fallback to 90.0 if offline
- localStorage for persisting variable costs and custom defaults
- All NZD figures rounded to whole numbers (zero decimal places)
- Hosted on GitHub Pages for access from any device

## Design
- Dark theme: navy gradient background (#1a1a2e → #0f3460)
- Glassmorphism card with backdrop blur
- Accent blue (#5dade2) for NZD, red (#e74c3c) for Yen
- Green (#2ecc71) for toggle switches and auto labels
- Responsive, mobile-friendly (460px card width)

## Yen Rate
- Mid rate fetched live from `open.er-api.com`
- **Estimated Spot Buy rate** = API mid rate + Spot Buy Rate Adjustment (default -0.80, configurable in Variables)
- **Customer rate** = Estimated Spot Buy rate - 0.70 (margin hardcoded)
- Both rates displayed at top of app

## localStorage Keys
- `fobCalcCosts` - current variable cost values
- `fobCalcDefaults` - custom default values (set via "Save as New Defaults")

## Phase History
- **Phase 1** (2026-02-15): Simple NZD/JPY converter with toggle - REPLACED
- **Phase 2** (2026-02-15): Full calculation flow from spreadsheet - REPLACED
- **Phase 2b** (2026-02-15): Fixed GST to exclude Freight and Heat Treatment - REPLACED
- **Phase 2c** (2026-02-15): Fixed GST to exclude ALL fixed costs (Brokerage, Freight, Heat Treatment, SoC, Compliance) - KEY FIX
- **Phase 2d** (2026-02-15): Added bi-directional conversion (Yen→NZD and NZD→Yen) with correct GST in both directions - KEY FEATURE
- **Phase 2e** (2026-02-15): Moved result boxes next to input, added input/output highlighting
- **Phase 2f** (2026-02-15): Renamed to FOB Calculator, upgraded Variables panel with localStorage persistence, table layout, and "Save as New Defaults" feature
- **Phase 2g** (2026-02-15): Rounded all NZD figures to zero decimal places
- **Phase 3** (2026-02-15): Deployed to GitHub Pages
- **Phase 3b** (2026-02-15): Added custom Apple Touch Icon for iPhone home screen
- **Phase 3c** (2026-02-16): Added Marine Insurance GST ($110) as always-on auto deduction
- **Phase 3d** (2026-02-16): Added configurable Spot Buy Rate Adjustment variable (default -0.80), renamed rate labels
- **Phase 3e** (2026-02-16): Fixed variables not updating on mobile (added change event, saveAsDefaults triggers recalculation) - CURRENT

## Hosting / GitHub
- **Live URL**: https://meganwood321.github.io/fob-calculator/
- **GitHub repo**: https://github.com/meganwood321/fob-calculator
- **GitHub username**: meganwood321
- **Branch**: main
- **GitHub Pages**: enabled, serves from main branch root
- `index.html` redirects to `CurrencyConverter.html`
- To update: push changes to main branch, site auto-updates in ~1 min
- localStorage is per-browser/device (each computer has its own saved Variables)
- **Apple Touch Icon**: `$ → ¥.png` - custom icon for iPhone home screen shortcut
- Git credentials stored locally (no need to re-enter token for pushes)

## Key File Locations
- **Megan's actual Desktop**: `D:/Users/Megan/OneDrive/Desktop/` (OneDrive-synced)
- **Project folder**: `C:/ClaudeProjects/Currency Converter/`
- **Git repo**: `C:/ClaudeProjects/Currency Converter/.git/`
- **Note**: `C:/Users/Megan/Desktop/` is NOT the visible desktop
