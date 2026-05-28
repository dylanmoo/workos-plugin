---
name: create-project
description: "Bootstraps a new project in a WorkOS-style workspace. Use whenever the user says 'create a project,' 'new project,' 'spin up a project,' 'add a project,' 'start a new client,' 'add an engagement,' or asks to start a new client engagement, recurring workflow, or area of work that should have its own context. Creates the subfolder with CLAUDE.md and MEMORY.md following the standard project structure, then registers the project in the appropriate routing map (root or domain) so future sessions auto-load it. The successor to /create-workstation in v1.x — same skill, simpler name."
---

# Create Project

A repeatable scaffold for spinning up a new project in a WorkOS-style workspace. Run this when the user wants to add a new area of work and have it integrated into the routing correctly the first time.

## What this skill does

Three things, and only three things:

1. **Gathers the minimum info** needed to create a meaningful project (name, identity, routing condition).
2. **Creates the project folder** with `CLAUDE.md` and `MEMORY.md` following the standard structure.
3. **Registers the project** in the right routing map (root if it lives at the workspace top level, the parent domain's `CLAUDE.md` if it lives inside a domain) so future sessions auto-load it.

That's it. No automated content migration, no inferred workflows, no pre-populated rules. Just the scaffold the user can grow.

## Step 1: Discover the workspace root and parent

Find the WorkOS root dynamically. Look for a `CLAUDE.md` in the mounted workspace folder that has a "Routing Map" section. That's the root.

Then determine where the new project should live:

- If the user is in the workspace root and the project doesn't naturally fit any existing domain, the project sits at the root level.
- If the user is inside a domain folder (one with its own `CLAUDE.md`), the project will be created inside that domain.
- If the user wants the project inside a specific group (e.g., `clients/`), confirm the group folder exists and the project will live at `<domain>/<group>/<project>/`.

Ask the user explicitly if the placement is ambiguous. Don't guess.

If you can't find a workspace root, stop and tell the user. Don't invent a structure.

## Step 2: Gather project info

Ask the user three questions, one at a time. Don't ask all three in one breath — it's easier to think clearly one at a time.

1. **What's the project name?** Will be the folder name. Use the user's exact phrasing — preserve capitalization, spacing, and any hyphens or punctuation. If the user says "name it whatever," propose something tight and confirm.
2. **One paragraph: who is Claude in this project?** This is the Identity. Examples:
   - "You are the user's email strategist and drafting partner."
   - "You are the user's partner for [client/engagement name] — context-specific framing if relevant."
   - "You are the user's scheduling and meeting-prep partner."
3. **When should future sessions route here?** This is the routing condition. Phrase as "...working on X" or "...need to do Y." Examples:
   - "...working on the [engagement name] engagement"
   - "...need to draft, reply to, or review any email"
   - "...preparing for or following up on a meeting"

If any answer is vague, propose a sharpened version and confirm before proceeding.

## Step 3: Create the project folder structure

Create:

### `[Project Name]/CLAUDE.md`

With these sections in this exact order:

- **Identity** — the paragraph from Step 2, plus one line noting when sessions route here.
- **Resources** — a table with "Resource" and "Read when..." columns. Start empty unless the user already named files that belong here.
- **Workflow** — numbered steps for the primary task. Start with a single placeholder step: "TBD — refine on first use." Don't invent steps the user hasn't named.
- **Editorial Rules** — open with: "Follow voice principles in `[path to root voice file, typically ~/workOS/MEMORY.md or resources/voice-principles.md]`." Then add any project-specific rules the user named in Step 2 or Step 3. If none, leave the section with the voice-principles line only.

### `[Project Name]/MEMORY.md`

With this structure:

- Header: `# [Project Name] Memory`
- A one-line intro: "Mutable state for the [name] [engagement/workflow/area]. Two-sentence-max entries. When something completes, move to the parent domain's `ARCHIVE.md` (or root `ARCHIVE.md` if the project is closed out entirely)."
- **Active work** — placeholder line: "*(nothing active yet)*"
- **Contacts** — placeholder line: "*(none yet)*"
- **Key decisions** — placeholder line: "*(none yet)*"

### What NOT to create

- **No empty `resources/` subfolder.** Only create one when a real reference file actually exists. Empty subfolders rot.
- **No invented workflow steps.** If the user hasn't told you the workflow, leave it as TBD. Workflow is the thing they'll refine over time; pre-populating with guesses creates rules they didn't choose.

## Step 4: Register in the right routing map

If the project lives at the workspace root, open the root `CLAUDE.md` and find the Routing Map section.

If the project lives inside a domain, open `<domain>/CLAUDE.md` and find its Routing Map section. (Create the Routing Map section in the domain `CLAUDE.md` if it doesn't exist yet — this is the first project being added there.)

Add a row using the path that's correct for the scope (relative to the file you're editing):

```
| `[Project Path]/` | [routing condition from Step 2] |
```

Match existing style — don't change table column headers or structure.

## Step 5: Confirm

Show the user:

1. What was created (paths).
2. The routing-map row added, and which `CLAUDE.md` it landed in.
3. One next-step suggestion: usually, "Add a real resource file to your new project's `resources/` folder when one exists, or write your first workflow step the next time you work in this area."

If anything was ambiguous during creation, surface that and ask the user to clarify before declaring done.

## What this skill does NOT do

- **Doesn't extract content from existing files into the new project.** If the user wants to migrate context from somewhere else, ask explicitly.
- **Doesn't pre-fill Editorial Rules with project-specific guesses.** The user adds those as they emerge.
- **Doesn't overwrite an existing project.** If a folder with the proposed name already exists, stop and ask whether to use a different name or merge into the existing folder.
- **Doesn't create sub-projects.** Projects are leaf nodes in WorkOS. If the user wants more structure inside a project, that's the project's internal organization to manage — not another scaffold layer.
