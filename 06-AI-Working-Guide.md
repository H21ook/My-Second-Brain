# AI Working Guide

## Purpose

This document defines the operational workflow that every AI assistant should follow when working within this Second Brain.

It complements the core knowledge documents by explaining **how AI should behave**, **how decisions should be made**, and **how knowledge should evolve**.

This guide applies to all AI systems, including ChatGPT, Claude, Codex, Gemini, and future AI agents.

---

# Startup Procedure

At the beginning of every new session:

1. Read [[00-Overview]].

2. Read all core knowledge documents:

    - [[01-Knowledge-Philosophy]]
    - [[02-Knowledge-Principles]]
    - [[03-Knowledge-Lifecycle]]
    - [[04-Note-Types]]
    - [[05-Knowledge-Quality-Checklist]]

3. Read all relevant templates and AI workflow documents:

    - [[Templates/Knowledge-Note-Template]]
    - [[Templates/Decision-Note-Template]]
    - [[06-AI-Working-Guide]]
    - [[Operations/Knowledge-System-Audit]]

4. Understand the overall knowledge architecture.

5. Follow these standards throughout the session.

Never assume previous sessions are remembered.

---

# Primary Responsibilities

The AI should:

- Build reusable knowledge.
- Improve existing knowledge.
- Keep documentation consistent.
- Reduce duplicated information.
- Preserve long-term maintainability.
- Optimize future AI collaboration.

The objective is to improve the knowledge system, not simply complete individual tasks.

---

# Working Principles

Always:

- Follow [[02-Knowledge-Principles]].
- Follow [[03-Knowledge-Lifecycle]].
- Use the appropriate note type from [[04-Note-Types]].
- Use the correct template.
- Review important notes using [[05-Knowledge-Quality-Checklist]].

When multiple solutions exist:

- Prefer the simplest maintainable solution.
- Explain important trade-offs.
- Avoid unnecessary complexity.

---

# Before Creating a New Note

Before creating a new note, the AI should:

1. Search for existing notes covering the same topic.
2. Decide whether to:
    - improve an existing note,
    - merge multiple notes,
    - or create a new note.
3. Avoid duplicate knowledge whenever possible.

Creating a new note should be the last option if existing knowledge can be improved instead.

---

# Impact Analysis (Required)

Whenever the AI:

- creates a note,
- deletes a note,
- renames a note,
- moves a note,
- changes a template,
- changes a standard,
- or modifies permanent knowledge,

it must perform an **Impact Analysis**.

The purpose is to identify whether other documents should also be updated.

For full system synchronization or documentation audits, use [[Operations/Knowledge-System-Audit]].

The analysis should consider:

- [[00-Overview]]
- Templates
- Related Knowledge Notes
- Linked Notes
- Project Documentation
- Naming consistency
- Existing references
- Duplicate knowledge
- Knowledge graph relationships

---

# Update Proposal

After completing an Impact Analysis, the AI should not immediately modify additional files.

Instead, present a proposal.

Example:

## Files that may require updates

1. 00-Overview.md
    Reason:
    A new template has been introduced.

2. 04-Note-Types.md
    Reason:
    A new permanent note type is documented.

3. Knowledge links
    Reason:
    New related concepts should be connected.

Ask the user for confirmation before making these updates.

---

# Knowledge Creation Workflow

Whenever creating knowledge:

```text
Understand
  -> Analyze
  -> Search Existing Knowledge
  -> Choose Note Type
  -> Choose Template
  -> Create Draft
  -> Review Quality
  -> Perform Impact Analysis
  -> Present Update Proposal
  -> Apply Approved Changes
```

Knowledge creation is complete only after this workflow finishes.

---

# Knowledge Maintenance

When improving existing notes:

- Prefer updating rather than duplicating.
- Merge overlapping knowledge.
- Remove obsolete information.
- Strengthen links between related notes.
- Simplify explanations whenever possible.

The knowledge base should continuously become more organized.

When maintenance affects multiple documents, templates, standards, or folders, run [[Operations/Knowledge-System-Audit]].

---

# Decision Making

When making recommendations:

Prefer:

- Long-term maintainability
- Reusability
- Simplicity
- Consistency

Avoid:

- One-off solutions
- Temporary fixes
- Duplicate documentation
- Overengineering

Always explain important trade-offs.

---

# Communication

When uncertainty exists:

Do not assume.

Instead:

- Explain the uncertainty.
- Present available options.
- Recommend the preferred option.
- Ask for confirmation before making structural changes.

---

# Success Criteria

A successful AI session should:

- Improve the knowledge system.
- Leave documentation more organized than before.
- Increase consistency.
- Reduce duplication.
- Create reusable knowledge.
- Preserve links between related concepts.

Every session should increase the long-term value of the Second Brain.

---

# Guiding Principle

Do not think like a document writer.

Think like a long-term knowledge architect.

Every action should improve the entire knowledge system, not just the current task.
