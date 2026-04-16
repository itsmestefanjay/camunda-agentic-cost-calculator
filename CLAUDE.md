# CLAUDE.md

## Scope

- Repo: browser-only calculators for Camunda agentic orchestration decisions
- Audience: CTOs, strategic decision-makers, ROI stakeholders
- Files:
  - `index.html` - landing page
  - `step1.html` - model cost comparison (was `index.html`)
  - `step2.html` - business-case / ROI estimate (was `project.html`)
  - `about.html` - project background, author, legal disclaimer
  - `style.css` - shared tokens and reusable UI

## Non-Negotiables

- HTML, CSS, JS only
- No server code
- No backend integration
- No Node.js tooling
- No package manifests
- No bundlers
- No transpilers
- No frameworks
- No TypeScript unless explicitly requested
- No silent dependency additions
- No commented-out code blocks
- No dead-code retention
- Browser-native execution only
- No build step

## File Boundaries

- `style.css`
  - shared tokens
  - shared components
- `index.html`
  - landing page structure
  - page-local CSS
  - page-local JS (theme + nav only)
- `step1.html`
  - cost-comparison page structure
  - page-local CSS
  - page-local JS
- `step2.html`
  - business-case page structure
  - page-local CSS
  - page-local JS
- `about.html`
  - about page structure
  - page-local CSS
  - page-local JS (theme + nav only)

## Shared UI In `style.css`

- `.panel`
- `.settings-panel`
- `.slider-group`
- `.action-btn`
- `.icon-btn`
- `.executive-insight`
- `.summary-card`
- `.decision-badge`
- `.chart-container`
- `.chart-tabs`
- `.chart-helper`
- `.nav-drawer`
- `.nav-backdrop`

## Architecture

### `index.html`

- Landing page only — no calculator logic
- Theme and nav drawer JS only

### `step1.html`

- Chart.js 4.x via CDN
- Main state key: `camunda-cost-calc`
- Scenario key: `camunda-scenarios`
- Share state: URL hash via `btoa` / `atob`
- `recalculate()` = source of truth
- Prices stored as slider positions
- Currency switch via `convertPrices(factor)`
- Custom Chart.js plugins are product-critical
- `Chart.Interaction.modes.intersectionOnly` is product-critical

### `step2.html`

- Chart.js 4.x via CDN
- Main state key: `camunda-project-calc`
- `compute()` + `recalculate()` pattern
- Money stored in USD
- Display conversion via `displayRate()`

### Navigation

- Burger menu on every page
- Active nav item uses `active`
- Backdrop click closes drawer
- `Escape` closes drawer

## Implementation Priorities

- readability
- maintainability
- explicit behavior
- low cognitive load
- browser reliability
- minimal dependency surface

## Preferred Style

- small focused functions
- explicit DOM updates
- clear naming
- simple control flow
- local reasoning
- thin event handlers
- section markers: `/* ===== Section ===== */`

## Avoid

- speculative architecture
- hidden side effects
- generic utility layers without payoff
- premature modularization
- framework-like abstractions

## HTML Rules

- semantic HTML where practical
- clear information hierarchy
- native controls first
- explicit labels

## CSS Rules

- preserve existing design language unless redesign is requested
- low specificity
- readable selectors
- no fragile selector chains
- preserve collapse behavior
- preserve responsive behavior
- remove unused CSS immediately

## JS Rules

- plain modern JavaScript
- `const`, then `let`
- never `var`
- business logic in named functions
- no duplicated derived state without reason
- new output-affecting controls must flow through authoritative recalculation
- keep chart config, persistence, formatting, and business logic separable

## State Rules

- do not change state shape casually
- do not store redundant derived state without reason
- preserve slider-position storage for model pricing in `index.html`
- if state shape changes, update:
  - save/load
  - share logic
  - reset logic
  - migration handling
  - README if user-visible

## Persistence Rules

- keep keys explicit
- keep keys stable
- separate main app state from scenarios
- avoid silent destructive migrations

## Product Logic

- treat calculation logic as product behavior
- preserve distinctions between:
  - per-instance cost
  - fixed cost
  - monthly total cost
  - manual cost comparison
  - budget reference lines
  - break-even information

### Cost Formula

- `totalInputTokens = N * inputTokens + N * (N - 1) / 2 * outputTokens`
- `totalOutputTokens = N * outputTokens`
- `costPerInstance = totalInputTokens * inputPrice + totalOutputTokens * outputPrice`
- chart Y = `instances * costPerInstance + fixedCost`

### If Formula Changes

- update UI text
- update cards
- update insights
- update table
- update exports
- update README

## Currency Rules

- preserve current currency model unless intentionally redesigning it
- `index.html`: slider-position based pricing
- `project.html`: USD state + display conversion only
- if currency behavior changes, verify:
  - USD/EUR switch
  - save/load
  - share links
  - presets
  - exports
  - chart/card consistency

## Chart Rules

- no additional chart library without explicit justification
- keep config explicit
- keep config readable
- preserve executive readability
- do not bury plugin logic inside unrelated utilities

### Product-Critical Chart Behaviors

- budget reference line
- manual cost reference line
- volume marker
- budget reach markers
- break-even markers
- intersection-only hover behavior

### Chart Change Checks

- current tabs / views
- exports
- persisted state
- readability for non-technical decision-makers

## Export / Share Rules

- preserve PNG export
- preserve CSV export
- preserve shareable URL state
- no backend or service-based replacement

## Dependency Policy

- default: no new dependencies
- allowed current dependency: Chart.js 4.x via CDN
- any new dependency must:
  - solve a concrete browser-side problem
  - work without package managers
  - load cleanly via CDN or standalone browser build
  - reduce net complexity

## Naming / Readability

- prefer product-language names:
  - model
  - scenario
  - budget
  - manual cost
  - break-even
  - monthly instances
  - token pricing
- avoid vague names:
  - `data`
  - `tmp`
  - `updateStuff`

## Refactoring

### Good

- extracting repeated DOM updates
- isolating chart config builders
- simplifying panel-toggle logic
- reducing repetition in scenario or preset handling

### Bad

- classes for small pages
- homemade framework layers
- generic helpers that obscure domain logic
- architecture expansion without payoff

## Dead Code

- remove unused functions
- remove stale constants
- remove stale DOM refs
- remove obsolete migrations
- remove unused CSS
- remove abandoned experimental UI

## Browser / A11y / UX

- target modern evergreen browsers
- prefer stable platform APIs
- no transpilation assumptions
- preserve keyboard usability
- keep labels and summaries clear
- do not hide critical interpretation behind hover only
- do not trade clarity for novelty

## README

- update `README.md` when changes affect:
  - behavior
  - architecture
  - compatibility
  - persistence
  - exports / sharing
  - calculation semantics
  - workflow
  - dependencies
  - limitations

## Validation

- page loads directly in browser
- no console errors
- sliders behave correctly
- `recalculate()` updates dependent UI
- charts render correctly
- cards and table stay consistent
- break-even logic stays coherent
- reference lines align
- scenarios still save/load/delete
- share URL round-trips state
- reset restores defaults
- PNG export works
- CSV export works
- persistence remains intact
- README updated when required

## Git

- one commit per logical change
- coherent commits
- precise commit messages
- no unrelated work in one commit
- no history rewrite unless explicitly requested
- no direct work on `main`
- no direct push to `main`
- no force-push

### Branching

- feature work on `feature/<name>`

### Good Commit Messages

- `feat: add scenario duplication`
- `fix: correct eur slider conversion`
- `refactor: simplify chart dataset generation`
- `docs: update browser support notes`

## Execution Protocol

1. Inspect affected code and flows
2. Classify work: feature, fix, refactor, docs, cleanup
3. Implement the smallest correct solution
4. Remove dead code exposed by the change
5. Preserve browser-only operation
6. Avoid dependencies unless clearly justified
7. Update `README.md` if externally meaningful
