# Woodworking Architect

## About This Project
Operational hub for a custom cabinetry and woodworking business. Manages client projects from initial consultation through final delivery: space design, 3D modeling in FreeCAD, cut list generation, material estimation, pricing/quoting, and professional client-facing deliverables.

## Owner Profile
- One-person custom cabinetry shop
- Project types: kitchens, bathrooms, closets, entertainment centers, bookcases, shelving, general woodworking
- Measures on-site, photographs spaces, transcribes client conversations
- FreeCAD beginner — agents handle all FreeCAD complexity

## Shop Tools & Constraints
All designs MUST be buildable with these tools:
- **Job-site table saw** — limited rip capacity vs cabinet saw
- **10" miter saw** — max crosscut ~6" at 90°
- **7.25" circular saw** + **Kreg Accu-Cut** — primary method for ripping 4x8 sheet goods
- **Hand router** — no router table
- **Jigsaw, multi-tool**
- **Kreg pocket hole jig** — primary joinery method
- **NO CNC, NO wide-belt sander, NO edge bander machine**

### Cut Constraints
- Kerf allowance: 1/8" (0.125") for all saw cuts
- Edge banding: iron-on or peel-and-stick, applied by hand
- Sheet goods: 4'x8' (48" x 96") standard
- All cuts must be achievable manually — no designs requiring extreme precision tolerances
- Grain direction matters for visible hardwood faces

## Measurement Conventions
- Use inches and feet-inches: e.g., 34-1/2", 8'-0"
- Fractions preferred over decimals for dimensions under 1"
- Always specify thickness as actual, not nominal

## Cabinet Standards (Starting Points)
- Base cabinets: 34-1/2"H (36" with countertop), 24"D, widths 9"-48"
- Wall cabinets: 30"/36"/42"H, 12"D, widths 9"-48"
- Tall/pantry cabinets: 84"/90"/96"H
- Toe kick: 4-1/2"H x 3"D
- 18" between countertop and bottom of wall cabinets
- These are starting points — adjust to space and client needs

## Project Structure
- Client projects: `projects/{client-name}-{type}-{YYYY-MM}/`
- Project template: `projects/_template/` — copy for new projects
- Reference materials: `reference/`
- Project-type templates: `templates/`
- Pricing data: `pricing/`

## Agents
Sub-agents in `.claude/agents/` handle specialized tasks:
- **consultation-prep** — Pre-visit checklists per project type
- **space-design-expert** — Layout design, ergonomics, storage optimization
- **cut-list-materials** — Cut lists, material optimization, sheet nesting
- **freecad-modeler** — 3D modeling via FreeCAD MCP
- **pricing-estimator** — Material costs, labor estimates, client quotes

## FreeCAD Integration
- FreeCAD controlled via MCP server (freecad-mcp)
- MCP tools: create_document, create_object, edit_object, delete_object, execute_code, get_view, insert_part_from_library, get_objects, get_object, get_parts_list, list_documents
- For PDF/STEP/STL exports: use execute_code with FreeCAD Python API
- Save FreeCAD files to project's `models/freecad/` directory
- Save exported views to project's `models/exports/` directory

## Workflow
Typical project flow:
1. `/new-project` — Create project folder, capture client info
2. `/consult-prep` — Generate site visit checklist
3. On-site: measure, photograph, discuss with client
4. Enter intake data (measurements, notes, transcripts, reference images)
5. `/design` — Generate layout options
6. Client approves a design
7. `/cut-list` — Generate cut list and shopping list
8. `/freecad-model` — Create 3D model with views
9. `/quote` — Generate pricing
10. `/present` — Compile client presentation

## Design Notes
- Every design should include a "construction sequence" section suggesting build order
- Always account for obstacles: plumbing, electrical, windows, doors, HVAC
- Note which elements are standard sizes vs custom-required
- Include 10-15% waste factor for sheet goods in material estimates
