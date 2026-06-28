---
title: Decision Records
type: knowledge
status: active
created: 2026-06-28
updated: 2026-06-28
tags:
  - knowledge
  - knowledge-management
  - decision-making
aliases:
  - Decision Reports
  - Architecture Decision Records
  - ADRs
related:
  - "[[Knowledge-Distillation]]"
  - "[[Atomic-Knowledge]]"
  - "[[Evergreen-Knowledge]]"
  - "[[Knowledge-Linking]]"
  - "[[System/Templates/Decision-Note-Template]]"
---

# Decision Records

## Summary

Decision Records are permanent documents that preserve important decisions and the reasoning behind them.

Their purpose is not to record every discussion.

Their purpose is to capture why a particular choice was made, what alternatives were considered, and under what conditions that decision should change.

Decision Records reduce repeated thinking and help both humans and AI build upon previous reasoning.

---

## Problem

Important decisions are often forgotten.

Months later, people may remember what was chosen, but not:

- why it was chosen
- what alternatives existed
- what trade-offs were accepted
- what assumptions were made

Without this context, the same discussions happen repeatedly.

Teams may also reverse good decisions simply because the original reasoning has been lost.

---

## When to Use

Create a Decision Record when making a decision that will likely influence future work.

Examples include:

- selecting a technology
- defining an architecture
- choosing a database
- adopting a coding standard
- changing a workflow
- introducing a new process
- making a long-term business decision

A Decision Record should answer questions that future contributors are likely to ask.

---

## When Not to Use

Do not create Decision Records for:

- everyday tasks
- temporary experiments
- personal preferences with no long-term impact
- routine implementation details
- discussions that never resulted in a decision

Only decisions with lasting value should be preserved.

---

## Core Concepts

A Decision Record preserves reasoning rather than conversation history.

Every meaningful decision should explain:

- What problem existed?
- What options were considered?
- Why was one option selected?
- What trade-offs were accepted?
- When should the decision be reconsidered?

The objective is to preserve the thinking process, not the entire conversation.

---

## How It Works

```text
Problem
  -> Identify Options
  -> Evaluate Trade-offs
  -> Choose One Direction
  -> Record the Reasoning
  -> Review When Conditions Change
```

Decision Records are living documents.

They may evolve when better information becomes available, but previous reasoning should remain understandable.

---

## Best Practices

- Record decisions as soon as they are made.
- Focus on reasoning instead of discussion history.
- Explain rejected alternatives briefly.
- Describe important trade-offs.
- Define conditions that would justify changing the decision.
- Link related Knowledge Notes and Project Notes.
- Use [[System/Templates/Decision-Note-Template|Decision Note Template]] for important decisions.

---

## Common Mistakes

### Recording conversations instead of decisions

Decision Records should preserve conclusions, not meeting transcripts.

### Explaining only the final choice

Future readers also need to understand why other options were rejected.

### Never updating decisions

Some decisions become obsolete.

Update the status when a decision is replaced or deprecated rather than deleting its history.

### Recording insignificant choices

Too many Decision Records create unnecessary maintenance.

Only preserve decisions that are expected to matter in the future.

---

## Trade-offs

Advantages:

- prevents repeated debates
- improves long-term consistency
- preserves architectural reasoning
- helps AI understand historical context
- speeds up onboarding
- makes future decisions more informed

Disadvantages:

- requires discipline to maintain
- adds documentation effort
- not every decision deserves a record

The value increases over time as more important decisions accumulate.

---

## Examples

Poor:

```text
We decided to use PostgreSQL.
```

This explains what happened but not why.

Better:

```text
Decision:
Use PostgreSQL.

Reason:
Strong relational capabilities, mature ecosystem, and excellent support for Supabase integration.

Alternatives:
- MySQL
- MongoDB

Trade-offs:
More complex schema design, but stronger consistency and SQL support.
```

Another example:

```text
Decision:
Adopt Next.js App Router.

Reason:
Improves server rendering, layouts, and long-term maintainability.

Alternatives:
- Pages Router

Trade-offs:
Learning curve, but better architecture for future development.
```

---

## Related Notes

- [[Knowledge-Distillation|Knowledge Distillation]]
- [[Atomic-Knowledge|Atomic Knowledge]]
- [[Evergreen-Knowledge|Evergreen Knowledge]]
- [[Knowledge-Linking|Knowledge Linking]]
- [[02-Knowledge-Principles|Knowledge Principles]]
- [[System/Templates/Decision-Note-Template|Decision Note Template]]

---

## References

- Architecture Decision Records
- Michael Nygard, Documenting Architecture Decisions
- Tiago Forte, Building a Second Brain

---

## Personal Insights

Every important project eventually reaches a point where people ask:

> Why did we decide to do it this way?

If the answer exists only in memory or old chat conversations, the knowledge has already begun to disappear.

Decision Records transform temporary discussions into long-term organizational knowledge.

The greatest value of a Decision Record is preserving the reasoning that future humans and AI can build upon without repeating the same thinking.
