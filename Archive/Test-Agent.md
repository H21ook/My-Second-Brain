# AI Project Instructions

This project uses an external Obsidian Second Brain vault as project memory and AI context.

## Vault Path

```text
D:\own\My-Second-Brain
```

## Project Notes

For E-Geree-v3, project notes live here:

```text
D:\own\My-Second-Brain\Projects\E-Geree-v3
```

Use these notes as project memory. Use production code as the source of truth.

If vault notes conflict with code, trust the code after verification and update the affected notes.

## Before Work

Before implementing non-trivial changes, read the relevant project notes from the vault.

Start with:

- `Projects/E-Geree-v3/E-Geree-v3-Overview.md`
- `Projects/E-Geree-v3/E-Geree-v3-Architecture.md`
- `Projects/E-Geree-v3/E-Geree-v3-Contract-Create-Feature.md`
- `Projects/E-Geree-v3/E-Geree-v3-State-Management.md`
- `Projects/E-Geree-v3/E-Geree-v3-Routing.md`
- `Projects/E-Geree-v3/E-Geree-v3-Networking-BFF.md`
- `Projects/E-Geree-v3/E-Geree-v3-RHF-Migration-Plan.md`
- the latest `Projects/E-Geree-v3/E-Geree-v3-Worklog-YYYY-MM-DD.md`

Choose only the notes relevant to the current task. Do not load the entire vault unless the user asks for a broader audit.

## During Work

When working on code:

1. Inspect the actual production code before making claims.
2. Prefer existing project patterns over new abstractions.
3. Keep changes scoped to the requested task.
4. Record important progress, findings, decisions, and verification results in the current E-Geree-v3 worklog.
5. Use absolute dates in notes instead of relative dates like "today" or "tomorrow".

Current worklog format:

```text
Projects/E-Geree-v3/E-Geree-v3-Worklog-YYYY-MM-DD.md
```

## After Work

After completing implementation:

1. Update the current E-Geree-v3 worklog.
2. Check whether architecture, routing, state management, API contracts, data flow, validation, persistence, or user workflow changed.
3. If any project-level truth changed, update the affected project note.
4. If the lesson is reusable outside E-Geree-v3, propose or update a `Knowledge/` note in the Second Brain.
5. Do not store raw chat history as permanent knowledge.

## Documentation Impact

Update these project notes when their area changes:

- Architecture changes -> `E-Geree-v3-Architecture.md`
- Route or BFF changes -> `E-Geree-v3-Routing.md` or `E-Geree-v3-Networking-BFF.md`
- Contract create wizard changes -> `E-Geree-v3-Contract-Create-Feature.md`
- Redux, persistence, or global state changes -> `E-Geree-v3-State-Management.md`
- PDF viewer behavior changes -> `E-Geree-v3-PDF-Viewer.md`
- RHF migration progress or decisions -> `E-Geree-v3-RHF-Migration-Plan.md`

If no project documentation needs updating, say so explicitly in the final response.

## Filesystem Access

If the AI agent cannot read or write `D:\own\My-Second-Brain` because of sandbox permissions, ask the user to add the vault as a readable or writable workspace root, or ask the user to paste the relevant notes.

Do not silently skip vault updates when documentation impact exists.

## For Claude Code

Codex reads `AGENTS.md`.

Claude Code reads `CLAUDE.md`.

If this project uses Claude Code too, create a `CLAUDE.md` that points to `AGENTS.md`:

```md
See AGENTS.md.
```
