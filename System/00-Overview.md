# AI Second Brain Constitution

## Purpose

This file is the entry point for the AI Second Brain.

It explains:

- what this knowledge system is for
- which constitution documents define its rules
- how those documents depend on each other
- how knowledge should move through the system
- how humans and AI assistants should contribute

The goal is not to collect more notes.

The goal is to build a durable thinking system that reduces repeated research, preserves important decisions, and makes future AI collaboration more effective.

---

## Constitution Map

The constitution is made of five core documents, one AI workflow guide, four standards, one operational procedure layer, and one template layer.

| Order | Document | Role | Depends On | Used For |
|---|---|---|---|---|
| 1 | [[01-Knowledge-Philosophy]] | Defines the purpose and mindset of the system. | None | Understanding why knowledge is stored. |
| 2 | [[02-Knowledge-Principles]] | Defines the rules every valuable note should follow. | [[01-Knowledge-Philosophy]] | Deciding what should or should not become knowledge. |
| 3 | [[03-Knowledge-Lifecycle]] | Defines how raw information becomes reusable knowledge. | [[01-Knowledge-Philosophy]], [[02-Knowledge-Principles]] | Moving ideas from capture to permanent knowledge. |
| 4 | [[04-Note-Types]] | Defines the purpose and lifetime of each note type. | [[02-Knowledge-Principles]], [[03-Knowledge-Lifecycle]] | Choosing where a note belongs. |
| 5 | [[05-Knowledge-Quality-Checklist]] | Defines the review standard for permanent notes. | All previous documents | Checking whether a note is clear, reusable, and maintainable. |
| 6 | [[06-AI-Working-Guide]] | Defines the operational workflow for AI assistants. | All previous documents | Performing impact analysis, update proposals, and approved maintenance. |
| 7 | [[07-Naming-Conventions]] | Defines naming standards for notes, folders, templates, operations, and links. | [[04-Note-Types]], [[06-AI-Working-Guide]] | Creating stable names and preventing duplicate knowledge. |
| 8 | [[08-Metadata-Standards]] | Defines metadata, frontmatter, tag, status, and relationship standards. | [[04-Note-Types]], [[07-Naming-Conventions]] | Keeping properties useful, searchable, and maintainable. |
| 9 | [[Standards/Knowledge-Taxonomy]] | Defines domain, cross-domain, and project/general classification rules. | [[04-Note-Types]], [[07-Naming-Conventions]], [[08-Metadata-Standards]] | Choosing what kind of knowledge a note represents. |
| 10 | [[Standards/Vault-Structure]] | Defines top-level folders, folder responsibilities, and storage rules. | [[04-Note-Types]], [[07-Naming-Conventions]], [[Standards/Knowledge-Taxonomy]] | Choosing where notes should physically live in the vault. |
| 11 | [[Operations/Knowledge-System-Audit]] | Defines the system-wide audit and synchronization process. | [[06-AI-Working-Guide]], [[07-Naming-Conventions]], [[08-Metadata-Standards]], [[Standards/Knowledge-Taxonomy]], [[Standards/Vault-Structure]] | Auditing changes, finding affected documents, and validating consistency. |
| 12 | [[Templates/Knowledge-Note-Template]] | Provides a standard structure for permanent Knowledge Notes. | [[02-Knowledge-Principles]], [[03-Knowledge-Lifecycle]], [[05-Knowledge-Quality-Checklist]], [[08-Metadata-Standards]], [[Standards/Knowledge-Taxonomy]], [[Standards/Vault-Structure]] | Writing consistent long-term knowledge notes. |
| 13 | [[Templates/Decision-Note-Template]] | Provides a standard structure for permanent Decision Notes. | [[01-Knowledge-Philosophy]], [[04-Note-Types]], [[06-AI-Working-Guide]], [[08-Metadata-Standards]] | Preserving decisions, reasoning, trade-offs, and reversal conditions. |

---

## Recommended Reading Order

Read the constitution in this order when working with this vault for the first time:

1. [[01-Knowledge-Philosophy]]
2. [[02-Knowledge-Principles]]
3. [[03-Knowledge-Lifecycle]]
4. [[04-Note-Types]]
5. [[05-Knowledge-Quality-Checklist]]
6. [[06-AI-Working-Guide]]
7. [[07-Naming-Conventions]]
8. [[08-Metadata-Standards]]
9. [[Standards/Knowledge-Taxonomy]]
10. [[Standards/Vault-Structure]]
11. [[Operations/Knowledge-System-Audit]]
12. [[Templates/Knowledge-Note-Template]]
13. [[Templates/Decision-Note-Template]]

Each document narrows the previous one:

- Philosophy defines why the system exists.
- Principles define what good knowledge must satisfy.
- Lifecycle defines how knowledge matures.
- Note Types define where knowledge belongs.
- Quality Checklist defines whether the note is good enough.
- AI Working Guide defines how AI assistants should apply the system.
- Naming Conventions define stable names for notes, folders, templates, operations, and links.
- Metadata Standards define stable properties, tags, statuses, and relationships.
- Knowledge Taxonomy defines stable domains and cross-domain classification.
- Vault Structure defines stable storage folders and folder responsibilities.
- Operational procedures define repeatable system-wide maintenance workflows.
- Templates turn the rules into repeatable writing structures.

---

## Core Operating Model

Every valuable idea should move through the same path:

```text
Capture
  -> Evaluate
  -> Distill
  -> Choose Note Type
  -> Write or Update Note
  -> Connect Related Notes
  -> Review Quality
  -> Store
  -> Improve Continuously
```

This workflow prevents the vault from becoming a storage dump.

Raw information may be captured temporarily, but permanent knowledge must be evaluated, distilled, organized, connected, and reviewed.

---

## Document Relationships

### Philosophy controls direction

[[01-Knowledge-Philosophy]] is the foundation. It defines the system as a knowledge operating system, not a note-taking archive.

Use it when deciding whether a piece of information deserves long-term storage.

### Principles control content

[[02-Knowledge-Principles]] turns the philosophy into rules.

Use it before writing or changing any important note. It answers questions such as:

- Is this reusable?
- Is this evergreen?
- Is this written for future use?
- Does this reduce future thinking?
- Does this improve AI collaboration?

### Lifecycle controls process

[[03-Knowledge-Lifecycle]] defines how information matures.

Use it when deciding whether something is still raw capture, ready for distillation, ready for permanent storage, or ready to archive/delete.

### Note Types control placement

[[04-Note-Types]] defines the primary role of each note.

Use it before creating a note. Every note should have one primary type:

- Inbox
- Daily
- Knowledge
- Decision
- Project
- Reference
- Template
- Resource
- Archive

### Quality Checklist controls readiness

[[05-Knowledge-Quality-Checklist]] defines the review standard for permanent knowledge.

Use it before treating an important note as long-term knowledge.

### AI Working Guide controls AI execution

[[06-AI-Working-Guide]] defines how AI assistants should work inside this vault.

Use it before making structural changes, creating notes, changing templates, or updating permanent knowledge.

### Naming Conventions control names and links

[[07-Naming-Conventions]] defines how files, folders, templates, operations, notes, and wikilinks should be named.

Use it before creating, renaming, moving, or reorganizing notes.

### Metadata Standards control properties

[[08-Metadata-Standards]] defines how Obsidian properties, frontmatter, tags, aliases, statuses, dates, and relationship fields should be used.

Use it before adding or changing note metadata.

### Knowledge Taxonomy controls classification

[[Standards/Knowledge-Taxonomy]] defines how knowledge should be classified by note type, domain, topic, and relationship.

Use it before creating Knowledge Notes, choosing domains, or separating Project Knowledge from General Knowledge.

### Vault Structure controls storage

[[Standards/Vault-Structure]] defines what belongs in `Knowledge/`, `Projects/`, `Resources/`, `Inbox/`, `Daily/`, `Archive/`, and `System/`.

Use it before choosing a folder, moving notes, creating folders, or deciding whether a note is temporary, project-specific, source-oriented, permanent, historical, or system-level.

### Operations control system maintenance

[[Operations/Knowledge-System-Audit]] defines the repeatable process for auditing and synchronizing the whole knowledge system.

Use it when documentation, standards, templates, folders, or permanent knowledge changes may affect multiple files.

### Templates control execution

Templates turn the constitution into repeatable note structures.

Use [[Templates/Knowledge-Note-Template]] when creating a permanent Knowledge Note.

Use [[Templates/Decision-Note-Template]] when recording an important decision.

---

## Rules for Creating or Editing Notes

Before creating a new note:

1. Check whether the information is likely to be useful again.
2. Search for an existing note that should be improved instead.
3. Choose the correct primary note type.
4. Distill the idea before saving it.
5. Link related notes using Obsidian wikilinks.
6. Review important notes with the quality checklist.

Prefer improving existing knowledge over creating duplicates.

Prefer decisions, principles, patterns, and lessons over raw discussions.

Prefer concise reusable notes over long temporary records.

---

## For AI Assistants

AI assistants working in this vault must treat the constitution as the source of truth.

Before making meaningful changes, AI should:

- read this overview
- follow [[02-Knowledge-Principles]]
- respect [[03-Knowledge-Lifecycle]]
- choose the correct note type from [[04-Note-Types]]
- validate important notes with [[05-Knowledge-Quality-Checklist]]
- follow [[06-AI-Working-Guide]] for impact analysis and update proposals
- use [[Standards/Knowledge-Taxonomy]] when choosing domains and cross-domain classification
- use [[Standards/Vault-Structure]] when choosing storage folders
- use templates when creating structured permanent notes
- improve existing notes instead of creating unnecessary duplicates

AI should not store raw conversations, temporary context, copied documentation, or low-value notes as permanent knowledge.

Every AI contribution should make the vault clearer, more reusable, and easier to maintain.

---

## What Belongs in This Vault

Store knowledge that:

- will likely be useful again
- improves future decisions
- reduces repeated research
- helps future AI assistants understand context quickly
- explains reasoning, trade-offs, patterns, or lessons
- can be connected to other knowledge

Avoid storing:

- raw AI conversations
- temporary thoughts
- copied documentation
- duplicate notes
- easily searchable information with no added understanding
- low-value records that will not matter later

---

## Long-Term Standard

This Second Brain should become more valuable over time.

That means every important note should be:

- clear
- atomic
- self-contained
- reusable
- connected
- maintainable
- AI-readable

Knowledge is an asset only when it improves future thinking.

If a note does not create future value, improve it, archive it, or remove it.
