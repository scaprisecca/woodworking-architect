---
description: Create a new client project with standard folder structure and intake forms
allowed-tools: Read, Write, Edit, Bash(mkdir:*), Bash(cp -r:*), Bash(ls:*), Glob
argument-hint: [client-name] [project-type]
---

# New Project Setup

Create a new client project. Gather the following information either from the arguments or by asking:

1. **Client name** (last name or business name)
2. **Project type** (kitchen, bathroom, closet, entertainment-center, bookcase, shelving, or general-woodworking)
3. **Brief description** of what the client wants
4. **Client address** (for the job site)
5. **Client contact info** (phone, email — optional at this stage)
6. **Budget range** (if discussed)
7. **Any initial notes** from first contact

## Steps

1. Create the project folder at `projects/{client-last-name}-{project-type}-{YYYY-MM}/` using the current date
2. Copy the contents of `projects/_template/` into the new project folder
3. Update `README.md` with all gathered information, setting status to "Intake"
4. Update `intake/client-notes.md` with the description, preferences, and any initial notes
5. If the user provides a conversation transcript, append it to `intake/client-notes.md`

## After Creation

Print a summary:
```
Project created: projects/{folder-name}/

Next steps:
  1. Run /consult-prep {project-type} to generate your site visit checklist
  2. Schedule the on-site consultation
  3. After the visit, add measurements to intake/measurements.md
  4. Add photos to intake/photos/
  5. Run /design {folder-name} to start the design phase
```

Use `$ARGUMENTS` for any provided arguments. First argument is client name, second is project type.
