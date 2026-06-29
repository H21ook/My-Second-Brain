---
title: "AI Workspace Setup Guide"
type: resource
status: draft
created: 2026-06-30
updated: 2026-06-30
tags:
  - resource
  - imported
  - codex
  - ai-workflow
source_path: "D:/own/obsidian-vaults/Codex-Workspace-Guide/00-AI-WORKSPACE-SETUP.md"
---

# AI Workspace Setup Guide

## Зорилго

Энэ гарын авлагын зорилго нь шинэ компьютер дээр дараах орчныг бүрэн сэргээхэд оршино.

- Codex CLI
- Codex Desktop
- Obsidian
- Graphify
- Superpowers
- Custom Skills
- MCP Servers
- VSCode Extensions
- AI Knowledge Base

---

# 1. Prerequisites

## Windows

Install:

- Git
- Node.js LTS
- VSCode
- Obsidian
- Docker Desktop

```powershell
winget install Git.Git
winget install OpenJS.NodeJS
winget install Microsoft.VisualStudioCode
winget install Obsidian.Obsidian
winget install Docker.DockerDesktop
```

## Mac

```bash
brew install git
brew install node
brew install --cask visual-studio-code
brew install --cask obsidian
brew install --cask docker
```

---

# 2. Install Codex CLI

```bash
npm install -g @openai/codex
```

Verify:

```bash
codex --version
```

---

# 3. Install Codex Desktop

Install latest version.

Login using OpenAI account.

---

# 4. Install Obsidian

Create vault:

```text
knowledge/
└── obsidian-vault/
```

Recommended Plugins:

- Dataview
- Excalidraw
- Tasks
- Templater
- Omnisearch
- Advanced Tables

---

# 5. Install Graphify

Purpose:

- Architecture visualization
- Dependency mapping
- Project exploration

Recommended use:

- Source code analysis
- Knowledge graph generation

### Install

**macOS quick install (Homebrew):**

```
brew install python@3.12 uv
```


**Windows quick install:**

```
winget install astral-sh.uv
```

**Windows Manually**

If you installed via plain `pip` and receive a `graphify: command not found` error, you must add the Python scripts directory manually.

1. Press the **Windows Key** and type **Environment Variables**.
2. Click on **Edit the system environment variables**.
3. Click the **Environment Variables...** button at the bottom of the window.
4. Under **System variables** or **User variables**, scroll down and select **Path**, then click **Edit**.
5. Click **New** and paste the exact path to your Python Scripts folder (usually `C:\Users\<username>\AppData\Roaming\Python\Python3xx\Scripts` or `C:\Program Files\Python3xx\Scripts`).
6. Click **OK** on all open windows, and restart your terminal for the changes to take effect.

**Step 1 — install the package:**

```shell
# Recommended (uv puts graphify on PATH automatically):
uv tool install graphifyy

# Alternatives:
pipx install graphifyy
pip install graphifyy  # may need PATH setup — see note below
```

**Step 2 — register the skill with your AI assistant:**

```shell
graphify install
```

That's it. Open your AI assistant and type `/graphify .`

To install the assistant skill into the current repository instead of your user profile, add `--project`:

```shell
graphify install --project
graphify install --project --platform codex
```

---

# 6. Install Superpowers

Purpose:

- Prompt libraries
- Reusable workflows
- AI productivity enhancement

### Codex App

Superpowers is available via the [official Codex plugin marketplace](https://github.com/openai/plugins).

- In the Codex app, click on Plugins in the sidebar.
- You should see `Superpowers` in the Coding section.
- Click the `+` next to Superpowers and follow the prompts.

### Codex CLI

Superpowers is available via the [official Codex plugin marketplace](https://github.com/openai/plugins).

- Open the plugin search interface:
    
    ```shell
    /plugins
    ```
    
- Search for Superpowers:
    
    ```shell
    superpowers
    ```
    
- Select `Install Plugin`.

---

# 7. Configure MCP Servers

Recommended:

- Filesystem
- GitHub
- PostgreSQL
- Supabase
- Browser

Store configs in:

```text
codex/mcp/
```

### Configure with the CLI

#### Add an MCP server

```
codex mcp add <server-name> --env VAR1=VALUE1 --env VAR2=VALUE2 -- <stdio server-command>
```

For example, to add Context7 (a free MCP server for developer documentation), you can run the following command:

```
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

---

# 8. Install VSCode Extensions

Export:

```bash
code --list-extensions > extensions.txt
```

Restore:

```bash
cat extensions.txt | xargs -L 1 code --install-extension
```

Recommended:

- ESLint
- Tailwind CSS
- Prisma
- GitLens
- Error Lens
- Markdown All In One

---

# 9. Workspace Structure

```text
workspace/
│
├── projects/
├── knowledge/
├── codex/
├── scripts/
└── templates/
```