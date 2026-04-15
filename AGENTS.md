# AGENTS.md

## Project identity

This repository contains a browser-only cost calculator for Camunda agentic orchestration scenarios.

Primary artifact:
- `index.html`

Primary audience:
- CTOs
- strategic decision-makers
- stakeholders evaluating automation ROI

The application must remain:
- single-file
- browser-native
- dependency-light
- understandable without a build pipeline

## Non-negotiable repository rules

- Keep the application as a single-file web app unless explicitly instructed otherwise.
- Use only HTML, CSS, and JavaScript.
- Do not introduce server-side code.
- Do not introduce Node.js, package managers, bundlers, transpilers, or build tooling.
- Do not add package manifests or lockfiles.
- The app must run directly in modern browsers without installation steps.
- Preserve compatibility with modern evergreen browsers appropriate for 2025 and later.
- Do not push to remote.
- Create a git commit for each logically separate change.
- Update `README.md` when behavior, architecture, compatibility, workflow, or dependencies materially change.

## Existing architecture constraints

Preserve and respect the current architecture unless a task explicitly requires changing it.

- The app is implemented as a single `index.html`.
- External dependency surface must remain minimal.
- Current dependency:
  - Chart.js 4.x from CDN
- State persistence uses:
  - `localStorage`
  - `btoa` / `atob` URL hash sharing
- `recalculate()` is the single source of truth for:
  - chart updates
  - comparison table updates
  - breakeven info
  - summary cards
  - state persistence side effects tied to recalculation flow
- Scenario storage and main state use separate localStorage keys and must remain clearly separated.
- Prices are stored internally as slider positions, not raw numeric currency values.
- Currency conversion currently works by moving slider positions, not by reinterpreting saved raw prices.
- Chart rendering includes custom reference-line plugins that must remain explicit and easy to reason about.

## Implementation priorities

Every change must optimize for:
- readability
- maintainability
- explicit behavior
- low cognitive load
- browser reliability
- minimal dependency surface

Prefer:
- simple control flow
- local reasoning
- small functions
- explicit DOM updates
- stable naming
- direct inspection over clever abstraction

Avoid:
- speculative frameworks
- premature modularization
- hidden side effects
- cross-cutting utility layers without clear payoff
- architecture expansion that exceeds the scale of a single-page calculator

## File policy

- Treat `index.html` as the canonical application artifact.
- Keep code sections clearly organized:
  - HTML structure
  - CSS
  - JavaScript
- Maintain strong internal separation within the file, even though it is single-file.
- Use section markers and consistent ordering so future edits remain tractable.
- If the file grows, improve structure inside the file before proposing a multi-file split.

## HTML rules

- Use semantic HTML wherever possible.
- Preserve a clear information hierarchy for decision-support use cases.
- Keep interactive controls properly labeled.
- Prefer native controls before custom UI constructs.
- Keep markup concise and intention-revealing.
- Do not introduce unnecessary wrapper elements.
- Ensure exported, shared, and persisted user flows remain understandable from the DOM structure.

## CSS rules

- Keep CSS readable and predictable.
- Preserve the current design language unless explicitly asked to redesign it.
- Maintain `var(--accent)` as the primary accent unless a design change is requested.
- Do not introduce fragile selector chains.
- Do not increase specificity unnecessarily.
- Keep stateful visual behavior easy to trace.
- Preserve compatibility assumptions around modern CSS features already in use.
- If using CSS nesting, do not deepen nesting to the point that selector intent becomes opaque.
- When changing layout or panels, verify collapsible behavior still works correctly.
- Remove unused styles immediately when markup changes.

## JavaScript rules

- Use plain modern JavaScript supported in target browsers.
- Never introduce TypeScript unless explicitly requested.
- Never introduce framework code.
- Prefer `const`, then `let`. Never use `var`.
- Keep functions focused and named by domain behavior.
- Keep state transformations explicit.
- Do not store the same truth in multiple mutable places without a clear reason.
- Preserve `recalculate()` as the central recomputation path unless a task explicitly requires changing that architecture.
- Any new UI control that affects computed output must flow through the same authoritative recalculation path.
- Avoid hidden coupling between DOM events and derived state.
- Keep chart generation, persistence, formatting, and business calculations clearly separable in code.

## State management rules

Respect the current state model and its conventions.

### Process settings
`processSettings: { inputTokens, outputTokens, tools }`

### Models
`models[]: { id, color, name, inputPrice, outputPrice, fixedCost }`

### Globals
`globals: currency, eurRate, budget, monthlyInstances, manualCost`

Rules:
- Do not change state shape casually.
- Do not introduce redundant derived state when it can be recomputed.
- Preserve the convention that model prices are stored as slider positions.
- Do not mix raw displayed currency values with internal slider-position representation.
- If state shape changes are necessary, update:
  - load logic
  - save logic
  - share logic
  - reset logic
  - migration handling if old state may exist
  - README if the change affects documented behavior

## Domain-specific calculation rules

The calculator is not a generic demo. It encodes business logic. Treat calculation behavior as product logic.

- Preserve the documented cost formula unless the task explicitly changes business rules.
- Preserve the distinction between:
  - per-instance cost
  - fixed cost
  - monthly total cost
  - manual cost comparison
  - budget reference line
  - breakeven information
- Do not change log slider mappings casually.
- Treat `logToValue` and `valueToLog` as critical conversion utilities.
- Any change to slider mapping, currency conversion, or token pricing must be validated against current behavior and documented if user-visible.

Current formula (agentic history accumulation):

In an agentic chain of N tool calls, each step receives the full conversation history as input, so input tokens grow with each call:
- `totalInputTokens  = N × inputTokens + N×(N-1)/2 × outputTokens`
- `totalOutputTokens = N × outputTokens`
- `costPerInstance   = totalInputTokens × inputPrice + totalOutputTokens × outputPrice`

Chart Y:
`instances * costPerInstance + fixedCost`

If these formulas change, update:
- UI text
- summary cards
- comparison table
- export output if affected
- README

## Currency and pricing rules

- Preserve the current pricing model unless explicitly changing product semantics.
- Prices are internally represented as slider positions.
- Currency switching currently moves slider positions using a conversion factor.
- Saved state reflects positions in the currently active currency.
- Do not silently replace this model with raw-value persistence without a deliberate migration plan.
- If changing currency behavior, verify:
  - switching USD/EUR
  - saving
  - loading
  - sharing
  - presets
  - export values
  - summary and chart consistency

## Chart rules

- Chart.js is allowed because it already exists and materially supports the product.
- Do not add another charting library.
- Keep chart configuration explicit and readable.
- Preserve multi-model comparison clarity.
- Preserve custom plugin behavior for all reference-line and marker plugins:
  - `budgetLinePlugin` — red dashed horizontal line at budget level
  - `manualCostLinePlugin` — purple dashed horizontal line at manual cost level
  - `volumeMarkerPlugin` — green dashed vertical line at monthly instances
  - `budgetReachMarkerPlugin` — per-model colored vertical lines from x-axis up to budget level
  - `breakevenMarkerPlugin` — per-model colored vertical lines from x-axis up to manual cost level
- A custom `Chart.Interaction.modes.intersectionOnly` is registered globally to restrict hover activation to intersection datasets only, preventing curve datasets from becoming active and causing rendering issues.
- Three intersection point datasets are layered on top of curves (`order: 0`):
  - Volume intersections — circles, green, at `x = monthlyInstances`
  - Budget intersections — circles, red, at `x = budgetReachInstances`
  - Break-even intersections — diamonds (`rectRot`), purple, at `x = breakevenInstances`
- Do not bury plugin logic inside unrelated utility code.
- Any chart behavior change must be checked against:
  - linear/log scale toggle
  - model visibility assumptions if present
  - exported PNG correctness
  - readability for executive users

## Export and sharing rules

- Preserve browser-native export behavior.
- Do not replace simple export/share flows with service-based or backend-based implementations.
- Maintain compatibility for:
  - PNG export
  - CSV export
  - URL share state
- If output format changes, ensure it remains understandable and useful for business review.

## localStorage and persistence rules

- Keep localStorage keys explicit and stable.
- Maintain clear separation between:
  - main app state
  - saved scenarios
- Avoid silent destructive migrations.
- If persistence structure changes, handle old stored data safely where feasible.
- Do not introduce persistence side effects in scattered places. Keep them tied to clear state transitions.

## Dependency policy

Default rule: no new dependencies.

- Do not add libraries unless they provide clear benefit that exceeds their complexity cost.
- Any proposed library must:
  - solve a concrete problem in this repo
  - work directly in browsers without package managers
  - support modern browsers in the project target range
  - integrate cleanly via CDN or standalone browser-friendly distribution
  - reduce net complexity

Before adding a library, state:
- why native browser APIs are insufficient
- what the library solves
- how it will be loaded
- why it fits the single-file architecture
- what risk it introduces

### Allowed current external dependency
- Chart.js 4.x via CDN

### Typical cases where a library may be justified
- high-quality CSV export edge cases
- accessibility-safe dialog/polyfill support if a real browser gap exists
- lightweight utility for date/number formatting only if native Intl is demonstrably insufficient

### Typical cases where a library is not justified
- DOM helpers
- state management
- routing
- styling frameworks
- general utility bundles
- build-only tooling

## Dead code policy

Remove dead code aggressively.

This includes:
- unused functions
- unused constants
- obsolete presets
- stale DOM references
- unreachable branches
- redundant formatting helpers
- commented-out code
- unused CSS blocks
- abandoned experimental UI code
- stale localStorage migration branches no longer needed

Do not leave code in place “just in case”.

## Readability rules

- Use domain names that match the product language:
  - model
  - scenario
  - budget
  - manual cost
  - breakeven
  - monthly instances
  - token pricing
- Avoid generic names like `data`, `handleThing`, `updateStuff`, `tmp`.
- Prefer explicit function names such as:
  - `updateSummaryCards`
  - `renderComparisonTable`
  - `applyCurrencyConversion`
  - `saveScenario`
- Keep event handlers thin.
- Push business logic into named functions, not anonymous inline blocks.
- Keep control flow shallow.

## Refactoring rules

Refactor only when it improves the touched area.

Good refactors:
- extracting repeated DOM update logic
- clarifying state conversion functions
- isolating chart config generation
- making panel toggle logic more understandable
- reducing repetition in preset creation

Bad refactors:
- introducing classes for a small stateful page without clear benefit
- converting everything into a homemade framework
- creating abstraction layers that obscure calculator logic
- splitting obvious domain logic into overly generic helpers

## Browser compatibility rules

Target modern browsers current in 2025+.

- Prefer stable, well-supported platform APIs.
- Avoid experimental APIs without fallback.
- Do not rely on transpilation.
- If a feature depends on a browser support floor, document it in README when relevant.
- Preserve current assumptions around modern CSS and JavaScript support.
- For any newly introduced API, verify broad support across current Chrome, Safari, Firefox, and Edge generations.

## Accessibility and UX rules

- Preserve keyboard accessibility of controls.
- Ensure labels, buttons, inputs, and toggles remain understandable.
- Keep executive-facing outputs legible and low-friction.
- Do not trade clarity for visual novelty.
- Any color-dependent chart or summary indicator should remain interpretable in context.
- Avoid hiding critical business interpretation behind hover-only interactions.

## README maintenance rules

Update `README.md` when relevant changes affect:
- browser compatibility
- architecture
- external libraries
- persistence behavior
- export/share behavior
- calculation semantics
- model preset coverage
- workflow or usage instructions
- known limitations

Do not update README for purely internal refactors with no externally meaningful effect.

## Git rules

- Create one git commit per logical change.
- Keep commits coherent and reviewable.
- Use precise commit messages.
- Do not combine unrelated cleanup with feature work unless the cleanup is directly required.
- Do not push.
- Do not rewrite history unless explicitly requested.

Suggested commit message patterns:
- `feat: add scenario duplication`
- `fix: correct eur slider conversion`
- `refactor: simplify chart dataset generation`
- `docs: update browser support notes`
- `chore: remove unused preset helpers`

## Required validation after changes

After every non-trivial change, verify as applicable:

- page loads without build steps
- no console errors
- sliders still behave correctly
- `recalculate()` updates all dependent UI
- chart renders correctly
- summary cards match table values
- breakeven output remains coherent
- budget and manual cost reference lines still align
- scenario save/load/delete still work
- share URL still round-trips state correctly
- reset still restores expected defaults
- PNG export still works
- CSV export still works
- localStorage behavior remains intact
- README is updated if behavior changed

## Execution protocol for repository changes

For each task:
1. Inspect affected code and related flows.
2. Identify whether the change is feature, fix, refactor, docs, or cleanup.
3. Implement the smallest correct solution.
4. Remove dead code exposed by the change.
5. Preserve single-file browser-only operation.
6. Avoid adding dependencies unless clearly justified.
7. Update README if relevant.
8. Create a focused git commit.
9. Do not push.

## Hard prohibitions

- No server code
- No backend integration
- No Node.js tooling
- No package manager files
- No bundlers
- No transpilers
- No TypeScript unless explicitly requested
- No framework migration
- No silent dependency additions
- No dead code retention
- No commented-out code blocks
- No pushing to remote