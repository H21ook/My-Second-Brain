---
title: "Plan prompt"
type: resource
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - resource
  - imported
  - prompt-library
  - project/e-geree-v3
source_path: "D:/own/obsidian-vaults/E-Geree-v3-docs/awesome-prompts/Audit prompt.md"
---

Analyze the entire repository.

Focus on:
- Next.js App Router architecture
- Feature boundaries
- Shared components
- Server Actions
- API routes
- Supabase integration
- AI/RAG modules

Create a report containing:
1. Current architecture overview
2. Files larger than 300 lines
3. Components with multiple responsibilities
4. Duplicate logic
5. Dead code candidates
6. Folder structure issues
7. Naming inconsistencies
8. Refactoring opportunities ranked by ROI

DO NOT MODIFY ANY FILES.

ONLY CREATE E-Geree-v3-docs/architecture-audit.md

# Plan prompt

Using E-Geree-v3-docs/architecture-audit.md

Design a scalable architecture for this project.

Requirements:
- Next.js App Router
- Feature-first architecture
- Shared UI layer
- Shared lib layer
- Supabase
- AI agent modules
- Future SaaS scalability

Generate:

E-Geree-v3-docs/target-architecture.md

Include:
- folder tree
- migration plan
- risks
- priorities

Do not modify application code.

# Refactoring Tickets

Using  architecture-audit.md and target-architecture.md  from my obsidian vaults
  
Generate a refactoring backlog.  
  
Split all work into tickets.  
  
Each ticket must contain:  
  
- goal  
- affected files  
- expected outcome  
- risk level  
- estimated complexity  
  
Save as E-Geree-v3-docs/refactor-backlog.md

# Refactor Git rules
Үou are performing a large-scale repository refactor.

Rules:

Create and use ONLY the current branch.
Never create, switch, merge, rebase, or delete branches.

Current branch:
refactor/architecture-cleanup

Work ticket-by-ticket.
After completing each ticket:
Run lint
Run typecheck
Run tests if available
Fix any issues introduced by the ticket
Review the diff before committing.
Create ONE atomic commit per ticket.
Use Conventional Commits format.

Examples:

refactor(auth): extract authentication service

refactor(chat): split chat page into feature modules

refactor(shared): consolidate duplicate utilities

After every commit output:
Ticket ID
Files changed
Commit hash
Commit message
Remaining tickets
Never combine multiple tickets into one commit.
Never continue to the next ticket until the current ticket has been successfully committed.
If a ticket is too large, split it into sub-tickets and commit each sub-ticket separately.

Workflow:

For each ticket:

STEP 1:
Analyze ticket scope

STEP 2:
Implement changes

STEP 3:
Run validation

STEP 4:
Review git diff

STEP 5:
Create commit

STEP 6:
Report results

Then continue with the next ticket.

Begin with the first ticket from E-Geree-v3-docs/refactor-backlog.md.

# Execute ticket

Execute TICKET-001 only.

Rules:
- Maximum 15 files changed
- Run lint
- Run typecheck
- Do not touch unrelated code
- Create progress report afterwards
  
  
  

	/graphify update ажиллуулах