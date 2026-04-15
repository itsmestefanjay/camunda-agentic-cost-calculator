# Camunda Agentic Orchestration Calculators

Browser-only tools for evaluating LLM costs and project ROI in Camunda agentic orchestration setups. Built for CTOs and strategic decision-makers.

No build step. No server. Open a page in a modern browser and start calculating.

---

## Pages

| Page | Purpose |
|---|---|
| `index.html` | Compare LLM model costs across process volumes — cost curves, break-even, forecasts, and strategic fit |
| `project.html` | Evaluate a specific automation project — implementation cost, payback period, NPV, and Go/Conditional/No-Go verdict |

---

## Features

### Cost Comparison (`index.html`)

- **Multi-model comparison** — add any number of LLM models with custom input/output token prices and fixed monthly costs
- **Agentic cost model** — accounts for history accumulation across tool calls: input tokens grow with each step in the chain (see [Cost formula](#cost-formula))
- **Four chart views** — cost vs. instances, forecast, break-even, and strategic score
- **Reference lines** — budget ceiling (red), manual cost baseline (purple), and current volume (green)
- **Intersection markers** — visual dots and diamonds where each model's cost curve crosses a reference line, with hover tooltips
- **Executive insights** — tab-specific plain-language analysis: cost leaders, annual burn range, break-even dates, ROI multiples, and fit-vs-cost tradeoffs
- **Scenarios** — save, load, and delete named configurations via localStorage
- **URL sharing** — encode full state into a `#` hash for shareable links
- **Export** — download chart as PNG or full data as CSV

### Project Business Case (`project.html`)

- **Implementation cost** — developer days × rate + infrastructure + training
- **Manual process baseline** — instances × handling time × hourly rate
- **Automation running costs** — LLM cost per instance + platform license + maintenance
- **Financial metrics** — payback period, 3-year net benefit, NPV at configurable discount rate
- **Verdict badge** — Go / Conditional / No-Go with rationale
- **Two chart views** — cumulative ROI over time and annual net savings
- **Executive insights** — payback date, NPV assessment, FTE hours freed

### Shared

- **Currency** — switch between USD and EUR with a configurable exchange rate
- **Dark / light theme**
- **Navigation** — burger menu links between calculators

---

## Running

```
open index.html
open project.html
```

No installation. No package manager. No build. Requires a modern browser (see [Compatibility](#browser-compatibility)).

---

## Architecture

Three files, all browser-native:

| File | Contents |
|---|---|
| `style.css` | Shared design tokens, theme variables, and reusable UI components |
| `index.html` | Cost comparison calculator — page layout, model panels, table, and all JS |
| `project.html` | Project business case calculator — page layout, result panels, and all JS |

External dependency: **Chart.js 4.x** loaded from CDN (both pages).

### Key design decisions

| Concern | Approach |
|---|---|
| Shared styles | `style.css` — design tokens, reusable components, nav drawer |
| State persistence (index) | `localStorage` (key: `camunda-cost-calc`) |
| Scenario storage | Separate `localStorage` key (`camunda-scenarios`) |
| URL sharing | `btoa`/`atob` JSON encoded in `location.hash` |
| Price storage | Slider positions (0–1000 integer), not raw currency values |
| Currency switching (index) | `convertPrices(factor)` physically moves slider positions |
| Currency switching (project) | `displayRate()` multiplier applied at display time; raw USD stored |
| State persistence (project) | `localStorage` (key: `camunda-project-calc`) |
| Chart | Chart.js scatter with `showLine: true` for cost curves |
| Recalculation | `recalculate()` is the single source of truth on each page |
| Navigation | Burger menu + `<nav class="nav-drawer">` on every page |

### Chart plugins

Five custom `afterDraw` plugins draw reference lines and markers on top of chart datasets:

| Plugin | Visual | Condition |
|---|---|---|
| `budgetLinePlugin` | Red dashed horizontal line | Budget > 0 |
| `manualCostLinePlugin` | Purple dashed horizontal line | Manual cost > 0 |
| `volumeMarkerPlugin` | Green dashed vertical line | Instances/mo > 0 |
| `budgetReachMarkerPlugin` | Per-model colored vertical lines to budget level | Budget > 0 |
| `breakevenMarkerPlugin` | Per-model colored vertical lines to manual cost level | Manual cost > 0 |

A custom `Chart.Interaction.modes.intersectionOnly` interaction mode ensures hover events only activate intersection marker datasets, preventing the chart from re-rendering curve datasets in a broken active state.

### Intersection datasets

Three additional Chart.js datasets (rendered above curves via `order: 0`) mark where each model's cost curve crosses a reference line:

| Dataset | Shape | Color | X position |
|---|---|---|---|
| Volume intersections | Circle | Green | `monthlyInstances` |
| Budget intersections | Circle | Red | `budgetReachInstances` |
| Break-even intersections | Diamond (`rectRot`) | Purple | `breakevenInstances` |

---

## Cost formula

The calculator models **agentic orchestration** specifically: in a multi-step tool chain, each call sends the full conversation history as input context. Input tokens therefore grow with every step.

For **N tool calls** (the *Tools / chain steps* parameter):

```
totalInputTokens  = N × inputTokens + N×(N-1)/2 × outputTokens
totalOutputTokens = N × outputTokens
costPerInstance   = totalInputTokens × inputPrice + totalOutputTokens × outputPrice
```

The `N×(N-1)/2` term is the triangular sum of accumulated history. Cost grows **quadratically** with chain length, which is why long agentic chains are disproportionately expensive.

Monthly cost:
```
monthlyCost = monthlyInstances × costPerInstance + fixedCost
```

---

## State shape

```js
processSettings: { inputTokens, outputTokens, tools }

models[]: {
  id,           // UUID
  color,        // hex
  name,         // display label
  inputPrice,   // slider position 0–1000
  outputPrice,  // slider position 0–1000
  fixedCost,    // raw currency value per month
  decisionScore // 0–100 strategic fit score
}

globals: {
  currency,         // 'USD' | 'EUR'
  eurRate,          // exchange rate
  budget,           // monthly budget ceiling
  monthlyInstances, // target process volume
  manualCost        // monthly cost of manual alternative
}
```

Prices are stored as **slider positions**, not raw token prices. `logToValue` / `valueToLog` convert between them. This means saved state is always in the active currency's slider space — no conversion happens on load.

---

## Model presets

12 pre-configured models with current pricing:

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|---|---|---|
| GPT-4.1 | $2.00 | $8.00 |
| GPT-4.1 mini | $0.40 | $1.60 |
| GPT-4.1 nano | $0.10 | $0.40 |
| GPT-4o | $2.50 | $10.00 |
| GPT-4o mini | $0.15 | $0.60 |
| Claude Opus 4 | $15.00 | $75.00 |
| Claude Sonnet 4 | $3.00 | $15.00 |
| Claude Haiku 3.5 | $0.80 | $4.00 |
| Gemini 2.5 Pro | $1.25 | $10.00 |
| Gemini 2.5 Flash | $0.15 | $0.60 |
| Llama 4 Maverick | $0.20 | $0.60 |
| Mistral Large | $2.00 | $6.00 |

Prices are approximate and subject to change. Update `MODEL_PRESETS` in the source to reflect current provider pricing.

---

## Browser compatibility

Requires modern evergreen browsers. The following features set the minimum bar:

| Feature | Minimum |
|---|---|
| CSS nesting | Chrome 120, Firefox 117, Safari 17.2 |
| `btoa` / `atob` | Universal |
| `localStorage` | Universal |
| ES2020 (`??`, `?.`) | Chrome 80, Firefox 74, Safari 13.1 |

Internet Explorer and legacy Edge are not supported.

---

## Keyboard / UX

- All sliders, inputs, and buttons are keyboard accessible
- Theme toggle preserves preference in saved state
- Collapsible panels use CSS `grid-template-rows` transitions (no JS height calculation)
- Chart tooltips only appear on intersection markers — not on every point along a curve — to reduce noise for executive users

---

## Extending

**Adding a model preset**: add an entry to `MODEL_PRESETS` with `name`, `inPrice` ($/token), and `outPrice` ($/token).

**Changing the cost formula**: update `recalculate()` and the formula documentation in `CLAUDE.md`. The formula affects the chart, summary cards, comparison table, break-even calculation, and CSV export — all driven from the same computed `costPerInstance`.

**Adding a chart tab**: implement a `buildXChart(theme)` function returning `{ config, plugins, helperMessages }`, register it in `renderActiveChart()`, add a tab button in the HTML, and handle it in `updateExecutiveInsight()`.
