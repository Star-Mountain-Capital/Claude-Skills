---
name: picture-generation
description: Generate professional images, charts, diagrams, and visuals for Star Mountain Capital use cases — including fund performance charts, portfolio composition graphics, org charts, process flow diagrams, deal flow maps, and LP-facing presentation visuals. When active, Claude asks clarifying questions about format, audience, and brand requirements before generating. All outputs reflect institutional standards appropriate for LP-facing or leadership materials.
---

# Picture Generation — Star Mountain Capital

You help the SMC team create professional visuals — charts, diagrams, org charts, process flows, and presentation graphics — that meet institutional standards for LP communications, IC presentations, and internal documentation.

## When to Use This Skill

- Fund performance charts (IRR over time, MOIC distribution, DPI/RVPI waterfall)
- Portfolio composition visuals (sector breakdown, geography heatmap, vintage distribution)
- Deal flow diagrams (pipeline funnel, deal sourcing breakdown)
- Org charts (fund structure, team hierarchy, portfolio company management)
- Process flow diagrams (close process, capital call workflow, diligence stages)
- LP update presentation graphics
- Investment structure diagrams (fund → holdco → portfolio company stack)
- Executive dashboards and KPI scorecards
- Timeline charts (fund life, investment period, portfolio milestones)

## Before Generating — Clarification Checklist

Ask the user for the following before producing any visual. If they've already specified, proceed directly.

1. **Visual type** — What kind of chart or diagram? (bar chart, pie chart, org chart, flow diagram, waterfall, timeline, etc.)
2. **Audience** — Who will see this? (LPs, IC, board, internal team, public/marketing)
3. **Data available** — Do you have the underlying numbers or is this a template/mockup?
4. **Format** — How will this be used? (PowerPoint slide, Word document, PDF report, Notion page, standalone image)
5. **Brand requirements** — Should it follow SMC's color palette and design standards?
6. **Approximate size/complexity** — Simple single chart or multi-panel dashboard?

## SMC Brand Standards

When generating visuals for LP-facing or official firm materials:

**Color palette (primary):**
- SMC Navy: `#1B2A4A` (headers, primary bars, dominant elements)
- SMC Gold: `#C9A84C` (highlights, accents, secondary emphasis)
- White: `#FFFFFF` (backgrounds, labels on dark fields)
- Light Gray: `#F5F5F5` (alternate row backgrounds, secondary panels)

**Color palette (data visualization — sequential):**
- Primary data series: `#1B2A4A`
- Secondary series: `#C9A84C`
- Tertiary series: `#6B8CAE`
- Neutral / comparison: `#9E9E9E`

**Typography:**
- Titles: Bold, clean sans-serif
- Body / labels: Regular weight, same family
- Numbers: Right-aligned in tables; centered in chart labels

**Visual standards:**
- No 3D effects (flat design only)
- Minimal gridlines — horizontal only for bar/line charts
- All axes labeled; all units specified (%, $M, x)
- Source and "As of [date]" included on all LP-facing charts
- "Unaudited" label where applicable

## Visual Templates by Type

### Fund Performance Summary

```
Chart type: Grouped bar or line chart
X-axis: Quarterly periods (Q1 2024 – Q2 2026)
Y-axis: Net IRR (%) or NAV ($M)
Series: Fund III, Fund IV, Credit Fund (if multi-fund)
Labels: Period-end values on data points
Footer: "As of [date]. Net of fees and expenses. Unaudited."
```

### Portfolio Composition (Sector Breakdown)

```
Chart type: Horizontal bar chart (preferred over pie for institutional)
Categories: Sector names (Business Services, Healthcare, Industrial, etc.)
Values: % of fair value or # of portfolio companies
Sort: Descending by value
Labels: % and $ values shown
Color coding: One SMC color per sector (use sequential palette)
```

### Fund Structure Diagram

```
Diagram type: Top-down org chart / hierarchy
Levels:
  1. SMC [Fund Name] LP
  2. General Partner entity + Limited Partner pool
  3. Fund vehicle (LP entity)
  4. Portfolio Company A | Portfolio Company B | ...
Annotations: Ownership %, entity type
Style: Clean boxes with thin borders, SMC Navy for headers
```

### Investment Sourcing Funnel

```
Chart type: Funnel or waterfall (left to right)
Stages: Reviewed → Screened → Diligence → IC → Closed
Values: Deal count at each stage
Color: SMC Navy (lightest at top, darkest at closed)
Conversion rates: % shown between stages
```

### Capital Call / Distribution Timeline

```
Chart type: Timeline / Gantt-style
X-axis: Date (by quarter or year)
Events: Capital calls (down arrows, negative), distributions (up arrows, positive)
Cumulative line: Running DPI and called capital %
Labels: $ amounts on each event
```

### Deal Process Flow

```
Diagram type: Left-to-right flow (horizontal swimlane or simple boxes)
Stages: Origination → Screen → LOI → Diligence → IC → Sign → Close
Actors per stage: Deal team / IC / Legal / FinOps
Decision points: Diamond shapes at IC and final approval
Color: Stage boxes in SMC Navy, actor labels in light gray
```

## Generating the Visual

Claude can produce visuals in two ways depending on what tools are active:

### 1. As a structured description / spec (for Canva, PowerPoint, or design tool)

When Claude cannot directly render images, produce a precise visual specification that a team member or designer can execute:

```
VISUAL SPEC: [Chart name]

Type: [Chart type]
Dimensions: [Slide size or pixel dimensions]
Background: [Color]

DATA:
[Table of data to plot]

STYLING:
- Primary color: SMC Navy #1B2A4A
- Accent: SMC Gold #C9A84C
- Font: [Calibri / Arial]
- Grid: Horizontal lines only

LABELS:
- Title: "[Title text]"
- Subtitle: "[Subtitle if applicable]"
- X-axis: "[Label]"
- Y-axis: "[Label] ([unit])"
- Footer: "As of [date]. [Data source]. [Unaudited if applicable]"

NOTES FOR DESIGNER:
[Any specific instructions]
```

### 2. As Python code (for Claude Code users)

For data-driven charts that can be generated programmatically:

```python
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import numpy as np

# SMC color palette
SMC_NAVY = '#1B2A4A'
SMC_GOLD = '#C9A84C'
SMC_BLUE = '#6B8CAE'

def plot_fund_performance(periods: list, nav_values: list, fund_name: str, output_path: str):
    """Generate a clean NAV trend chart for LP reporting."""
    fig, ax = plt.subplots(figsize=(10, 5))
    fig.patch.set_facecolor('white')
    
    ax.plot(periods, nav_values, color=SMC_NAVY, linewidth=2.5, marker='o', markersize=5)
    ax.fill_between(periods, nav_values, alpha=0.1, color=SMC_NAVY)
    
    # Minimal styling
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)
    ax.yaxis.grid(True, linestyle='--', alpha=0.4, color='#CCCCCC')
    ax.set_axisbelow(True)
    
    ax.set_title(f'{fund_name} — NAV Over Time', fontsize=14, fontweight='bold', 
                 color=SMC_NAVY, pad=12)
    ax.set_xlabel('Period', color=SMC_NAVY)
    ax.set_ylabel('NAV ($M)', color=SMC_NAVY)
    
    # Add current value label
    ax.annotate(f'${nav_values[-1]:.1f}M', xy=(periods[-1], nav_values[-1]),
                xytext=(5, 5), textcoords='offset points',
                fontsize=10, color=SMC_NAVY, fontweight='bold')
    
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches='tight', facecolor='white')
    plt.close()
    return output_path
```

## Output Checklist Before Delivering

Before presenting any visual or visual spec to the user, confirm:
- [ ] Data is correctly labeled and matches the input provided
- [ ] Chart type is appropriate for the data (avoid pie charts for >5 categories)
- [ ] Units are specified on all axes
- [ ] "As of" date is included for LP-facing outputs
- [ ] "Unaudited" label present if applicable
- [ ] Color palette follows SMC standards (if LP-facing or IC-facing)
- [ ] No PII or non-public portfolio company data in the visual

## Example Requests

```
Create a sector breakdown chart for our portfolio as of Q2 2026. We have:
Business Services: 35%, Healthcare: 25%, Industrial: 20%, Government: 15%, Consumer: 5%.
This is for our LP quarterly update deck.
```

```
I need a fund structure diagram showing SMC Fund IV LP → the GP entity → the fund vehicle → our 8 portfolio companies. Clean, institutional.
```

```
Generate the Python code to create a bar chart showing deal flow by quarter from Q1 2024 through Q2 2026. I'll paste the data.
```
