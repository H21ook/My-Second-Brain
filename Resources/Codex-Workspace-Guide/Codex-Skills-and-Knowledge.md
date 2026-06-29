---
title: "Codex Skills & Knowledge System"
type: resource
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - resource
  - imported
  - codex
  - ai-workflow
source_path: "D:/own/obsidian-vaults/Codex-Workspace-Guide/01-CODEX-SKILLS-AND-KNOWLEDGE.md"
---

# Codex Skills & Knowledge System

## Skill гэж юу вэ

Skill нь Codex-д тухайн сэдвийн туршлагатай мэргэжилтэн шиг ажиллах контекст өгдөг.

Examples:

- Next.js Architect
- Supabase Expert
- Database Reviewer
- Documentation Writer

---
## Create a skill

If you already know the workflow and it’s easier to show than describe, use [Record & Replay](https://developers.openai.com/codex/record-and-replay). Codex records the workflow, inspects the steps, and drafts a reusable skill from the demonstration.

If you want to describe the skill instead, use the built-in creator:

```
$skill-creator
```

The creator asks what the skill does, when it should trigger, and whether it should stay instruction-only or include scripts. Instruction-only is the default.

You can also create a skill manually by creating a folder with a `SKILL.md` file:

```
---
name: skill-name
description: Explain exactly when this skill should and should not trigger.
---

Skill instructions for Codex to follow.
```

Codex detects skill changes automatically. If an update doesn’t appear, restart Codex.

## Where to save skills

Codex reads skills from repository, user, admin, and system locations. For repositories, Codex scans `.agents/skills` in every directory from your current working directory up to the repository root. If two skills share the same `name`, Codex doesn’t merge them; both can appear in skill selectors.

|Skill Scope|Location|Suggested use|
|---|---|---|
|`REPO`|`$CWD/.agents/skills`  <br>Current working directory: where you launch Codex.|If you’re in a repository or code environment, teams can check in skills relevant to a working folder. For example, skills only relevant to a microservice or a module.|
|`REPO`|`$CWD/../.agents/skills`  <br>A folder above CWD when you launch Codex inside a Git repository.|If you’re in a repository with nested folders, organizations can check in skills relevant to a shared area in a parent folder.|
|`REPO`|`$REPO_ROOT/.agents/skills`  <br>The topmost root folder when you launch Codex inside a Git repository.|If you’re in a repository with nested folders, organizations can check in skills relevant to everyone using the repository. These serve as root skills available to any subfolder in the repository.|
|`USER`|`$HOME/.agents/skills`  <br>Any skills checked into the user’s personal folder.|Use to curate skills relevant to a user that apply to any repository the user may work in.|
|`ADMIN`|`/etc/codex/skills`  <br>Any skills checked into the machine or container in a shared, system location.|Use for SDK scripts, automation, and for checking in default admin skills available to each user on the machine.|
|`SYSTEM`|Bundled with Codex by OpenAI.|Useful skills relevant to a broad audience such as the skill-creator and plan skills. Available to everyone when they start Codex.|

Example:

```text
skills/
├── nextjs.md
├── supabase.md
├── facebook.md
├── gemini.md
└── documentation.md
```

---

# Skill Template

```md
# Next.js Architect

Role:
Senior Next.js Architect

Rules:

- App Router only
- TypeScript strict
- Server Components first
- Avoid unnecessary client components
- Use Server Actions when appropriate

Checklist:

- Performance
- Security
- Maintainability
- Scalability
```

---

# Project Context

Every project should contain:

```text
project-context.md
```

Example:

```md
# AI Sales Assistant SaaS

Purpose:

AI chatbot platform for Mongolian online stores.

Stack:

- Next.js
- TypeScript
- Supabase
- Gemini
- Redis

Architecture:

Multi-tenant SaaS

Roles:

- Superadmin
- Store Owner
- Store Admin
```

---

# Recommended Skills

## General

- Architect
- Reviewer
- Debugger
- Refactorer
- Documentation Writer

## Web

- Next.js
- React
- Tailwind
- shadcn/ui
- Supabase

## AI

- Prompt Engineer
- RAG Architect
- Gemini Specialist

---

# Knowledge Base

```text
knowledge/
│
├── architecture/
├── adr/
├── database/
├── prompts/
└── notes/
```

Purpose:

Long-term memory for projects.