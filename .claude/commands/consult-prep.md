---
description: Generate a consultation preparation checklist tailored to a specific project type
allowed-tools: Read, Write, Glob, Bash(ls:*), Agent
argument-hint: [project-type]
---

# Consultation Prep

Generate a comprehensive pre-consultation checklist for a **$ARGUMENTS** project.

## Steps

1. Identify the project type from the argument: `$ARGUMENTS`
   - Valid types: kitchen, bathroom, closet, entertainment-center, bookcase, shelving, general-woodworking
2. Read the corresponding template from `templates/$ARGUMENTS.md` if it exists
3. Delegate to the **consultation-prep** agent with the project type and any template content
4. Save the generated checklist

## Output Location
- If there's an active project context, save to that project's `intake/consultation-checklist.md`
- Otherwise, display the checklist directly for the user to copy

The checklist should be formatted with `[ ]` checkboxes so it can be printed and used as a field worksheet.
