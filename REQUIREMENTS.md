# Agentic Decision Framework — Requirements

Browser-only decision tool for CTOs and strategic decision-makers evaluating agentic process automation using Camunda.
No server, no build step, no framework. HTML + CSS + JS only. Chart.js 4.x via CDN.

---

## Technical Constraints

- HTML, CSS, vanilla JS — no transpilers, no bundlers, no Node.js tooling
- Browser-native execution only
- Chart.js 4.x loaded via CDN (only allowed external dependency)
- JSZip 3.x via CDN (for chart ZIP export)
- State persisted in `localStorage`; share via URL hash (`btoa`/`atob`)
- All monetary values stored in USD internally; display converted via EUR rate
- No backend, no analytics, no external API calls

---

## File Structure

```
index.html      Landing page
step1.html      Business Case tool
step2.html      Process Prioritization tool
step3.html      Model Cost & Fit tool
summary.html    Consolidated summary, export, contact
about.html      Project background, legal disclaimer
style.css       Shared design tokens and components
assets/
  favicon.png
  logo-black.png          Consid logo (dark mode)
  logo-white.png          Consid logo (light mode)
  camunda-logo-black.png  Camunda logo (dark mode)
  camunda-logo-white.png  Camunda logo (light mode)
  partner-black.png       Camunda Silver Partner badge (dark mode)
  partner-white.png       Camunda Silver Partner badge (light mode)
```

---

## Story 1 — Shared Shell

**As a user, I want consistent navigation, theme, and currency controls on every page so I can move freely through the tool without losing context.**

### Navigation Drawer
- Hamburger button in header opens a left-side nav drawer (256px wide)
- Drawer contains: Home link, section header "Tools", links to steps 1–4, divider, About link, version number at bottom
- Each step link shows a numbered badge (1–4) before the label
- Active page is highlighted
- Version number shown subtly at the bottom of the drawer (`.nav-version`)
- Backdrop overlay (semi-transparent) covers the rest of the page
- Clicking backdrop or pressing Escape closes the drawer
- Drawer and backdrop animate open/close (0.24s)
- ARIA labels on all interactive controls

### Header
- Sticky at top (z-index 100), frosted-glass background (blur 14px)
- Left: hamburger + site name (links to index.html)
- Right: currency switcher, reset button, theme toggle
- Dark mode: `rgba(26,29,39,0.88)` bg; Light mode: `rgba(255,255,255,0.88)` bg

### Theme
- Toggle between dark and light mode
- Dark: bg `#0f1117`, surface `#1a1d27`, accent `#EC6B6A`
- Light: bg `#EFE6DD`, surface `#ffffff`, accent `#B5223F`
- Persisted in `localStorage['camunda-global']` as `{ theme, currency, eurRate }`
- Applied immediately on load via inline script before CSS paint (no flash)

### Currency
- USD / EUR toggle buttons (aria-pressed)
- EUR exchange rate input (default 0.85), dimmed when USD is active
- All monetary values stored in USD; multiplied by rate for display only
- Persisted globally; respected across all steps and summary

### Step Navigation Bar
- Shown on steps 1–4 and summary
- 4 items: Business Case · Prioritization · Model Cost & Fit · Summary
- Active step: bold, accent colour
- Left button: back to previous step (secondary style)
- Right button: forward to next step (primary style)
- Responsive: labels hide ≤520px

### Footer
- Centered: "Made by [Consid logo]" linked to consid.com
- Right-aligned: "© 2026 Consid AB" (absolutely positioned)
- Logo swaps between black (`assets/logo-black.png`) and white (`assets/logo-white.png`) based on theme
- Hidden on print (`no-print` class on all pages)

### Page Title
- Each tool page has `<h2>` with a numbered red circle badge (1–4) + title + subtitle paragraph
- Compact spacing: minimal gap between title and first content panel

### Panels
- Surface background, 1px border, 12px radius, 1.25rem padding
- Collapsible panels: chevron rotates −90° when collapsed, smooth grid-row-height animation

---

## Story 2 — Landing Page (`index.html`)

**As a CTO, I want a clear entry point that explains what the tool does and lets me start from any step.**

- Hero section: eyebrow "Agentic Decision Framework", headline "Size the opportunity / before you commit" (`clamp(2.25rem, 5vw, 3.5rem)`), subheadline, "About the project →" link on new line
- Four cards in a responsive grid (3-col → 2-col → 1-col): one per step (Business Case, Prioritization, Model Cost & Fit, Summary)
- Each card: large background number (decorative), title, one-line description, call-to-action link
- Hover on card: lifts 2px, accent-tinted border glow
- Summary card spans 2 columns
- Privacy note and disclaimer link at bottom

---

## Story 3 — Business Case (`step1.html`)

**As a CTO, I want to enter implementation cost and process volume, and see a payback period and net value across three risk scenarios.**

Storage key: `camunda-project-calc`

### Scope & Scale Panel (always visible)
- **Range profile** select: Small / Growth (default) / Enterprise
  - Adjusts min/max/step/default of all downstream sliders when changed
  - Profile ranges (Small / Growth / Enterprise):
    - Process count: 1–20 / 1–80 / 1–300
    - Instances/process/mo: 20–1000 / 0–5000 / 0–30000
    - Dev weeks: 1–36 / 1–100 / 2–300
    - Infra cost: $0–75k / $0–250k / $0–2M
    - Training: $0–30k / $0–120k / $0–800k
    - Camunda clusters: 1–3 / 1–6 / 1–20
    - Cluster yearly rate: $5k–10k / $5k–11k / $5k–12k
    - Existing allocated: $0–25k / $0–75k / $0–300k
    - Self-managed infra: $0–180k / $0–800k / $0–5M
  - Profile is auto-detected from loaded state volume
- **Automation context** select: Starting fresh (greenfield) / Adding to existing setup (incremental) / Replacing brittle automation (replace) / Exploring options (exploring)
  - Shows a contextual note below the select
- **Analysis horizon** range: 1–10 years, step 1, default 5

### Build Cost Panel (collapsible)
| Control | Range | Step | Default |
|---|---|---|---|
| Developers (FTE) | 0.5–20 | 0.5 | 1 |
| Developer weeks | 1–100 (profile) | 1 | 12 |
| Daily rate | $100–$3000 | $25 | $100 |
| Infrastructure & tooling | $0–profile max | $1000 | $5000 |
| Training & change mgmt | $0–profile max | $500 | $2000 |

Formula: `implementationCost = fte × devWeeks × 5 × dailyRate + infraCost + trainingCost`

### Manual Baseline Panel (collapsible)
| Control | Range | Step | Default |
|---|---|---|---|
| Candidate processes | 1–profile max | 1 | 12 |
| Instances/process/mo | 0–profile max | 50 | 0 |
| Avg people/instance | 0.5–5 | 0.1 | 1.2 |
| Handling hours/instance | 0.5–8h | 0.5h | 0.5h |
| Hourly rate | $10–$300 | $5 | $50 |
| Share automated at full scale | 10–100% | 5% | 80% |

Formula:
- `totalInstances = processCount × instancesPerProcess`
- `manualPerInstance = handlingHours × hourlyRate × peoplePerInstance`
- `monthlyManualCost = totalInstances × manualPerInstance`

### Advanced Ops Panel (collapsible, collapsed by default)
- **Deployment mode** select:
  - Camunda already in place (default) — platform cost = 0 (incremental) or allocated share
  - Add Camunda SaaS — shows cluster count + per-cluster yearly rate
  - Self-hosted Camunda — shows annual infra cost
  - Other orchestration stack — shows annual infra cost
  - Selecting a mode applies preset defaults for that mode
- **Existing platform cost mode** (Camunda already in place only): Incremental only / Include allocated share
- **Allocated platform cost/year** (visible when allocated mode): $0–profile max, step $250–$10k
- **Camunda SaaS clusters** (SaaS mode): 1–profile max, step 1, default 1–5
  - Note: "Estimation only. Camunda enterprise pricing is individual, not published, not affiliated."
- **Cost per cluster/year** (SaaS mode): $5k–12k, step $250, default $7500
  - Same disclaimer note
- **Self-managed infra/year** (self-hosted / other): $0–profile max
- **Orchestration variable cost/instance**: $0–$200, step $1, default $8
  - Note: "Estimation only. Camunda enterprise pricing is individual, not published, not affiliated."
- **LLM cost/automated instance** (number input): default $0.12
  - Note: "Model-dependent, additional to orchestration cost. Use cost/inst. from Step 3."
- **Human review rate**: 0–100%, step 1%, default 20%
- **Review time/reviewed instance**: 1–120 min, step 1, default 8
- **Sent back to humans (fallback rate)**: 0–60%, step 1%, default 8%
- **Quality assumptions confidence** select: Measured / Estimated (default) / Guessed
  - Affects conservative scenario: coverage, captured value, review rate, fallback rate
- **Maintenance days/month**: 0–20, step 1, default 3
- **Ops assumptions confidence** select: Measured / Estimated (default) / Guessed
  - Affects conservative scenario: cost multipliers, maintenance days

### Advanced Finance Panel (collapsible, collapsed by default)
- **Captured value %**: 0–100%, step 5%, default 60%
  - Percent of theoretical labor savings actually realized (accounts for redeployment friction)
- **Value-capture assumptions confidence**: Measured / Estimated (default) / Guessed
- **Pilot phase**: 0–12 months, step 1, default 2
- **Rollout ramp**: 1–24 months, step 1, default 6
- **Discount rate**: 0–25%, step 1%, default 8%

Rollout: pilot months at 25% coverage, then linear ramp to full coverage over ramp months.

### Scenario Spread (Confidence-Driven)
Three scenarios — Conservative, Base, Upside — derived from base inputs:

**Quality confidence** drives: coverage multiplier, captured value multiplier, added review rate, added fallback rate:
| Level | Coverage | Capture | +Review | +Fallback |
|---|---|---|---|---|
| Measured | ×0.92 | ×0.85 | +6pp | +5pp |
| Estimated | ×0.80 | ×0.70 | +12pp | +10pp |
| Guessed | ×0.65 | ×0.50 | +22pp | +18pp |

**Ops confidence** drives: cost multiplier, infra multiplier, added maintenance days:
| Level | Cost mult | Infra mult | +Maint days |
|---|---|---|---|
| Measured | ×1.10 | ×1.08 | +1 |
| Estimated | ×1.20 | ×1.15 | +2 |
| Guessed | ×1.45 | ×1.35 | +4 |

Upside uses fixed multipliers regardless of confidence (optimism doesn't scale with uncertainty).

### Results (Right Column)

**Decision badge**: Promising (green) / Needs Pilot (amber) / Weak (red)
- Promising: conservative NPV > 0, payback ≤ 36 months, conservative savings > 0
- Needs Pilot: base NPV > 0, base savings > 0
- Weak: otherwise

**Summary cards**:
1. Project investment (implementation cost, one-time)
2. Manual baseline / month (current monthly cost)
3. Realized labor benefit / month (savings × capture %, signed, green/red)
4. Steady-state net / month (labor benefit − run cost, signed)
5. Payback period (months, with conservative & upside in sub-label)
6. Conservative / Base / Upside monthly net (3-column mini-grid)
7. N-Year net value (NPV, signed, configured horizon)

**ROI Chart** (Chart.js line chart):
- Two tabs: Cumulative ROI | Annual Benefit by Year
- Three lines: Conservative (dashed red), Base (solid accent), Upside (dashed green)
- Break-even horizontal plugin line at y=0, labeled "Break-even"
- X-axis: months 0 to horizon; Y-axis: cumulative net in currency
- Annual tab: grouped bars by year, 3 datasets

**Handoff to Step 2**: writes `{ implCostPerWeek, processes[], createdAt }` to `camunda-s1s2-handoff`

**Share**: encodes full state as base64 URL hash
**Reset**: modal confirmation, restores all defaults

---

## Story 4 — Process Prioritization (`step2.html`)

**As a CTO, I want to enter candidate processes and see them ranked by combined ROI, feasibility, and agentic fit so I know which to automate first.**

Storage key: `camunda-process-calc`

### Handover Tile (from Step 1)
- Reads `camunda-s1s2-handoff` from localStorage, removes after load
- Shows: implementation cost/week, FTE × daily rate formula
- Slider-free: displayed as read-only tile

### Process Management
- Add process button opens an overlay preset menu with a "New Process" option and preset colors
- Each process is a collapsible panel with:
  - Collapse chevron
  - Color dot picker (hex input + swatch presets: raspberry, blue, green, amber, purple, pink, cyan, red, lime, orange)
  - Editable name field
  - Remove button
  - Collapsed summary: "N inst/mo · Score X · $/mo"

### Process Inputs (per process)

**Economics**
| Field | Type | Range | Default |
|---|---|---|---|
| Existing automation state | select | None / Partial (−40%) / BPM (−30%) | None |
| Monthly volume | range | 50–10000, step 50 | 500 |
| Manual hours/instance | range | 0.5–8h, step 0.5 | 0.5h |
| Hourly rate | range | $15–$300, step $5 | $60 |
| Monthly error cost | range | $0–50k, step $100 | $0 |
| Build effort (weeks) | range | 1–52, step 1 | 8 |
| Strategic priority | range | 1–5, step 1 | 3 |

**Feasibility**
- Integration readiness: High (80) / Medium (50, default) / Low (20)

**Agentic Fit**
- Decision structure: Deterministic rules (20) / Mixed (55, default) / Requires reasoning (85)
- Input data type: Structured (20) / Semi-structured (55, default) / Unstructured (85)
- Path variability: Fixed steps (20) / Branching (55, default) / Dynamic paths (85)

**Risk**
- Change frequency: Stable (0) / Quarterly (−8) / Frequent (−20)

### Scoring Formulas
```
monthlyManualCost = volume × manualHours × hourlyRate
incrementalFactor = 1 − automationOverlap  // 0 | 0.4 | 0.7
monthlySavings    = (monthlyManualCost + errorCost) × incrementalFactor
implCost          = effortWeeks × implCostPerWeek
payback           = implCost / monthlySavings
netBenefit3yr     = 36 × monthlySavings − implCost

savingsScore = min(100, log10(max(1, monthlySavings)) / log10(150000) × 100)
valueScore   = savingsScore × 0.7 + (strategicPriority − 1) × 25 × 0.3

feasibilityScore = readiness value (20 / 50 / 80)

agenticFitScore = (decisionScore + inputScore + pathScore) / 3

compositeScore = max(0,
  valueScore × 0.45 +
  feasibilityScore × 0.30 +
  agenticFitScore × 0.25 −
  volatilityPenalty
)
```

### Badge Logic
| Badge | Condition | Color |
|---|---|---|
| Traditional First | agenticFit < 40, value ≥ 50, feasibility ≥ 50 | Orange |
| Automate First | composite ≥ 55, value ≥ 50, feasibility ≥ 50 | Green |
| Strategic Bet | value ≥ 50, feasibility < 45 | Blue |
| Quick Win | feasibility ≥ 55, value < 45 | Teal |
| Pilot | composite ≥ 30 | Amber |
| Defer | otherwise | Gray |

Recommended for automation: Automate First, Strategic Bet, Quick Win.

### Summary Cards
1. **Worth pursuing** — count of recommended processes
2. **Monthly savings** — sum of monthlySavings for recommended only
3. **Total effort** — sum of effortWeeks for recommended only

### Charts
- **Value vs Feasibility Matrix** (scatter): 4 quadrants at 50/50. Labels: Strategic Bet (top-left), Automate First (top-right), Defer (bottom-left), Quick Win (bottom-right). Tooltip: process name + scores.
- **3-Year Net Benefit** (bar): sorted by composite score. Green bars positive, red bars negative.

### Priority Table
Columns: Rank · Process (colour dot) · Recommendation · Monthly savings · Payback · Score (sub-scores on hover)
Sorted by compositeScore descending. Striped rows.

### Handoff to Step 3
- Modal: select a process (radio buttons), pre-selects top-ranked
- Shows: volume, monthly manual cost, suggested budget (30% of savings), complexity (Simple/Medium/Complex)
- Writes `camunda-step2-handoff` to localStorage

### Share & Reset
- Hash encodes: `{ pr: processes[], ic: implCostPerWeek, cur, rate }`
- Reset: clears all processes, implCostPerWeek → default

---

## Story 5 — Model Cost & Fit (`step3.html`)

**As a CTO, I want to compare LLM models by cost at my process volume and see which fits my quality needs and budget.**

Storage key: `camunda-cost-calc`

Link in page subtitle: [Camunda guidance on choosing a model →](https://docs.camunda.io/docs/components/agentic-orchestration/choose-right-model-agentic/)

### Handover Tile (from Step 2, read-only)
- Monthly instances, monthly manual cost, suggested budget
- Notes explaining the source of each value

### Process Settings Panel (collapsible)
| Field | Range | Step | Default |
|---|---|---|---|
| Input tokens/tool call | 100–10000 | 100 | 100 |
| Output tokens/tool call | 50–5000 | 50 | 50 |
| Tool calls/process | 1–50 | 1 | 1 |

Token formula (accounts for context accumulation across tool calls):
```
totalInput  = tools × inputTokens + tools×(tools−1)/2 × outputTokens
totalOutput = tools × outputTokens
costPerInstance = totalInput × inputPrice + totalOutput × outputPrice
```

### Business Settings Panel (collapsible)
- Monthly instances: handoff or 0
- Monthly budget: 0 = no budget set
- Manual cost baseline: handoff or 0
- Monthly volume growth rate: 0–100%, step 1%, default 0%
- Forecast months: 1–60, step 1, default 12

### Model Presets
15 models available from the Add Model menu, grouped by vendor:

| Model | Input $/1M | Output $/1M |
|---|---|---|
| GPT-5 | $15.00 | $60.00 |
| GPT-4o | $2.50 | $10.00 |
| GPT-4o mini | $0.15 | $0.60 |
| Claude Opus 4 | $15.00 | $75.00 |
| Claude Sonnet 4 | $3.00 | $15.00 |
| Claude Haiku 3.5 | $0.80 | $4.00 |
| Gemini 2.5 Pro | $1.25 | $10.00 |
| Gemini 2.5 Flash | $0.15 | $0.60 |
| DeepSeek-V3 | $0.27 | $1.10 |
| DeepSeek-R1 | $0.55 | $2.19 |
| Grok-3 | $3.00 | $15.00 |
| Grok-3 mini | $0.30 | $0.50 |
| Qwen3-235B-A22B | $0.22 | $0.60 |
| Qwen3-72B | $0.40 | $1.20 |
| Qwen2.5-72B | $0.40 | $1.20 |
| Llama 4 Maverick | $0.20 | $0.60 |
| Llama 3.3 70B | $0.59 | $0.79 |

All models support tool calling and OpenAI-compatible APIs.
Custom model option: user enters name, input price, output price, fixed cost/mo, fit scores.

### Per-Model Configuration
Each added model row shows:
- Colour dot picker
- Model name
- Input price slider (log scale)
- Output price slider (log scale)
- Optional fixed monthly cost
- Fit score inputs: Compliance / Quality / Ops (each 0–100)

### Fit Score
`fitScore = (fitCompliance + fitQuality + fitOps) / 3`
- Compliance: cost vs budget (user-rated)
- Quality: reasoning capability for this use case (user-rated)
- Ops: operational reliability / API availability (user-rated)

### Charts (4 tabs)
1. **Cost vs Score** (scatter): X = fit score, Y = monthly cost. Budget line if set. Best value = high score, low cost.
2. **Cost vs Instances** (line): cost curves per model. X = 0 to 2× user volume. Reference lines: budget (horizontal), manual cost (horizontal), volume marker (vertical). Intersection markers. If manual cost is far above scale, shows off-chart label.
3. **Forecast** (bar/line): cost over forecastMonths at growth rate. Budget line.
4. **Break-even** (line): instances at which cost equals manual baseline per model.

Chart always destroys and recreates on recalculation (no caching) to keep reference lines fresh.

### Model Table
Columns: Model · Fit score · Cost/inst. · Monthly · vs Manual · % of budget · Break-even

- **% of budget**: `monthlyCost / budget × 100`. Colour-coded: green < 50%, amber 50–100%, red > 100%
- **Break-even**: months at current volume; capped display at "60+" if large; "—" if no manual cost set
- Sortable columns (click header)
- Striped rows
- Rows removed via × button on each row

### Share & Reset
- Hash encodes compact state: `{ m, ps, cur, rate, bgt, mi, gr, fm, mc, ct, pp, bp }`
- Reset: clears all models, resets settings to defaults

---

## Story 6 — Summary (`summary.html`)

**As a CTO, I want a consolidated view of all three assessments with one-click export so I can present findings to stakeholders.**

### Executive Summary
- Investment signal badge (Promising / Needs Pilot / Weak) with prose explanation
- Key metrics: payback period, N-year net value (from step 1)
- Top-ranked process name + composite score + monthly savings potential (from step 2)
- Best-fit model name + monthly cost + fit score (from step 3)
- Narrative paragraph combining all three

### Divergence Banner
- Shown if step 1 manual cost baseline and step 2 total monthly manual cost diverge by > 5%
- Amber warning with explanation and guidance

### Section 1 — Business Case
- Metrics: payback, NPV, 3-year net benefit
- Scenario table: Conservative / Base / Upside rows, each showing monthly savings, run cost, net/mo, payback, NPV
  - Conservative row: amber tint; Base row: accent tint (bold); Upside row: green tint
- Cumulative net benefit chart (line, same as step 1)

### Section 2 — Process Prioritization
- Metrics: total processes, recommended count, combined monthly savings potential
- Process table: rank · name · badge · composite score · monthly savings · payback · viable (score ≥ 50)
- Composite score bar chart

### Section 3 — Model Cost & Fit
- Metrics: model count, instances/mo, budget
- Model table: model · fit score · cost/inst. · monthly · vs manual · % budget · break-even
- Within-budget indicators

### Inputs & Assumptions Section
- Grid of input cards (2 columns), one card per group:
  - Build (Step 1): FTE, weeks, daily rate, infra, training, total implementation cost
  - Manual baseline (Step 1): processes, instances, people, handling hours, hourly rate, context
  - Automation assumptions (Step 1): deployment, coverage, capture %, review rate, fallback, LLM cost, maintenance
  - Financial assumptions (Step 1): pilot, ramp, discount rate, horizon
  - Processes (Step 2): impl cost/week, process count, recommended count, total volume, total build effort
  - Scenario (Step 3): instances, budget, manual cost, token profile, model count

### Share & Export Bar
- Prominent bar with accent-tinted background and "Share & export" label
- **Copy Link** (primary button, SVG icon): encodes full state as base64 URL hash, copies to clipboard
- **Export PDF** (secondary button, SVG icon): triggers print dialog with print stylesheet applied
- **Download Charts** (secondary button, SVG icon): exports all rendered canvases as PNGs in a ZIP file
- **Download JSON** (secondary button, SVG icon): exports full assessment as structured JSON

### JSON Export Format
LLM-optimised structure:
```json
{
  "_meta": {
    "tool": "Agentic Decision Framework",
    "about": "...who built it and why...",
    "why": "...three core questions answered...",
    "purpose": "...suitable for further analysis...",
    "howToInterpret": {
      "monthlySavings": "...",
      "npv": "...",
      "paybackMonths": "...",
      "compositeScore": "...",
      "fitScore": "...",
      "scenarios": "...",
      "badges_process": "...",
      "badges_investment": "..."
    }
  },
  "businessCase": {
    "_description": "...",
    "investmentSignal": { "value": "promising|needs-pilot|weak", "description": "..." },
    "automationContext": { "value": "...", "description": "..." },
    "deploymentMode": { "value": "...", "description": "..." },
    "confidenceLevel": { "quality": "...", "ops": "..." },
    "inputs": {
      "developers_fte": { "value": 2, "unit": "FTE", "description": "..." },
      "implementationCost": { "value": 50000, "unit": "EUR", "description": "..." }
      // ... all inputs with value + unit + description
    },
    "results": {
      "monthlySteadyStateSavings": { "value": 12000, "unit": "EUR/month" },
      "paybackMonths": { "value": 8, "unit": "months" },
      "npv": { "value": 240000, "unit": "EUR" },
      "scenarios": [
        { "name": "Conservative", "_description": "...", "paybackMonths": 14, "npv": 180000, "monthlySavings": 9000 },
        { "name": "Base", "_description": "...", ... },
        { "name": "Upside", "_description": "...", ... }
      ]
    }
  },
  "processPrioritization": {
    "_description": "...",
    "implCostPerWeek": { "value": 5000, "unit": "EUR/week" },
    "summary": { "totalProcesses": 5, "recommendedForAutomation": 3 },
    "processes": [{
      "name": "Invoice Processing",
      "recommendation": { "badge": "Automate First", "compositeScore": 72 },
      "scores": { "value": 80, "feasibility": 50, "agenticFit": 85 },
      "economics": {
        "monthlyVolume": { "value": 1200, "unit": "instances/month" },
        "monthlyManualCost": { "value": 18000, "unit": "EUR/month" },
        "monthlyPotentialSavings": { "value": 15000, "unit": "EUR/month" },
        "paybackMonths": { "value": 4, "unit": "months" }
      }
    }]
  },
  "modelCostFit": {
    "_description": "...",
    "scenario": { "monthlyInstances": 1200, "monthlyBudget": 4500, "manualCostBaseline": 18000 },
    "tokenProfile": {
      "inputTokensPerRun": { "value": 500, "unit": "tokens" },
      "outputTokensPerRun": { "value": 200, "unit": "tokens" },
      "toolCallsPerInstance": { "value": 3, "unit": "calls" }
    },
    "bestFitModel": { "name": "Claude Haiku 3.5", "fitScore": 78, "monthlyCost": 1200 },
    "models": [{
      "name": "Claude Haiku 3.5",
      "monthlyCost": { "value": 1200, "unit": "EUR/month" },
      "costPerInstance": { "value": 0.001, "unit": "EUR" },
      "withinBudget": true,
      "budgetUsagePct": 26,
      "fitScore": { "value": 78, "description": "..." },
      "fitBreakdown": { "compliance": 90, "quality": 65, "ops": 80 }
    }]
  }
}
```

### Consid Contact Section
- Pitch paragraph + service bullets
- Contact form: Name, Company, Email, Message
- Submit opens mailto with pre-filled subject and body
- Note: "Opens your email client"

### About Page (`about.html`)
- Project description and four-step workflow summary
- Consid section: company intro, Camunda Silver Partner badge (`assets/partner-black.png` / `assets/partner-white.png`, centered), partnership text, link to consid.com
- Camunda section: Camunda logo (`assets/camunda-logo-black.png` / `assets/camunda-logo-white.png`), company description
- Legal disclaimer box: estimates only, not financial advice, not affiliated with Camunda, built independently by Consid, AI-assisted build disclosure

---

## Story 7 — State & Persistence

**As a user, I want my inputs to survive page refresh and be shareable via URL.**

### localStorage Keys
| Key | Used by | Content |
|---|---|---|
| `camunda-global` | all pages | `{ theme, currency, eurRate }` |
| `camunda-project-calc` | step1 | full step 1 state |
| `camunda-process-calc` | step2 | `{ processes[], implCostPerWeek }` |
| `camunda-cost-calc` | step3 | full step 3 state |
| `camunda-s1s2-handoff` | step1→step2 | impl cost/week + process list |
| `camunda-step2-handoff` | step2→step3 | volume, cost, budget, token defaults |

### URL Hash Sharing
- Each tool encodes its full state as `btoa(JSON.stringify(state))` into the URL hash
- On load: parse hash first; fall back to localStorage; fall back to defaults
- Summary page shares all three steps' states combined

### Migration
- `devDays` → `devWeeks` (÷5)
- `handlingMinutes` → `handlingHours` (÷60, round to 0.5)
- `manualMinutes` → `manualHours` (÷60, round to 0.5)
- Legacy `platformCost` → `platformFixedYearly` (×12)

---

## Story 8 — Accessibility & Responsiveness

**As a user on any device, I want the tool to be readable and operable.**

- All sliders have visible `<output>` labels showing current value with unit
- All form controls have `<label>` elements
- Keyboard navigation: all interactive elements focusable, focus rings visible
- ARIA labels on icon-only buttons
- Escape key closes nav drawer
- Responsive breakpoints: 860px (two-col → one-col), 640px (compact grid), 520px (hide step labels)
- Print stylesheet: hides nav, header controls, action buttons; white background; manages page breaks

---

## Story 9 — Design System

**As a developer, I want consistent visual language across all pages.**

### Colours (CSS custom properties)
| Token | Dark | Light |
|---|---|---|
| `--bg` | `#0f1117` | `#EFE6DD` |
| `--surface` | `#1a1d27` | `#ffffff` |
| `--accent` | `#EC6B6A` | `#B5223F` |
| `--text` | `#e8e8ec` | `#1C1C1C` |
| `--muted` | `#8b8d97` | `#525862` |
| `--accent-rgb` | `236,107,106` | `181,34,63` |

Semantic colours (fixed across themes): positive `#22c55e`, negative `#ef4444`, warning `#F59E0B`.

### Typography
- System UI font stack (`system-ui, -apple-system, sans-serif`)
- Panel titles: 0.75rem, uppercase, 600 weight, letter-spacing 0.04em
- Body: 0.875rem
- Metrics/values: tabular-nums

### Components
All defined in `style.css`:
- `.panel` — surface card
- `.settings-panel` — collapsible panel with chevron
- `.slider-group` — label + range/select + output + note
- `.summary-card` — metric card with value, label, sub-label
- `.decision-badge` — coloured pill badge (promising/needs-pilot/weak/empty)
- `.proc-badge` — small coloured process recommendation badge
- `.action-btn` / `.icon-btn` — button variants
- `.nav-drawer` / `.nav-backdrop` — navigation overlay
- `.nav-version` — version label pinned to bottom of nav drawer
- `.site-footer` — footer with centered logo and right-aligned copyright
- `.step-nav-btn` — previous/next navigation buttons
- `.step-bar` — step progress bar

---

## Non-Functional Requirements

- No console errors on load
- No network requests except CDN resources (Chart.js, JSZip)
- All calculations run client-side synchronously
- State round-trips correctly: save → reload → same values
- Share URL round-trips correctly: encode → new tab → same view
- Chart renders on first load without interaction
- Print layout: no interactive controls visible, charts render in print resolution
- All pages link to each other correctly; no broken hrefs
