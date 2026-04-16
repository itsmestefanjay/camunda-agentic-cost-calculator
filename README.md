# Camunda Agentic Orchestration Calculators

Browser-only tools to evaluate whether agentic AI orchestration makes economic sense.

No build step. No server. Open in a modern browser.

## Pages

| Page | Use |
|---|---|
| `index.html` | Landing page — overview and entry point |
| `step1.html` | Step 1: Project business case / ROI estimate |
| `step2.html` | Step 2: Process prioritization — which processes, in what order |
| `step3.html` | Step 3: Model cost & fit comparison |
| `about.html` | Project background, author, legal disclaimer |

## Run

```bash
open index.html
```

## Stack

- HTML, CSS, JavaScript
- Chart.js 4.x via CDN
- `style.css` for shared UI
- `localStorage` for persistence
- URL hash sharing in `step3.html`

## Cost Model (Step 3)

```text
totalInputTokens  = N * inputTokens + N * (N - 1) / 2 * outputTokens
totalOutputTokens = N * outputTokens
costPerInstance   = totalInputTokens * inputPrice + totalOutputTokens * outputPrice
monthlyCost       = monthlyInstances * costPerInstance + fixedCost
```

## Scoring Model (Step 2)

```text
monthlyManualCost  = volume × (manualMinutes / 60) × hourlyRate
monthlyErrorCost   = volume × (errorRate / 100) × errorCost
monthlySavings     = monthlyManualCost + monthlyErrorCost

savingsScore       = min(100, log10(max(1, monthlySavings)) / log10(150000) × 100)
valueScore         = savingsScore × (strategicPriority / 5)

rawFeasibility     = (dataAvailability + processStability + (6 − complianceRisk)) / 3
feasibilityScore   = ((rawFeasibility − 1) / 4) × 100

compositeScore     = valueScore × 0.6 + feasibilityScore × 0.4
```

Badges: `Automate First` / `Strategic Bet` / `Quick Win` / `Pilot` / `Defer`

## Key Behaviors

- `step1.html`
  - implementation + run-cost estimate
  - rollout ramp and confidence inputs
  - conservative / base / upside framing
  - payback, 3-year net benefit, NPV, sensitivity drivers
- `step2.html`
  - per-process sliders: volume, labor, error rate, effort, feasibility factors
  - composite score from value (60%) + feasibility (40%)
  - priority table ranked by composite score
  - value vs feasibility 2×2 scatter chart
  - 3-year net benefit bar chart in ranked order
  - localStorage persistence (`camunda-process-calc`)
- `step3.html`
  - multi-model comparison
  - budget, manual-cost, and volume reference lines
  - cost vs instances, forecast, break-even, and cost-vs-score views
  - PNG export, CSV export, shareable state, saved scenarios

## Notes

- `step3.html` stores model prices as slider positions, not raw token prices.
- `step1.html` stores money in USD and converts for display.
- `step2.html` stores process data in raw field values.
- If formulas or persistence change, update `README.md` and `CLAUDE.md`.

## Browser Support

Requires a modern evergreen browser. No Internet Explorer. No legacy Edge.
