# Metadata Standards

## Purpose

This document defines the metadata standards for notes in this Second Brain.

Metadata exists to make knowledge easier to:

- organize
- search
- filter
- audit
- maintain
- connect
- understand across AI sessions

Metadata should improve retrieval and maintenance.

It should not become administrative noise.

---

# Scope

These standards apply to Obsidian properties, YAML frontmatter, tags, aliases, status fields, dates, note type fields, and relationship fields.

Use this document when:

- creating a new permanent note
- creating or updating templates
- renaming or reorganizing notes
- adding properties to existing notes
- auditing metadata consistency
- building future Bases, tables, filters, or dashboards

---

# Core Principles

Metadata should be:

- minimal
- consistent
- searchable
- useful for retrieval
- useful for maintenance
- understandable without hidden context

Avoid metadata that is:

- decorative
- duplicated in the note body without purpose
- hard to maintain
- inconsistent across note types
- too detailed for long-term use
- dependent on one tool or one temporary workflow

---

# Frontmatter Format

Use YAML frontmatter at the top of notes when metadata is needed.

Format:

```yaml
---
title: Example Note
type: knowledge
status: draft
created: 2026-06-28
updated: 2026-06-28
tags:
  - knowledge
aliases:
  - Example Alias
related:
  - "[[00-Overview]]"
---
```

Rules:

- Put frontmatter at the very top of the file.
- Use lowercase property names.
- Use kebab-case values for tags and controlled status values.
- Use ISO dates: `YYYY-MM-DD`.
- Use YAML lists for multi-value properties.
- Use quoted wikilinks inside YAML lists.
- Do not add properties that will not be maintained.

---

# Required Metadata

Permanent notes should usually include:

```yaml
---
title:
type:
status:
created:
updated:
tags:
---
```

Required properties may be omitted only when the note is a constitution document whose filename and role already make the context obvious.

---

# Core Properties

| Property | Type | Purpose |
|---|---|---|
| `title` | text | Human-readable title of the note. |
| `type` | text | Primary note type from [[04-Note-Types]]. |
| `status` | text | Current maturity or lifecycle state. |
| `created` | date | Date the note was created. |
| `updated` | date | Date the note was last meaningfully updated. |
| `tags` | list | Searchable categorization. |
| `aliases` | list | Alternative names used for linking or discovery. |
| `related` | list | Important related notes. |

---

# Note Type Values

Use lowercase values for `type`.

Allowed values:

```yaml
type: inbox
type: daily
type: knowledge
type: decision
type: project
type: reference
type: template
type: resource
type: archive
```

The `type` value must match the note's primary note type in [[04-Note-Types]].

Every note should have one primary type.

---

# Status Values

Use `status` to describe maturity or current use.

General status values:

```yaml
status: temporary
status: draft
status: active
status: reviewed
status: archived
```

Decision-specific status values:

```yaml
status: proposed
status: accepted
status: superseded
status: rejected
status: deprecated
```

Rules:

- Use lowercase values.
- Use one status value at a time.
- Update status when the note meaningfully changes.
- Do not create new status values without updating this standard.

---

# Date Properties

Use ISO format:

```yaml
created: 2026-06-28
updated: 2026-06-28
```

Rules:

- `created` records when the note was first created.
- `updated` records the last meaningful content update.
- Do not update `updated` for formatting-only changes unless the formatting affects usability.
- Use dates only when they improve maintenance or retrieval.

---

# Tags

Tags should support search, grouping, and review.

Rules:

- Use lowercase tags.
- Use hyphens for multi-word tags.
- Use nested tags only when the hierarchy is stable.
- Avoid creating many one-off tags.
- Do not use tags as a replacement for clear note titles or links.

Examples:

```yaml
tags:
  - knowledge
  - ai-workflow
  - decision-making
```

Nested examples:

```yaml
tags:
  - project/active
  - knowledge/database
```

---

# Aliases

Use `aliases` for alternative names that improve discovery.

Examples:

```yaml
aliases:
  - AI Working Instructions
  - Assistant Workflow
```

Rules:

- Use aliases sparingly.
- Add aliases for common alternate terms.
- Do not use aliases to hide unclear filenames.
- If an alias becomes the primary name, consider renaming the note using [[07-Naming-Conventions]].

---

# Related Notes

Use `related` for important relationship links.

Example:

```yaml
related:
  - "[[02-Knowledge-Principles]]"
  - "[[03-Knowledge-Lifecycle]]"
```

Rules:

- Include only meaningful relationships.
- Prefer a few strong links over many weak links.
- Use body sections for explanations of relationships.
- Keep related links current during audits.

---

# Source Metadata

Use source metadata for Reference Notes or notes derived from external material.

Example:

```yaml
source:
  - https://example.com
author:
published:
accessed: 2026-06-28
```

Rules:

- Prefer official sources.
- Summarize external material instead of copying it.
- Use `accessed` when source content may change.

---

# Decision Metadata

Decision Notes may include:

```yaml
decision_date: 2026-06-28
decision_owner:
supersedes:
superseded_by:
```

Rules:

- Use these only when they clarify the decision history.
- Link superseded decisions with wikilinks.
- Keep the decision reasoning in the note body, not only in metadata.

---

# Project Metadata

Project Notes may include:

```yaml
project:
status:
owner:
start_date:
target_date:
```

Rules:

- Use project metadata only for project-specific notes.
- Do not add project metadata to reusable Knowledge Notes unless the project is directly relevant.

---

# Template Metadata

Templates may include example metadata blocks, but the examples should remain generic.

Rules:

- Template metadata should teach structure.
- Template metadata should not include real project-specific values.
- Template metadata must follow this standard.

---

# When Not to Use Metadata

Do not add metadata when:

- the note is temporary
- the property will not be maintained
- the metadata duplicates obvious information
- the metadata only exists because a tool supports it
- the note body is clearer without it

Metadata should make notes easier to use.

If metadata makes the note harder to maintain, remove it.

---

# AI Instructions

Before adding or changing metadata, AI assistants should:

- read [[00-Overview]]
- follow [[02-Knowledge-Principles]]
- choose the correct note type from [[04-Note-Types]]
- follow [[07-Naming-Conventions]]
- use metadata only when it improves retrieval or maintenance
- avoid inventing new properties without proposing an update
- perform impact analysis using [[06-AI-Working-Guide]]
- use [[Operations/Knowledge-System-Audit]] for system-wide metadata changes

---

# Review Checklist

Before finalizing metadata, confirm:

- Required properties are present when appropriate.
- Property names are lowercase and consistent.
- Dates use `YYYY-MM-DD`.
- Tags are useful and not excessive.
- The note type matches [[04-Note-Types]].
- Status uses an approved value.
- Related links are meaningful.
- Metadata does not duplicate unnecessary body content.
- Any new property has a clear long-term purpose.

---

# Principle

Metadata should make knowledge easier to retrieve, maintain, and connect.

If a property does not serve that purpose, it should not exist.
