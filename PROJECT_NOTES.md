# Jason's Calculator - Project Notes

## Overview
A bi-directional FOB cost calculator that converts between NZD and Japanese Yen. It applies various GST-exempt deductions (brokerage, freight, compliance, etc.), correctly handles GST in both directions, uses a live exchange rate with a configurable rate adjustment and a 0.70 customer margin. Includes Car Cost calculation (FOB minus agent fee) and total Margin calculation.

Built for Megan starting 2026-02-15. Renamed to "Jason's Calculator" on 2026-02-22.

## Current State: Phase 6 - Reworked Yen→NZD flow, Japan costs, UI improvements (Complete)
- Single HTML file: `CurrencyConverter.html`
- Desktop copy: `D:/Users/Megan/OneDrive/Desktop/CurrencyConverter.html`
- Project folder: `C:/ClaudeProjects/Currency Converter/`
- Spreadsheet reference: `Currency Converter.xlsx` (original spec, includes Agent Fees table and Input Table)

## App Name
**Jason's Calculator** (renamed from "FOB Calculator" / "Currency Converter")

## Bi-Directional Conversion
The app has a direction toggle at the top. **Defaults to Yen to NZD** on load.

### NZD to Yen
- Input label: "Input - NZD Cost to Client"
1. Input NZD Cost to Client
2. Subtract ALL applicable fixed costs (all are GST-exempt)
3. Remaining = GST-inclusive base
4. Extract GST (base x 3/23 = 15% NZ GST)
5. NZD base (excl GST) x customer yen rate = FOB (yen)

### Yen to NZD (default)
- Input label: "Input - Yen FOB"
1. Input Yen FOB amount
2. Divide by customer yen rate = NZD base (excl GST)
3. Add 15% GST = NZD base (incl GST)
4. Add ALL applicable fixed costs (GST-exempt)
5. = NZD Cost to Customer

## Car Cost Calculation

### Yen → NZD (default direction, left side of toggle)
- **Input**: Car Cost (yen, ÷1000)
- Agent fee = direct tier lookup from car cost (no iteration needed)
- **FOB (yen)** = Car Cost + Agent Fee + Additional Japan Costs
- FOB ÷ customer rate → NZD base excl GST → + 15% GST → + NZ fixed costs → **NZD Cost to Customer**
- Left result box: NZD Cost to Customer | Right result box: FOB (yen)

### NZD → Yen (right side of toggle)
- **Input**: NZD Cost to Customer
- Subtract NZ costs → extract GST → × customer rate → **FOB (yen)**
- **Car Cost (yen)** = FOB − Agent Fee − Additional Japan Costs (iterative solver)
- Left result box: Car Cost (yen) | Right result box: FOB (yen)
- All agents use the same tiered fee schedule — iterative solver resolves the circular reference

## Agent Fee Data
All agents share one 28-tier fee schedule, defined once as `TIERED_FEES` and referenced by every agent in `AGENT_FEES`. Each tier has `fee` (oncharge) and `margin` (oncharge - actual agent fee).

### Agents (dropdown)
Nichibo, WEINS, NTP, Autobacs, Gulliver, Heiwa — all use the identical tiered schedule:
- 28 tiers from ¥1 to ¥14,000,000, fees from ¥88,000 to ¥795,000
- Agent selection still matters for **freight** (Nichibo has different freight, see below)

## Margin Calculation
Displayed in a row under Car Cost. Total margin = 3 parts:

### Part 1: Variable Margins
Charge to customer minus actual cost for each applicable variable:

| Variable | Charge (default) | Actual Cost | Margin |
|---|---|---|---|
| Brokerage | $575 | $0 | $575 |
| Freight (Nichibo) | $2,072 | $1,570 | $502 |
| Freight (others) | $2,262 | $1,850 | $412 |
| FAF | $1 | $1 | $0 |
| BAF | $1 | $1 | $0 |
| Marine Insurance | $110 | $110 | $0 |
| Statement of Compliance | $440 | $390 | $50 |
| Compliance (European) | $1,900 | $1,900 | $0 |
| Compliance (Other) | $1,100 | $1,100 | $0 |
| Heat Treatment | $225 | $225 | $0 |
| EV Loading | $575 | $575 | $0 |

Actual costs stored in `ACTUAL_COSTS` constant (except freight, which uses `AGENT_FREIGHT`). Margins recalculate dynamically if variables are changed.

### Part 2: Agent Fee Margin
= Agent margin in yen / customer rate
(margin = oncharge fee - actual agent fee, per tier)

### Part 3: Rate Markup
= FOB_yen × 0.70 / (customerRate × estimatedRate)
Profit from the 0.70 gap between estimated spot buy rate and customer rate on all FOB yen.

## GST Logic (Critical)
**ALL fixed costs are GST-exempt.** GST only applies to the core NZD amount derived from/going to the yen conversion.
- NZD→Yen: subtract costs first, then extract GST from what remains
- Yen→NZD: calculate GST on the base, then add costs on top

## Fixed Costs / Variables
All costs are GST-exempt and configurable via the Variables panel.

| Item | Default |
|---|---|
| Spot Buy Rate Adjustment | -0.80 |
| Brokerage | $575 |
| FAF | $1 |
| BAF | $1 |
| Marine Insurance GST | $110 |
| Statement of Compliance | $440 |
| Compliance (European) | $1,900 |
| Compliance (Other) | $1,100 |
| Heat Treatment | $225 |
| EV Loading | $575 |

Note: Freight is not in the Variables panel — it's driven by the selected agent (Nichibo $2,072 / others $2,262).

### Conditional Logic
- **Brokerage, Freight, FAF, BAF & Marine Insurance GST**: Always applied automatically
- **European?** (Y/N toggle): Controls what options appear
- **Statement of Compliance?**: Only shown if European = Yes. Adds $440 (actual cost $390)
- **Compliance?**: If European + Yes → $1,900; if Not European + Yes → $1,100
- **Heat Treatment?**: If Yes → $225
- **EV Loading?**: If Yes → $575

## Variables Panel Features
- **"Variables" button** in top-right header opens/closes the panel
- Clean table layout showing Spot Buy Rate Adjustment + all 9 cost values (Brokerage, FAF, BAF, Marine Ins GST, SoC, Compliance Euro, Compliance Other, Heat Treatment, EV Loading)
- **Auto-saves** to browser localStorage on every edit (persists across sessions)
- **"Reset to Defaults"** button: restores values to saved defaults
- **"Save as New Defaults"** button: saves current values as new defaults
- Shows "Saved" / "New defaults saved" confirmation flash

## UI Layout (top to bottom)
1. **Header row**: "Jason's Calculator" title + Variables button
2. **Rate info**: Estimated Spot Buy rate and customer rate
3. **Direction toggle**: NZD to Yen / Yen to NZD (defaults to Yen to NZD)
4. **Input field**: $ or ¥ prefix, comma formatting as you type, ÷1000 for yen input
5. **Result boxes** (side by side): FOB (NZD Cost to Customer) + FOB (Yen)
   - Input currency box is dimmed/dashed
   - Output currency box is highlighted/scaled up
6. **Agent dropdown**: Nichibo, WEINS, NTP, Autobacs, Gulliver
7. **Car Cost box**: Car Cost in yen with agent fee subtitle
8. **Margin row**: Total margin in NZD (green text) + "Breakdown" button
   - Toggles collapsible panel showing all 3 margin parts with formulas
9. **Options toggles**: European, SoC, Compliance, Heat Treatment, EV Loading
10. **Additional Japan Costs** input (¥, ÷1000): visible in both directions, auto-fills with agent default
11. **Auto deductions**: Brokerage, Freight, FAF, BAF, and Marine Insurance GST (always on)
11. **Breakdown**: Full step-by-step calculation including Car Cost section
12. **Status**: Rate update timestamp

## Tech Stack
- Pure HTML / CSS / JavaScript (single file, ~990 lines)
- No frameworks, no dependencies
- API: `https://open.er-api.com/v6/latest/NZD` (free, no key required)
- Rate refreshes every 30 minutes, fallback to 90.0 if offline
- localStorage for persisting variable costs and custom defaults
- All NZD figures rounded to whole numbers (zero decimal places)
- Hosted on GitHub Pages for access from any device

## Design
- Dark theme: navy gradient background (#1a1a2e → #0f3460)
- Glassmorphism card with backdrop blur
- Accent blue (#5dade2) for NZD, red (#e74c3c) for Yen/Car Cost
- Green (#2ecc71) for toggle switches, auto labels, and margin
- Responsive, mobile-friendly (460px card width)

## Yen Rate
- Mid rate fetched live from `open.er-api.com`
- **Estimated Spot Buy rate** = API mid rate + Spot Buy Rate Adjustment (default -0.80, configurable in Variables)
- **Customer rate** = Estimated Spot Buy rate - 0.70 (margin hardcoded as YEN_MARGIN)
- Both rates displayed at top of app

## localStorage Keys
- `fobCalcCosts` - current variable cost values
- `fobCalcDefaults` - custom default values (set via "Save as New Defaults")

## Phase History
- **Phase 1** (2026-02-15): Simple NZD/JPY converter with toggle - REPLACED
- **Phase 2** (2026-02-15): Full calculation flow from spreadsheet - REPLACED
- **Phase 2b** (2026-02-15): Fixed GST to exclude Freight and Heat Treatment - REPLACED
- **Phase 2c** (2026-02-15): Fixed GST to exclude ALL fixed costs - KEY FIX
- **Phase 2d** (2026-02-15): Added bi-directional conversion with correct GST - KEY FEATURE
- **Phase 2e** (2026-02-15): Moved result boxes next to input, added input/output highlighting
- **Phase 2f** (2026-02-15): Renamed to FOB Calculator, upgraded Variables panel with localStorage
- **Phase 2g** (2026-02-15): Rounded all NZD figures to zero decimal places
- **Phase 3** (2026-02-15): Deployed to GitHub Pages
- **Phase 3b** (2026-02-15): Added custom Apple Touch Icon for iPhone home screen
- **Phase 3c** (2026-02-16): Added Marine Insurance GST ($110) as always-on auto deduction
- **Phase 3d** (2026-02-16): Added configurable Spot Buy Rate Adjustment variable (default -0.80)
- **Phase 3e** (2026-02-16): Fixed variables not updating on mobile
- **Phase 4a** (2026-02-22): Renamed to "Jason's Calculator", changed Yen Total to FOB
- **Phase 4b** (2026-02-22): Added Car Cost (FOB - agent fee) with agent dropdown and iterative solver for Nichibo
- **Phase 4c** (2026-02-22): Added Margin calculation (variable margins + agent margin + rate markup)
- **Phase 4d** (2026-02-22): Default to Yen to NZD on load, updated input labels (NZD Cost to Client / Yen FOB)
- **Phase 4e** (2026-02-22): Styled margin as matching agent-row
- **Phase 4f** (2026-02-22): Moved Options section above Deductions section
- **Phase 4g** (2026-02-23): Added Margin breakdown toggle button with detailed 3-part breakdown panel
- **Phase 5a** (2026-02-28): Updated SoC to $440/$390, agent-specific freight (Nichibo $2041/$1540, others $2262/$1850), yen input ÷1000, comma-formatted input field
- **Phase 5b** (2026-04-15): All agents now use the same 28-tier fee schedule; FAF/BAF/EV Loading added; Nichibo freight updated; DEFAULTS_VERSION bumped to 4
- **Phase 6** (2026-04-27): Reworked Yen→NZD so input is Car Cost (not FOB); FOB = Car Cost + Agent Fee + Additional Japan Costs; toggle swapped so Yen→NZD is on left and defaults; NZD→Yen left box shows Car Cost (yen); agent fee displayed as subtitle under Agent label; Additional Japan Costs field visible in both directions with agent defaults from spreadsheet table; FAF/BAF fixed as zero-margin pass-through; Agents fees spreadsheet updated so all 6 agents have full 28-tier table - CURRENT

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

## Key File Locations
- **Megan's actual Desktop**: `D:/Users/Megan/OneDrive/Desktop/` (OneDrive-synced)
- **Project folder**: `C:/ClaudeProjects/Currency Converter/`
- **Git repo**: `C:/ClaudeProjects/Currency Converter/.git/`
- **Note**: `C:/Users/Megan/Desktop/` is NOT the visible desktop
