# WorkOS Plugin

A Claude Code plugin with five skills for running a Cowork-style markdown workspace. Designed to compound context over time and keep `CLAUDE.md` / `MEMORY.md` lean.

## Install

In Claude Code:

```
/plugin marketplace add dylanmoo/workos-plugin
/plugin install workos
```

That's it. The five skills below become available as slash commands.

## Skills

- **`/init-workos`** — One-time bootstrap. Recursively scans an existing directory (root → domain → workstation, depth 2) and lays the WorkOS scaffold on top: root `CLAUDE.md` with routing map, `MEMORY.md`, `ARCHIVE.md`, plus per-domain and per-workstation `CLAUDE.md` + `MEMORY.md` where appropriate. Tags every file it creates with a `workos-init-scaffold` sentinel. Never moves existing files.
- **`/uninit-workos`** — Reverse of init. Finds files by the `workos-init-scaffold` sentinel, removes clean scaffold files automatically, and asks file-by-file what to do with edited ones. Never deletes any folder or any file without the sentinel.
- **`/create-workstation`** — Bootstraps a new workstation (folder + `CLAUDE.md` + `MEMORY.md`) and registers it in the active workspace's routing map. Asks for name, identity, and routing condition.
- **`/session-audit`** — End-of-session capture. Scans the current conversation for corrections, preferences, decisions, and new context, then proposes the exact wording and target file for each.
- **`/workspace-audit`** — Periodic file-level health check. Five checks: wrong-location entries, archive candidates, size-ceiling status, entry format violations, and engagement-specific content that should cascade to a workstation. Never writes without approval.

## When to use each

| Skill | Cadence |
|---|---|
| `/init-workos` | Once per workspace, on the first session in a new directory |
| `/uninit-workos` | Only when you want to revert init-workos and get back to your pre-WorkOS structure |
| `/session-audit` | End of every working session |
| `/create-workstation` | When starting a new client engagement, recurring workflow, or area of work that deserves its own folder |
| `/workspace-audit` | Weekly first month, monthly after — or any time output quality slips |

## How to compound learnings over time

WorkOS gets sharper the more you use it. Every correction, decision, and fact has a home that future sessions automatically inherit. The loop:

1. **First session** — run `/init-workos` in your project directory. The scaffold lands on top of your existing files; nothing moves or gets renamed.
2. **Every session** — the agent auto-reads `MEMORY.md` at the relevant scope. Say "remember this" mid-session to file a fact. End the session with "audit this session" to run `/session-audit`, which scans the conversation for corrections and decisions you didn't think to save and proposes where each belongs.
3. **When work grows** — run `/create-workstation` whenever a new client, project, or recurring area emerges. Three questions, one new routable folder with its own `CLAUDE.md` and `MEMORY.md`.
4. **Every few weeks** — run `/workspace-audit`. Flags misplaced entries, oversized files, completed projects to archive, and content that should cascade to a more specific scope.

### Where things go

| What you have | Where it lives |
|---|---|
| A rule (`"always do X"`, `"never do Y"`) | `CLAUDE.md` — most-specific scope wins (workstation > domain > root) |
| A mutable fact (contact, status, decision) | `MEMORY.md` — same scoping |
| Durable knowledge true on a 6-month horizon | `{domain}/resources/<topic>.md` |
| Source material (transcripts, articles, raw notes) | `{domain}/raw/` |
| Completed projects, aged-out facts | `ARCHIVE.md` — write-only, never read at session start |

### The compounding effect

Corrections given once become rules. Facts mentioned once stay recalled. Lessons distilled from experience accumulate in `resources/`. The agent grows into your specific working style instead of resetting every session. Most of this happens automatically — you just need to actually run `/session-audit` at the end and `/workspace-audit` on a cadence.

## Update

When a new version ships:

```
/plugin update workos
```

Claude Code also checks for updates periodically on its own.

## Layout

```
workos-plugin/
├── .claude-plugin/
│   ├── marketplace.json   ← lets you `/plugin marketplace add` this repo
│   └── plugin.json         ← the plugin manifest
├── skills/
│   ├── init-workos/SKILL.md
│   ├── uninit-workos/SKILL.md
│   ├── create-workstation/SKILL.md
│   ├── session-audit/SKILL.md
│   └── workspace-audit/SKILL.md
└── README.md
```

## Source

Built from patterns in Jeff Su's Cowork tutorials and adapted for a more flexible multi-domain layout.

## License

MIT — see [LICENSE](LICENSE).
