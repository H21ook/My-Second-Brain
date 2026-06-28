# Vault Structure

## Purpose

This document defines where notes should be stored inside this Second Brain.

It answers:

- which folder a note belongs in
- what each top-level folder is responsible for
- what should not be stored in each folder
- how AI assistants should choose a folder for a new note
- how folder placement differs from knowledge taxonomy

Use [[Standards/Knowledge-Taxonomy]] to decide what kind of knowledge something is.

Use this document to decide where that note should physically live in the vault.

---

# Core Principle

Store notes by lifecycle and use, not by mood or temporary context.

The primary question is:

> Where should this note live so future humans and AI assistants can find, maintain, and reuse it?

Folder placement should make the note easier to:

- find
- maintain
- audit
- archive
- connect
- use in future AI sessions

If a folder does not improve these outcomes, do not create or use it.

---

# Top-Level Folder Model

The vault uses a small set of top-level folders.

```text
Knowledge/
Projects/
Resources/
Inbox/
Daily/
Archive/
System/
```

Each folder represents a different lifecycle or operating role.

Do not create new top-level folders casually.

---

# Folder Summary

| Folder | Primary Role | Lifetime |
|---|---|---|
| `Knowledge/` | Evergreen reusable knowledge | Long |
| `Projects/` | Project-specific working knowledge | Project lifetime |
| `Resources/` | Curated external resources and source summaries | Medium to long |
| `Inbox/` | Temporary capture | Very short |
| `Daily/` | Daily work log and short-term context | Short |
| `Archive/` | Historical inactive material | Long |
| `System/` | Rules, standards, templates, and operations | Long |

---

# Knowledge/

## Purpose

`Knowledge/` stores permanent reusable knowledge.

Use this folder for ideas that remain valuable beyond one day, one project, or one source.

## What Belongs Here

- evergreen concepts
- reusable principles
- patterns
- techniques
- frameworks
- lessons that apply across projects
- distilled explanations
- general best practices

Examples:

```text
Knowledge/Authentication-Patterns.md
Knowledge/Retrieval-Augmented-Generation.md
Knowledge/Pricing-Strategy.md
Knowledge/Prompt-Compression.md
```

## What Does Not Belong Here

- raw captures
- daily logs
- project status updates
- copied documentation
- unprocessed source notes
- task lists
- project-specific implementation details

## Rule

Only store a note in `Knowledge/` when it is reusable outside its original context.

If the note is still raw, keep it in `Inbox/`.

If it only makes sense inside one project, keep it in `Projects/`.

---

# Projects/

## Purpose

`Projects/` stores knowledge tied to a specific project, product, client, initiative, or implementation.

Use this folder when the note depends on project context.

## What Belongs Here

- project plans
- project architecture
- project decisions
- implementation notes
- project-specific trade-offs
- roadmaps
- requirements
- active project research
- project-specific debugging notes
- project status and delivery notes

Examples:

```text
Projects/Second-Brain-Roadmap.md
Projects/Client-Portal-API-Design.md
Projects/Mobile-App-Launch-Plan.md
```

## What Does Not Belong Here

- general principles
- reusable technical patterns
- evergreen business frameworks
- generic AI techniques
- notes that multiple projects should share

## Rule

If the note would lose most of its value when the project ends, store it in `Projects/`.

If the project produces a reusable lesson, extract that lesson into `Knowledge/` and link back to the project only when useful.

---

# Resources/

## Purpose

`Resources/` stores curated external material and source-oriented notes.

Resources point to useful material.

Knowledge explains reusable understanding.

## What Belongs Here

- book notes
- article summaries
- documentation summaries
- video summaries
- useful links
- tool lists
- library references
- source collections

Examples:

```text
Resources/Obsidian-Properties-Reference.md
Resources/Postgres-Indexing-Reference.md
Resources/AI-Tools-Directory.md
```

## What Does Not Belong Here

- copied articles without summary
- raw browser bookmarks without curation
- permanent principles that should be in `Knowledge/`
- project requirements
- active task lists

## Rule

Store source-oriented notes in `Resources/`.

Extract reusable insights from resources into `Knowledge/` when they become useful beyond the source.

---

# Inbox/

## Purpose

`Inbox/` stores temporary captured information before it is processed.

It is a staging area, not a permanent home.

## What Belongs Here

- quick thoughts
- rough ideas
- unprocessed notes
- temporary AI outputs
- raw meeting fragments
- links to evaluate later
- incomplete captures

Examples:

```text
Inbox/Raw-Idea.md
Inbox/AI-Conversation-Notes.md
Inbox/Links-To-Review.md
```

## What Does Not Belong Here

- polished Knowledge Notes
- long-term project documentation
- final decisions
- templates
- standards
- permanent references

## Rule

Inbox notes must eventually be:

- distilled into `Knowledge/`
- moved into `Projects/`
- summarized into `Resources/`
- archived
- deleted

`Inbox/` should not become a hidden archive.

---

# Daily/

## Purpose

`Daily/` stores daily work context.

Daily notes are useful for tracking activity, but they are not permanent knowledge by default.

## What Belongs Here

- daily plans
- daily progress
- meeting notes from the day
- work logs
- temporary context
- short discoveries
- task notes

Examples:

```text
Daily/2026-06-28.md
Daily/2026-06-29.md
```

## What Does Not Belong Here

- evergreen explanations
- permanent decisions
- system standards
- reusable project architecture
- long-term reference notes

## Rule

Daily notes can contain raw work.

Important lessons, decisions, and reusable knowledge should be extracted into the correct permanent folder.

---

# Archive/

## Purpose

`Archive/` stores inactive material that should be preserved but not maintained as active knowledge.

Archive is for historical value.

It is not a dumping ground.

## What Belongs Here

- completed project notes worth preserving
- obsolete but historically useful notes
- superseded decisions
- inactive resources
- old standards retained for reference
- important historical snapshots

Examples:

```text
Archive/Archived-Old-Project-Plan.md
Archive/Archived-Deprecated-Decision.md
```

## What Does Not Belong Here

- low-value temporary notes
- duplicate notes that should be deleted
- active project notes
- current standards
- current templates

## Rule

Archive notes only when keeping history has value.

Delete low-value temporary notes instead of archiving them.

---

# System/

## Purpose

`System/` stores the operating rules of the vault.

This folder defines how the Second Brain works.

## What Belongs Here

- constitution documents
- AI workflow guides
- naming standards
- metadata standards
- taxonomy standards
- vault structure standards
- templates
- operational procedures
- audit workflows

Examples:

```text
System/00-Overview.md
System/06-AI-Working-Guide.md
System/Standards/Knowledge-Taxonomy.md
System/Standards/Vault-Structure.md
System/Templates/Knowledge-Note-Template.md
System/Operations/Knowledge-System-Audit.md
```

## What Does Not Belong Here

- personal notes
- project plans
- raw ideas
- general knowledge notes
- external resources
- daily logs

## Rule

Store a note in `System/` only when it defines how the vault should operate.

System notes are standards, workflows, templates, or governance documents.

---

# AI Folder Selection Procedure

When AI creates a new note, it should choose the folder in this order:

1. Read [[00-Overview]].
2. Choose the note type using [[04-Note-Types]].
3. Choose the knowledge domain using [[Standards/Knowledge-Taxonomy]].
4. Decide whether the note is temporary, reusable, project-specific, source-oriented, historical, or system-level.
5. Search for existing notes that should be updated instead.
6. Choose the folder using the rules in this document.
7. Choose the filename using [[07-Naming-Conventions]].
8. Add metadata using [[08-Metadata-Standards]] when useful.
9. Link related notes.
10. Perform impact analysis using [[06-AI-Working-Guide]].

AI should not create a new folder when an existing top-level folder already describes the note's lifecycle.

---

# Folder Decision Table

| If the note is... | Store in... |
|---|---|
| Raw, temporary, or unprocessed | `Inbox/` |
| A daily work record | `Daily/` |
| Evergreen and reusable | `Knowledge/` |
| Specific to one project | `Projects/` |
| A curated source, link set, or reference summary | `Resources/` |
| Inactive but historically useful | `Archive/` |
| A rule, template, workflow, or standard for the vault | `System/` |

---

# Project vs Knowledge Storage

Project-specific material belongs in `Projects/`.

Reusable principles belong in `Knowledge/`.

When a project creates a general lesson:

1. Keep the implementation detail in `Projects/`.
2. Extract the reusable principle into `Knowledge/`.
3. Link the two notes.
4. Avoid copying the same explanation into both places.

This keeps project context and general knowledge separate without disconnecting them.

---

# Resource vs Knowledge Storage

Resources preserve source context.

Knowledge preserves understanding.

Store a source summary in `Resources/` when the note mainly answers:

- what did this source say?
- where did this come from?
- what external material should I revisit?

Store a distilled note in `Knowledge/` when the note mainly answers:

- what principle did I learn?
- when should I use it?
- what trade-offs matter?
- how does this improve future decisions?

---

# Inbox and Daily Processing

`Inbox/` and `Daily/` are temporary sources of future knowledge.

They should be reviewed regularly.

Useful material should move toward:

- `Knowledge/` when reusable
- `Projects/` when project-specific
- `Resources/` when source-oriented
- `Archive/` when historically useful
- deletion when low value

Do not let temporary folders become permanent storage.

---

# New Folder Rules

Do not create a new top-level folder unless:

- the existing folders cannot describe the note lifecycle
- the category will contain multiple notes
- the folder has a clear maintenance rule
- the folder reduces navigation cost
- the folder is likely to remain stable
- the change has been approved after impact analysis

Use subfolders only when they make a large folder easier to navigate.

Prefer shallow structure.

If a category can be handled by links, tags, or metadata, do not create a folder.

---

# Review Checklist

Before finalizing folder placement, confirm:

- The folder matches the note lifecycle.
- The note type from [[04-Note-Types]] supports the placement.
- The domain from [[Standards/Knowledge-Taxonomy]] is clear.
- The note does not belong in an existing note instead.
- Temporary notes are not stored as permanent knowledge.
- Project-specific notes are not mixed into general knowledge.
- Source summaries are not confused with distilled knowledge.
- System documents are only used for operating rules.
- Cross-folder relationships are handled with wikilinks.
- New folders are justified by stable structure, not temporary convenience.

---

# Principle

Vault structure should make the right storage location obvious.

If folder placement requires too much explanation, simplify the structure or improve the note's type, title, and links.
