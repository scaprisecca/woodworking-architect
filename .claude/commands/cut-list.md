---
description: Generate optimized cut list and materials shopping list from a finalized design
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(ls:*), Agent
argument-hint: [project-folder-name]
---

# Cut List Generation

Generate a complete cut list, sheet nesting, and shopping list for the project at `projects/$ARGUMENTS/`.

## Steps

1. **Verify design is finalized:**
   - Check that `projects/$ARGUMENTS/design/final/final-design.md` exists
   - If not, inform the user they need to finalize a design first (run `/design`)

2. **Load the design:**
   - Read `projects/$ARGUMENTS/design/final/final-design.md`
   - Read `reference/tools-inventory.md` for tool constraints

3. **Delegate to cut-list-materials agent** with:
   - The finalized design
   - Tool inventory and constraints
   - Material preferences from the design

4. **Save outputs:**
   - `projects/$ARGUMENTS/cut-lists/cut-list.md` — Part-by-part cut list
   - `projects/$ARGUMENTS/cut-lists/nesting.md` — Sheet nesting diagrams
   - `projects/$ARGUMENTS/materials/shopping-list.md` — Complete shopping list
   - `projects/$ARGUMENTS/cut-lists/cutting-sequence.md` — Recommended cutting order

5. **Update project status** in README to "Cut List Complete"

6. **Present summary:** Total sheets needed, estimated waste percentage, total hardware count, any flagged issues
