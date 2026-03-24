---
description: Compile client-facing presentation materials from design, renders, and quote
allowed-tools: Read, Write, Edit, Glob, Bash(ls:*), Agent, mcp__freecad__execute_code, mcp__freecad__get_view
argument-hint: [project-folder-name]
---

# Client Presentation

Compile all deliverables into a professional client-facing presentation for the project at `projects/$ARGUMENTS/`.

## Steps

1. **Gather all assets:**
   - Read `projects/$ARGUMENTS/README.md` for project overview
   - Read `projects/$ARGUMENTS/design/final/final-design.md`
   - List `projects/$ARGUMENTS/models/exports/` for rendered views
   - Read `projects/$ARGUMENTS/quotes/client-quote.md`

2. **Generate additional views if needed:**
   - If FreeCAD is available and the model exists, capture any missing angles
   - Generate detail views of specific features if the design warrants it

3. **Compile the presentation** at `projects/$ARGUMENTS/deliverables/client-presentation.md`:

   ### Document Structure:
   - **Cover:** Project title, client name, date, your business name
   - **Project Summary:** Scope of work, what's being built
   - **Design Overview:** Description with references to rendered views
   - **Materials & Finishes:** What materials are being used and why
   - **3D Views:** Reference all rendered images with captions
     - Overall view
     - Front elevation
     - Detail views of key features
   - **Features & Storage Solutions:** Highlight storage optimization, special hardware
   - **Quote Summary:** From client-quote.md (line items, total, payment terms)
   - **Timeline:** Estimated project duration with milestones
   - **Terms & Conditions:** Standard terms, warranty, change order policy
   - **Next Steps:** What the client needs to do to proceed (approve, deposit, etc.)

4. **Save to:** `projects/$ARGUMENTS/deliverables/client-presentation.md`

5. **Update project status** in README

6. **Notify the user** what was created and suggest reviewing before sharing with the client
