# Decision Note Template

## Purpose

This template defines the standard format for recording an important Decision Note.

Decision Notes preserve important choices and the reasoning behind them.

They prevent future contributors and AI assistants from repeating the same discussion without understanding why the original decision was made.

Before creating a Decision Note, ensure that the decision is important enough to preserve and belongs to the **Decision** note type.

---

# Instructions for AI

Before writing this note:

- Read [[00-Overview]] to understand the knowledge system.
- Follow [[02-Knowledge-Principles]] while writing.
- Confirm that the note belongs to the Decision note type in [[04-Note-Types]].
- Follow [[06-AI-Working-Guide]] when creating or changing permanent knowledge.
- Preserve the final decision and reasoning, not the full discussion.
- Include rejected alternatives only when they explain meaningful trade-offs.
- Link related Knowledge, Project, Reference, or Decision Notes.

---

# Title

Use a concise title that clearly states the decision.

Prefer titles that start with the decision subject.

Examples:

- Use Supabase for Authentication
- Adopt Next.js App Router
- Store Vector Embeddings in Postgres

---

# Metadata

Use frontmatter when creating the actual note.

Example:

```yaml
---
title:
type: decision
status: proposed
created:
updated:
decision_date:
tags:
  - decision
aliases:
related:
supersedes:
superseded_by:
---
```

Follow [[08-Metadata-Standards]] when adding or changing metadata.

---

# Decision

State the final decision in one or two clear sentences.

Answer:

- What was decided?
- What is the chosen direction?
- Who or what does this decision affect?

---

# Status

Record the current status of the decision.

Use one of:

- Proposed
- Accepted
- Superseded
- Rejected
- Deprecated

If the decision is superseded, link to the newer decision note.

---

# Date

Record the decision date.

Use ISO format when possible:

```text
YYYY-MM-DD
```

---

# Context

Explain the situation that required a decision.

Include:

- the problem or opportunity
- relevant constraints
- project or system context
- assumptions that shaped the decision

Avoid unnecessary conversation history.

---

# Problem

Describe the specific problem this decision solves.

Answer:

- What was unclear, blocked, risky, or inefficient?
- Why was a decision needed now?
- What would happen if no decision was made?

---

# Goals

List the outcomes this decision should support.

Good goals are specific and decision-relevant.

Examples:

- Reduce repeated implementation debate.
- Improve maintainability.
- Support faster AI-assisted development.
- Minimize operational complexity.

---

# Options Considered

List the meaningful options that were considered.

For each option, include a short explanation of the trade-off.

## Option A

Describe the option.

Pros:

- Benefit:

Cons:

- Cost:

## Option B

Describe the option.

Pros:

- Benefit:

Cons:

- Cost:

## Option C

Describe the option.

Pros:

- Benefit:

Cons:

- Cost:

---

# Reasoning

Explain why the chosen option was selected.

Focus on reusable reasoning:

- decision criteria
- constraints
- trade-offs
- risks accepted
- why rejected alternatives were not chosen

This section should help future contributors understand the logic without reopening the entire debate.

---

# Consequences

Document what changes because of this decision.

Include:

- expected benefits
- costs or limitations
- new responsibilities
- effects on future work
- technical, organizational, or workflow impact

---

# Trade-offs

Every decision accepts trade-offs.

Document both advantages and disadvantages.

Avoid describing the decision as universally best.

---

# Risks

List risks introduced by this decision.

For each risk, include how it should be monitored or reduced.

Examples:

- Operational risk
- Maintenance risk
- Vendor lock-in
- Performance limitations
- Knowledge gaps

---

# Reversal Conditions

Define when this decision should be revisited or changed.

Answer:

- What new evidence would make this decision wrong?
- What conditions would justify replacing it?
- What failure signals should future contributors watch for?

---

# Implementation Notes

Document practical guidance for applying the decision.

Include:

- required follow-up work
- affected files, systems, or workflows
- migration notes
- constraints future work must respect

Keep this section concise and actionable.

---

# Related Notes

Link related knowledge.

Possible relationships include:

- supporting Knowledge Notes
- affected Project Notes
- superseded Decision Notes
- related Reference Notes
- relevant templates or checklists

Examples:

- [[02-Knowledge-Principles]]
- [[04-Note-Types]]
- [[05-Knowledge-Quality-Checklist]]
- [[01-Knowledge-Philosophy#Decision Philosophy]]
- [[06-AI-Working-Guide]]

---

# References

List external resources that influenced the decision.

Prefer:

- official documentation
- specifications
- research papers
- trusted technical references

Summarize external information instead of copying it.

---

# Review Checklist

Before completing this note, confirm:

- The final decision is clear.
- The reason for the decision is preserved.
- The problem and context are understandable without the original discussion.
- Meaningful alternatives and trade-offs are documented.
- Consequences and risks are explicit.
- Reversal conditions are defined.
- Related notes are linked.
- The note follows [[02-Knowledge-Principles]].
- The note satisfies [[05-Knowledge-Quality-Checklist]].
- Impact analysis is completed using [[06-AI-Working-Guide]].

---

# Continuous Improvement

Decision Notes can be updated when understanding improves, but the original reasoning should remain clear.

When a decision changes:

- update the status
- explain what changed
- link to the newer decision if superseded
- preserve enough context for historical understanding

The goal is not to defend old decisions.

The goal is to preserve decision quality over time.
