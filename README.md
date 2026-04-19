# Agentic Orchestration Calculators

A set of browser-based decision tools for CTOs and other stakeholders evaluating whether to adopt agentic AI orchestration with Camunda. The goal is a defensible, directional answer in minutes — not a replacement for a full business case, but a concrete starting point that makes the conversation grounded.

## What it does

The tools walk through three questions in sequence:

**1. Is it worth doing?** (`step1.html`)
Enter your process details, implementation costs, and rollout assumptions. The tool calculates payback period, 3-year net benefit, and NPV across a conservative, base, and upside range. You get a go/no-go signal based on whether the numbers hold up under pessimistic conditions.

**2. Which processes, in what order?** (`step2.html`)
If you have multiple candidates, this helps you prioritize them. Each process gets scored on value (how much labor and error cost it displaces) and feasibility (data readiness, process stability, compliance exposure). The result is a ranked list and a 2×2 chart that separates quick wins from strategic bets.

**3. Which model fits the budget?** (`step3.html`)
Compare LLM cost curves across models at your expected call volume. Set a budget line and see where each model crosses it. Includes break-even analysis, forecast charts, and cost-per-score comparisons. Results can be exported as PNG or CSV, or shared via URL.

## How to use it

Open `index.html` in a modern browser. No installation, no server, no sign-in.

```
open index.html
```

Each step saves your inputs automatically. You can share a step's state via URL, reset to defaults, or export results at any point.

## What it is not

These tools produce estimates, not guarantees. The inputs are yours — the models are simplified and make assumptions that may not match your situation. Always validate material decisions with your own rigorous analysis.

See [about.html](about.html) for the full legal disclaimer.

## Author

Stefan Schultz — Camunda Champion. Independent project, not affiliated with or endorsed by Camunda Services GmbH.
