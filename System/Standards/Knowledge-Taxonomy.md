# Knowledge Taxonomy

## Purpose

This document defines how knowledge should be classified inside this Second Brain.

Taxonomy exists to answer:

- what domain a note belongs to
- what broad knowledge area the note improves
- how AI assistants should choose a domain for new notes
- how cross-domain knowledge should be handled
- how Project Knowledge differs from General Knowledge

Taxonomy is not decoration.

It is a decision system for keeping knowledge conceptually clear, reusable, and maintainable.

Use [[Standards/Vault-Structure]] to decide where the note should physically live.

---

# Core Principle

Classify knowledge by its reusable purpose, not by where it came from.

The primary question is:

> What future thinking or action will this note support?

A note should belong where it will be most useful later.

Do not classify a note only by:

- the conversation that produced it
- the tool used to create it
- the project where it first appeared
- the source where it was discovered
- a temporary personal association

Origin can be captured in metadata or links.

Classification should express long-term use.

---

# Classification Layers

Use these layers in order:

1. Note Type
2. Domain
3. Topic
4. Relationships

## 1. Note Type

First choose the note type from [[04-Note-Types]].

Examples:

- Knowledge
- Decision
- Project
- Reference
- Template
- Resource

Every note should have one primary note type.

## 2. Domain

Then choose the broad knowledge domain.

A domain is a stable area of knowledge, work, or responsibility.

Examples:

- Software Engineering
- AI Systems
- Business
- Learning
- Personal Knowledge Management
- Operations
- Writing

Domains should be broad enough to contain many notes.

They should not be created for one isolated note.

## 3. Topic

Then identify the specific concept inside the domain.

Examples:

- Authentication Patterns
- Retrieval-Augmented Generation
- Pricing Strategy
- Knowledge Distillation
- Note Naming

The topic usually becomes the note title.

## 4. Relationships

Finally connect the note to related knowledge using wikilinks and metadata.

Relationships handle nuance better than folders.

Use links for:

- related concepts
- prerequisites
- examples
- alternatives
- project applications
- source material
- decisions

---

# Domain Selection

Choose the domain based on the note's primary future use.

Ask:

- Where will I look for this later?
- What larger body of knowledge does this improve?
- Which future decision or workflow will this support?
- Would this note still matter outside the project where it appeared?
- Which domain would be harmed most if this note were missing?

If one domain clearly answers these questions, use that domain.

If multiple domains apply, choose the domain that owns the reusable principle.

---

# Taxonomy vs Vault Structure

Taxonomy answers:

> What kind of knowledge is this?

Vault structure answers:

> Where should this note be stored?

Examples:

| Question | Use |
|---|---|
| Is this AI, programming, business, marketing, or personal knowledge management? | [[Standards/Knowledge-Taxonomy]] |
| Should this live in `Knowledge/`, `Projects/`, `Resources/`, `Inbox/`, `Daily/`, `Archive/`, or `System/`? | [[Standards/Vault-Structure]] |
| Is this general knowledge or project-specific knowledge? | [[Standards/Knowledge-Taxonomy]] |
| Which folder owns the note lifecycle? | [[Standards/Vault-Structure]] |

Domain is conceptual.

Folder is structural.

Do not use taxonomy as a substitute for folder placement rules.

---

# AI Domain Selection Procedure

When AI creates a new note, it should follow this procedure:

1. Read [[00-Overview]].
2. Choose the note type using [[04-Note-Types]].
3. Search for an existing note that should be improved instead.
4. Identify the reusable purpose of the note.
5. Choose the primary domain based on future use.
6. Choose the storage folder using [[Standards/Vault-Structure]].
7. Choose a title using [[07-Naming-Conventions]].
8. Add metadata using [[08-Metadata-Standards]] when useful.
9. Link related notes.
10. Perform impact analysis using [[06-AI-Working-Guide]].

AI should prefer improving an existing note over creating a duplicate.

AI should not invent a new domain without checking whether an existing domain already covers the idea.

---

# Cross-Domain Knowledge

Some knowledge belongs to more than one domain.

Cross-domain knowledge should still have one primary domain.

Choose the primary domain based on the reusable principle.

Then connect the other domains through:

- related links
- tags
- aliases
- examples
- project application sections
- decision notes

Do not duplicate the same knowledge across multiple domain notes.

Duplication creates maintenance debt.

## Cross-Domain Rule

If the note explains a general principle, store it in the domain that owns the principle.

If the note explains a project-specific application, store it with the project and link to the general principle.

## Examples

| Knowledge | Primary Home | Related Links |
|---|---|---|
| Prompt compression as a reusable technique | AI Systems | Personal Knowledge Management, Writing |
| Authentication pattern used in one app | Software Engineering | Project note, Decision note |
| A business pricing decision for one product | Project Knowledge | General pricing principles |
| A general pricing model | Business | Project examples |
| Naming rules for notes | System Standards | Personal Knowledge Management |

---

# Project Knowledge vs General Knowledge

Project Knowledge is tied to one project.

General Knowledge is reusable across projects.

Use this distinction before choosing where a note belongs.

## Project Knowledge

Use Project Knowledge when the note depends on a specific project context.

Project Knowledge includes:

- project goals
- project architecture
- project roadmap
- project constraints
- implementation plans
- project-specific decisions
- project-specific bugs
- project-specific trade-offs
- delivery status

Ask:

- Would this note lose meaning if the project ended?
- Is this mostly about one product, client, app, or initiative?
- Does this depend on project-specific constraints?
- Will this be maintained only while the project is active?

If yes, store it as Project Knowledge.

## General Knowledge

Use General Knowledge when the note remains useful outside one project.

General Knowledge includes:

- principles
- patterns
- reusable techniques
- conceptual explanations
- best practices
- trade-off frameworks
- lessons that apply across projects

Ask:

- Would this help with a future unrelated project?
- Does it explain a reusable idea?
- Can it be understood without project context?
- Could multiple projects link to it?

If yes, store it as General Knowledge.

## Extraction Rule

When a project produces a reusable lesson:

1. Keep project-specific details in the Project Note.
2. Extract the reusable principle into a General Knowledge Note.
3. Link the Project Note to the General Knowledge Note.
4. Link the General Knowledge Note back only if the project is a useful example.

Do not turn every project detail into general knowledge.

Only extract lessons that reduce future thinking.

---

# Classification Tools

Use each classification tool for a different job.

| Tool | Best For | Avoid Using It For |
|---|---|---|
| Domain | Primary conceptual classification | Physical storage decisions |
| Tag | Search and filtering | Primary organization |
| Wikilink | Meaningful conceptual relationships | Replacing clear note titles |
| Metadata | Maintenance and structured filtering | Decorative properties |
| Alias | Alternate names and discovery | Hiding unclear names |

Use [[Standards/Vault-Structure]] for folder depth, top-level folder placement, and new folder rules.

When unsure about conceptual relationships, use links first.

---

# Review Checklist

Before finalizing taxonomy for a note, confirm:

- The note has one primary type.
- The note has one primary domain.
- The title names the concept clearly.
- The domain is based on future use, not origin.
- Existing notes were checked first.
- Cross-domain relationships are handled with links.
- Project-specific details are not mixed with general principles.
- General lessons are extracted only when reusable.
- Folder placement follows [[Standards/Vault-Structure]].
- Metadata and tags support retrieval without creating noise.

---

# Principle

Taxonomy should reduce future thinking.

If classification makes knowledge harder to find, maintain, or reuse, simplify it.
