---
description: Create or update a FreeCAD 3D model from a finalized design
allowed-tools: Read, Write, Glob, Bash(ls:*), Agent, mcp__freecad__create_document, mcp__freecad__create_object, mcp__freecad__edit_object, mcp__freecad__delete_object, mcp__freecad__execute_code, mcp__freecad__get_view, mcp__freecad__insert_part_from_library, mcp__freecad__get_objects, mcp__freecad__get_object, mcp__freecad__get_parts_list, mcp__freecad__list_documents
argument-hint: [project-folder-name]
---

# FreeCAD 3D Modeling

Create a 3D model in FreeCAD for the project at `projects/$ARGUMENTS/`.

## Prerequisites
- FreeCAD must be running with the MCP workbench active and RPC server started
- A finalized design must exist at `projects/$ARGUMENTS/design/final/final-design.md`

## Steps

1. **Verify prerequisites:**
   - Check that the finalized design exists
   - Check that the cut list exists (for exact dimensions)
   - Test FreeCAD MCP connection with `list_documents`

2. **Load design and cut list:**
   - Read `projects/$ARGUMENTS/design/final/final-design.md`
   - Read `projects/$ARGUMENTS/cut-lists/cut-list.md`

3. **Delegate to freecad-modeler agent** with:
   - The finalized design
   - The cut list (for exact part dimensions)
   - Save paths for the project

4. **Capture views:**
   After the model is built, capture views from multiple angles:
   - Isometric (overall 3D view)
   - Front elevation
   - Top/plan view
   - Side profiles
   Save to `projects/$ARGUMENTS/models/exports/`

5. **Save the FreeCAD file:**
   Save to `projects/$ARGUMENTS/models/freecad/`

6. **Update project status** in README

7. **Show the user** a summary of what was created and where the files are
