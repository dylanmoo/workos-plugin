---
name: create-workstation
description: "Bootstraps a new workstation in a WorkOS-style workspace. Use whenever the user says 'create a workstation,' 'new workstation,' 'spin up a workstation,' 'add a workstation,' or asks to start a new client engagement, recurring workflow, or area of work that should have its own context. Creates the subfolder with CLAUDE.md and MEMORY.md following the standard structure, then registers the workstation in the root routing map so future sessions auto-load it."
---

# Starter Create Workstation

A repeatable scaffold for spinning up a new workstation in a WorkOS-style workspace. Run this when the user wants to add a new area of work and have it integrated into the root setup correctly the first time.

## What This Skill Does

Three things, and only three things:

1. **Gathers the minimum info** needed to create a meaningful workstation (name, identity, routing condition).
2. **Creates the workstation folder** with `CLAUDE.md` and `MEMORY.md` following the standard structure.
3. **Registers the workstation** in the root `CLAUDE.md` routing map so future sessions auto-load it.

That's it. No automated content migration, no inferred workflows, no pre-populated rules. Just the scaffold the user can grow.

## Step 1: Discover the Workspace Root

Find the WorkOS root dynamically. Look for a `CLAUDE.md` in the mounted workspace folder that has a "Routing Map" section. That's the root.

If the user is operating inside a nested WorkOS (e.g., they're already in a domain folder that itself has a `CLAUDE.md` with a routing map), ask which level the new workstation should live at.

If you can't find a root, stop and tell the user. Don't invent a structure.

## Step 2: Gather Workstation Info

Ask the user three questions, one at a time. Don't ask all three in one breath — it's easier to think clearly one at a time.

1. **What's the workstation name?** Will be the folder name. Use the user's exact phrasing — preserve capitalization, spacing, and any hyphens or punctuation. If the user says "name it whatever," propose something tight and confirm.
2. **One paragraph: who is Claude in this workstation?** This is the Identity. Examples:
   - "You are the user's email strategist and drafting partner."
   - "You are the user's partner for [client/engagement name] — context-specific framing if relevant."
   - "You are the user's scheduling and meeting-prep partner."
3. **When should future sessions route here?** This is the routing condition. Phrase as "...working on X" or "...need to do Y." Examples:
   - "...working on the [engagement name] engagement"
   - "...need to draft, reply to, or review any email"
   - "...preparing for or following up on a meeting"

If any answer is vague, propose a sharpened version and confirm before proceeding.

## Step 3: Create the Workstation Folder Structure

Create:

### `[Workstation Name]/CLAUDE.md`

With these sections in this exact order:

- **Identity** — the paragraph from Step 2, plus one line noting when sessions route here.
- **Resources** — a table with "Resource" and "Read when..." columns. Start empty unless the user already named files that belong here.
- **Workflow** — numbered steps for the primary task. Start with a single placeholder step: "TBD — refine on first use." Don't invent steps the user hasn't named.
- **Editorial Rules** — open with: "Follow voice principles in `[path to root voice file, typically ~/workOS/MEMORY.md or resources/voice-principles.md]`." Then add any domain-specific rules the user named in Step 2 or Step 3. If none, leave the section with the voice-principles line only.

### `[Workstation Name]/MEMORY.md`

With this structure:

- Header: `# [Workstation Name] Memory`
- A one-line intro: "Mutable state for the [name] [engagement/workflow/area]. Two-sentence-max entries. When something completes, move to the root ARCHIVE.md."
- **Active work** — placeholder line: "*(nothing active yet)*"
- **Contacts** — placeholder line: "*(none yet)*"
- **Key decisions** — placeholder line: "*(none yet)*"

### What NOT to create

- **No empty Resources subfolder.** Only create `[Workstation Name] Resources/` when a reference file actually exists. Empty subfolders rot.
- **No invented workflow steps.** If the user hasn't told you the workflow, leave it as TBD. Workflow is the thing they'll refine over time; pre-populating with guesses creates rules they didn't choose.
- **No `voice-principles.md` copy** inside the workstation. The workstation references the root voice file; it doesn't have its own.

## Step 4: Register in the Routing Map

Open the root `CLAUDE.md`. Find the Routing Map section (usually a Markdown table with "Workstation" and "Route here when..." columns).

Add a row:

```
| `[Workstation Name]/` | [routing condition from Step 2] |
```

If the table's column headers use different conventions, match the existing style — don't change the table structure.

## Step 5: Confirm

Show the user:

1. What was created (paths)
2. The routing-map row added to root `CLAUDE.md`
3. One next-step suggestion: usually, "Add a real resource file to your new workstation's Resources folder when one exists, or write your first workflow step the next time you work in this area."

If anything was ambiguous during creation, surface that and ask the user to clarify before declaring done.

## What This Skill Does NOT Do

- Doesn't extract content from existing files into the new workstation. If the user wants to migrate context from somewhere else, ask explicitly.
- Doesn't create project subfolders inside the workstation. Projects are a deeper layer — create them only when the user explicitly asks.
- Doesn't pre-fill Editorial Rules with domain-specific guesses. The user adds those as they emerge.
- Doesn't overwrite an existing workstation. If a folder with the proposed name already exists, stop and ask whether to use a different name or merge into the existing folder.
