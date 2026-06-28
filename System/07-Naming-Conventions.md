# Naming Conventions

## Purpose

This document defines the naming standards for files, folders, notes, templates, and links in this Second Brain.

Consistent names make the knowledge system easier to:

- navigate
- search
- link
- maintain
- audit
- understand across AI sessions

Naming is not cosmetic.

Good names reduce future thinking and prevent duplicate knowledge.

---

# Scope

These conventions apply to:

- root constitution documents
- operational procedure documents
- templates
- permanent knowledge notes
- decision notes
- project notes
- reference notes
- resource notes
- archive notes
- folders
- Obsidian wikilinks

---

# Core Principles

Names should be:

- clear
- stable
- descriptive
- searchable
- human-readable
- AI-readable
- consistent with note type

Avoid names that are:

- vague
- temporary
- overly clever
- too long
- duplicated
- dependent on hidden context

---

# Root Constitution Documents

Root constitution documents should use numeric prefixes to preserve reading order.

Format:

```text
NN-Descriptive-Title.md
```

Rules:

- Use two digits for the numeric prefix.
- Use Title Case after the number.
- Use hyphens between words.
- Keep the title concise.
- Do not rename numbered constitution files casually because links and reading order depend on them.

Examples:

```text
00-Overview.md
01-Knowledge-Philosophy.md
02-Knowledge-Principles.md
03-Knowledge-Lifecycle.md
04-Note-Types.md
05-Knowledge-Quality-Checklist.md
06-AI-Working-Guide.md
07-Naming-Conventions.md
```

---

# Folder Names

Folder names should describe the role of the documents inside them.

Format:

```text
Folder-Name/
```

Rules:

- Use Title Case.
- Use singular names only when the folder represents one system area.
- Use plural names when the folder contains multiple documents of the same kind.
- Avoid generic names like `Misc`, `Stuff`, or `Old`.

Current standard folders:

```text
Standards/
Templates/
Operations/
```

Folder purpose:

| Folder | Purpose |
|---|---|
| `Standards/` | Reusable rules for classification, organization, and system-wide consistency. |
| `Templates/` | Reusable note structures and writing formats. |
| `Operations/` | Repeatable workflows for maintaining the knowledge system. |

---

# Template Files

Template files should clearly identify the note type they support.

Format:

```text
Note-Type-Template.md
```

Rules:

- End template filenames with `-Template.md`.
- Use the note type as the first part of the filename.
- Keep template names stable because they are referenced by other documents.

Examples:

```text
Templates/Knowledge-Note-Template.md
Templates/Decision-Note-Template.md
```

---

# Operational Procedure Files

Operational procedure files should describe the repeatable workflow they define.

Format:

```text
Operations/Workflow-Name.md
```

Rules:

- Use descriptive process names.
- Prefer nouns that describe the workflow outcome.
- Avoid numbering inside `Operations/` unless a reading order becomes necessary.

Example:

```text
Operations/Knowledge-System-Audit.md
```

---

# Knowledge Notes

Knowledge Notes should name one reusable concept.

Format:

```text
Concept-Name.md
```

Rules:

- Use one clear concept per filename.
- Prefer evergreen concept names.
- Avoid project-specific names unless the note is a Project Note.
- Avoid prefixes unless ordering is required.
- Do not include dates unless time is part of the meaning.

Good examples:

```text
Retrieval-Augmented-Generation.md
Prompt-Compression.md
Authentication-Patterns.md
```

Weak examples:

```text
Notes.md
Stuff-About-AI.md
Things-To-Remember.md
My-Thoughts.md
```

---

# Decision Notes

Decision Notes should state the decision subject clearly.

Format:

```text
Decision-Subject.md
```

Rules:

- Name the chosen decision, not the discussion.
- Prefer action-oriented titles.
- Avoid vague names like `Decision.md`.
- Add a date only if multiple decisions with the same title are expected.

Examples:

```text
Use-Supabase-for-Authentication.md
Adopt-Nextjs-App-Router.md
Store-Embeddings-in-Postgres.md
```

---

# Project Notes

Project Notes should identify the project and the document purpose.

Format:

```text
Project-Name-Document-Purpose.md
```

Examples:

```text
Second-Brain-Roadmap.md
Second-Brain-Architecture.md
Client-Portal-API-Design.md
```

---

# Reference Notes

Reference Notes should name the source or topic being summarized.

Format:

```text
Source-or-Topic-Reference.md
```

Rules:

- Summarize the source instead of copying it.
- Include `Reference` only when it helps distinguish the note from a Knowledge Note.

Examples:

```text
Obsidian-Properties-Reference.md
Postgres-Indexing-Reference.md
```

---

# Archive Notes

Archive Notes should preserve historical meaning.

Format:

```text
Archived-Original-Name.md
```

Rules:

- Preserve enough of the original name to understand what was archived.
- Use archive folders later if the archive grows.
- Do not archive low-value temporary notes that should be deleted.

---

# Wikilinks

Use Obsidian wikilinks for internal vault links.

Rules:

- Use Obsidian wikilink syntax for internal notes.
- Use display text when a shorter label improves readability.
- Use Markdown links only for external URLs.
- Prefer linking to the note, not duplicating the same explanation.

Examples:

```markdown
[[00-Overview]]
[[06-AI-Working-Guide]]
[[Operations/Knowledge-System-Audit]]
[[Templates/Knowledge-Note-Template|Knowledge Note Template]]
```

---

# Dates

Use dates only when the date is part of the note identity.

Format:

```text
YYYY-MM-DD
```

Use dates for:

- Daily Notes
- meeting notes
- time-specific decisions
- historical records

Avoid dates for evergreen Knowledge Notes.

---

# Renaming Rules

Rename a file when:

- the current name is misleading
- the note type changed
- the name is too vague to search
- the name duplicates another note
- the filename no longer matches the note content

Before renaming:

1. Check incoming links.
2. Check related notes.
3. Update references if Obsidian does not do it automatically.
4. Perform impact analysis using [[06-AI-Working-Guide]].
5. For system-wide changes, use [[Operations/Knowledge-System-Audit]].

---

# Anti-Patterns

Avoid:

- `Untitled.md`
- `New Note.md`
- `Misc.md`
- `Random.md`
- `AI Notes.md`
- `Important.md`
- `Final.md`
- `Final-Final.md`
- date prefixes on evergreen notes
- duplicate filenames with small wording differences

---

# AI Instructions

Before creating or renaming a note, AI assistants should:

- read [[00-Overview]]
- follow [[02-Knowledge-Principles]]
- choose the correct note type from [[04-Note-Types]]
- choose domains using [[Standards/Knowledge-Taxonomy]]
- choose folder placement using [[Standards/Vault-Structure]]
- use the correct template
- search for duplicates
- prefer improving existing notes over creating new ones
- use Obsidian wikilinks for internal references
- perform impact analysis using [[06-AI-Working-Guide]]

AI should not invent new naming styles without first proposing the change.

---

# Review Checklist

Before finalizing a new or renamed note, confirm:

- The filename clearly describes the note.
- The filename matches the note type.
- The note title and filename agree.
- The name is stable enough for long-term use.
- The name does not duplicate an existing note.
- Related notes are linked.
- Internal references use wikilinks.
- Any structural naming change has completed impact analysis.

---

# Principle

A good name should make the note easier to find, understand, link, and maintain.

If the name does not reduce future thinking, improve it.
