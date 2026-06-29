---
title: "Karpathy LLM Wiki Reference"
type: resource
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - resource
  - ai
  - knowledge-management
  - second-brain
aliases:
  - LLM Wiki
  - Persistent LLM Wiki
source:
  - https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
accessed: 2026-06-30
related:
  - "[[Retrieval-Augmented-Generation]]"
  - "[[AI-Context-Engineering]]"
  - "[[Knowledge-Distillation]]"
  - "[[Knowledge-Linking]]"
  - "[[Atomic-Knowledge]]"
---

# Karpathy LLM Wiki Reference

## Summary

Karpathy's LLM Wiki pattern describes a knowledge system where an AI agent incrementally maintains a persistent markdown wiki instead of only retrieving raw documents at question time.

The central idea is that useful synthesis should compound. When new sources arrive, the agent should update existing pages, add cross-links, record contradictions, and keep the wiki current. The user curates sources and asks questions; the agent performs the maintenance work that makes the wiki durable.

## Core Pattern

The pattern has three layers:

- **Raw sources:** immutable source material such as articles, papers, notes, and files.
- **Wiki:** structured markdown pages maintained by the LLM.
- **Schema:** operating rules such as `AGENTS.md`, `CLAUDE.md`, or equivalent instructions that define how the wiki should be maintained.

This maps closely to this vault's structure:

```text
Raw capture or source
  -> Evaluate future value
  -> Distill into reusable notes
  -> Connect related notes
  -> Maintain over time
```

## Operating Workflows

### Ingest

When a new source is added, the agent should:

1. Read the source.
2. Extract useful takeaways.
3. Create or update the relevant summary page.
4. Update related concept pages.
5. Add links and metadata.
6. Record what changed when useful.

The important point is that source processing should update the existing knowledge graph, not create isolated summaries forever.

### Query

When answering questions, the agent should search existing wiki pages first. If the answer produces reusable analysis, that analysis can become a new note or an update to an existing note.

### Lint

The wiki should be checked periodically for:

- stale claims
- contradictions
- orphan notes
- missing concept pages
- weak cross-links
- unresolved research gaps

## Why It Matters

Traditional RAG retrieves chunks and re-synthesizes them for every question. The LLM Wiki pattern adds a durable intermediate layer where synthesis is saved, linked, and improved.

For a Second Brain, this means the goal is not only retrieval. The goal is accumulated understanding.

## Practical Use in This Vault

Use this source as support for the existing vault operating model:

- raw notes should not remain in `Inbox/`
- source captures should become `Resources/` or distilled `Knowledge/`
- durable concepts should be linked across notes
- AI assistants should update existing notes instead of duplicating ideas
- system instructions should define how agents maintain the vault

## Related Notes

- [[Retrieval-Augmented-Generation|Retrieval-Augmented Generation]]
- [[AI-Context-Engineering|AI Context Engineering]]
- [[Knowledge-Distillation|Knowledge Distillation]]
- [[Knowledge-Linking|Knowledge Linking]]
- [[Atomic-Knowledge|Atomic Knowledge]]
- [[06-AI-Working-Guide|AI Working Guide]]

## References

- Andrej Karpathy, "LLM Wiki" gist.

## Personal Insights

This source strengthens the distinction between raw retrieval and maintained knowledge.

For this vault, RAG should be a retrieval layer over an already distilled knowledge base, not a substitute for maintaining the knowledge base itself.
