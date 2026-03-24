---
description: Generate a client quote with cost breakdown from cut list and materials
allowed-tools: Read, Write, Edit, Glob, Bash(ls:*), Agent
argument-hint: [project-folder-name]
---

# Quote Generation

Generate a cost estimate and client quote for the project at `projects/$ARGUMENTS/`.

## Steps

1. **Verify prerequisites:**
   - Check that cut list exists at `projects/$ARGUMENTS/cut-lists/cut-list.md`
   - Check that shopping list exists at `projects/$ARGUMENTS/materials/shopping-list.md`
   - If missing, inform user to run `/cut-list` first

2. **Load data:**
   - Read `projects/$ARGUMENTS/cut-lists/cut-list.md`
   - Read `projects/$ARGUMENTS/materials/shopping-list.md`
   - Read `projects/$ARGUMENTS/README.md` for client info
   - Read `pricing/material-prices.md` for current prices
   - Read `pricing/labor-rates.md` for labor estimates
   - Read `pricing/markup-rules.md` for pricing rules

3. **Delegate to pricing-estimator agent** with all pricing data and project materials

4. **Save outputs:**
   - `projects/$ARGUMENTS/quotes/internal-estimate.md` — Full cost breakdown (not for client)
   - `projects/$ARGUMENTS/quotes/client-quote.md` — Professional client-facing quote

5. **Update project status** in README

6. **Present summary:** Total quote amount, margin, and any flagged items (missing prices, budget concerns)
