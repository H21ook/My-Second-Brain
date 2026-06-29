---
title: "Using Codex in Real Projects"
type: resource
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - resource
  - imported
  - codex
  - ai-workflow
source_path: "D:/own/obsidian-vaults/Codex-Workspace-Guide/02-USING-CODEX-IN-REAL-PROJECTS.md"
---

# Using Codex in Real Projects

# Existing Project Workflow

## Step 1

Open project:

```bash
cd ai-sales-assistant
codex
```

## Step 2

Ask project analysis

Prompt:

```text
Analyze the entire project structure.
Generate architecture documentation.
Create missing documentation if needed.
```

---

## Step 3

Generate Knowledge Base

Prompt:

```text
Create Obsidian notes for:

- Architecture
- Database
- APIs
- Components
- Features

Store output inside docs/
```

---

## Step 4

Generate ADRs

Prompt:

```text
Analyze decisions and generate ADR documents.
```

---

# New Project Workflow

## Create Project

```bash
mkdir my-project
cd my-project
```

Create:

```text
project-context.md
```

Create:

```text
docs/
```

Create:

```text
knowledge/
```

---

## First Prompt

```text
Read project-context.md.

Act as a senior software architect.

Generate:

- Folder structure
- Database design
- API design
- Coding standards
- Documentation structure
```

---

# Daily Workflow

Morning:

```text
Read project-context.md.
Read architecture documents.

Summarize project status.
```

Before coding:

```text
Review architecture.
Suggest implementation plan.
```

After coding:

```text
Review all changes.

Check:

- bugs
- security
- performance
- architecture compliance
```

---

# Moving to Another Computer

## Step 1

Push workspace:

```bash
git push
```

Workspace:

```text
workspace/
```

contains:

- projects
- skills
- prompts
- obsidian vault
- knowledge

---

## Step 2

New computer

```bash
git clone
```

---

## Step 3

Run bootstrap

```bash
./bootstrap.sh
```

or

```powershell
.\bootstrap.ps1
```

---

## Step 4

Restore VSCode extensions

```bash
cat extensions.txt | xargs -L 1 code --install-extension
```

---

## Step 5

Open project

```bash
codex
```

Everything is restored.

---

# Golden Rule

Never start a project without:

- project-context.md
- architecture.md
- coding-standards.md
- docs/
- knowledge/

Codex becomes dramatically more useful when these files exist.