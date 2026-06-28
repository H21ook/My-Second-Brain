# Note Types

## Purpose

Not every note serves the same purpose.

Each note should belong to a clearly defined type.

Using consistent note types makes knowledge easier to organize, maintain, search, and reuse by both humans and AI.

---

# Overview

|Type|Purpose|Lifetime|
|---|---|---|
|Inbox|Temporary captured information|Very Short|
|Daily|Daily work and progress|Short|
|Knowledge|Permanent reusable knowledge|Long|
|Decision|Important architectural or business decisions|Long|
|Project|Project-specific documentation|Project Lifetime|
|Reference|External documentation summary|Long|
|Template|Reusable structures and prompts|Long|
|Resource|Collections of useful links, books, tools|Medium|
|Archive|Historical notes|Long|

---

# 1. Inbox Note

Purpose

Quickly capture information before it is forgotten.

Characteristics

- Unorganized
    
- Temporary
    
- Raw information
    
- No formatting required
    

Examples

- Random ideas
    
- AI conversation snippets
    
- Meeting notes
    
- Interesting links
    

Rule

Inbox notes should eventually be:

- distilled into Knowledge Notes
    
- moved into Project Notes
    
- archived
    
- deleted
    

---

# 2. Daily Note

Purpose

Track daily work.

Contents may include

- completed tasks
    
- discoveries
    
- debugging
    
- meetings
    
- progress
    
- temporary thoughts
    

Rule

Daily Notes are not permanent knowledge.

Useful information should later be extracted into permanent notes.

---

# 3. Knowledge Note

Purpose

Store evergreen reusable knowledge.

Characteristics

- Well organized
    
- Self-contained
    
- AI friendly
    
- Continuously improved
    

Examples

- Next.js Server Components
    
- Prompt Engineering
    
- Authentication Patterns
    
- RAG Architecture
    

These are the core of the Second Brain.

Use [[Templates/Knowledge-Note-Template]] when creating a permanent Knowledge Note.

---

# 4. Decision Note

Purpose

Record important decisions.

Every decision should include

- Decision
    
- Context
    
- Reason
    
- Alternatives
    
- Consequences
    

Examples

- Why Supabase was selected
    
- Why App Router is used
    
- Why Redis is required
    

Decision notes prevent repeating the same discussions.

Use [[Templates/Decision-Note-Template]] when recording an important decision.

---

# 5. Project Note

Purpose

Store project-specific knowledge.

Examples

- Architecture
    
- Roadmap
    
- API Design
    
- Database Schema
    
- Deployment
    
- TODOs
    

Unlike Knowledge Notes, these are usually tied to one project.

---

# 6. Reference Note

Purpose

Summarize external sources.

Sources may include

- Books
    
- Documentation
    
- Videos
    
- Research papers
    
- Blog posts
    

Reference Notes should summarize information instead of copying it.

Always add your own understanding.

---

# 7. Template Note

Purpose

Store reusable assets.

Examples

- Prompt templates
    
- Code templates
    
- Document templates
    
- AI instructions
    
- Checklists
    

Templates save future time.

AI workflow standards such as [[06-AI-Working-Guide]] are Template Notes because they provide reusable operating instructions.

Operational procedures such as [[Operations/Knowledge-System-Audit]] are also Template Notes because they define repeatable workflows.

Root standards such as [[07-Naming-Conventions]] define reusable rules that apply across note types.

---

# 8. Resource Note

Purpose

Maintain curated collections.

Examples

- Useful websites
    
- Learning resources
    
- AI tools
    
- Libraries
    
- Books
    
- Videos
    

Resources point to useful information.

Knowledge Notes explain it.

---

# 9. Archive Note

Purpose

Keep historical information.

Archive when

- no longer actively maintained
    
- replaced
    
- completed
    
- obsolete
    

Archive is not deleted knowledge.

It remains available for future reference.

---

# Relationships Between Note Types

```text
Inbox
    │
    ▼
Daily
    │
    ▼
Reference
    │
    ▼
Knowledge
    │
    ├────────► Template
    │
    ├────────► Decision
    │
    └────────► Project

Completed
    │
    ▼
Archive
```

---

# Rules

Every note should have exactly one primary type.

Knowledge Notes are the most valuable asset of the Second Brain.

Temporary notes should eventually disappear.

Permanent knowledge should continuously improve over time.

The goal is not to create more notes.

The goal is to create more reusable knowledge.
