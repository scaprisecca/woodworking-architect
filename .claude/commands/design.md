---
description: Enter design mode - analyze intake data and create layout options for a project
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(ls:*), Agent
argument-hint: [project-folder-name]
---

# Design Mode

Create layout options for the project at `projects/$ARGUMENTS/`.

## Steps

1. **Load intake data:**
   - Read `projects/$ARGUMENTS/intake/measurements.md`
   - Read `projects/$ARGUMENTS/intake/client-notes.md`
   - List contents of `projects/$ARGUMENTS/intake/photos/`
   - Read `projects/$ARGUMENTS/README.md` for project overview

2. **Validate intake completeness:**
   - Verify that measurements are populated (not just template placeholders)
   - Check that client preferences are documented
   - If critical data is missing, list what's needed and ask before proceeding

3. **Delegate to space-design-expert agent** with:
   - All intake data
   - The project type and client preferences
   - Any constraints from the README

4. **Save results:**
   - Layout options go to `projects/$ARGUMENTS/design/layout-options/`
   - Update `projects/$ARGUMENTS/README.md` status to "Design"

5. **Present options** to the user with a summary of each

## After Design Approval
When the client selects an option, copy it to `projects/$ARGUMENTS/design/final/final-design.md` and update the README status to "Design Approved."
