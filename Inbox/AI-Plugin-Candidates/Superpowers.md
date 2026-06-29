# Superpowers

[](https://github.com/obra/Superpowers#superpowers)

Superpowers is a complete software development methodology for your coding agents, built on top of a set of composable skills and some initial instructions that make sure your agent uses them.

## We're Hiring!

[](https://github.com/obra/Superpowers#were-hiring)

We're hiring someone to help out full time with Superpowers community and code work. You can read about the job at [https://primeradiant.com/jobs/superpowers-community-engineer/](https://primeradiant.com/jobs/superpowers-community-engineer/) If this sounds like someone you know, definitely send them our way.

## Quickstart

[](https://github.com/obra/Superpowers#quickstart)

Give your agent Superpowers: [Claude Code](https://github.com/obra/Superpowers#claude-code), [Antigravity](https://github.com/obra/Superpowers#antigravity), [Codex App](https://github.com/obra/Superpowers#codex-app), [Codex CLI](https://github.com/obra/Superpowers#codex-cli), [Cursor](https://github.com/obra/Superpowers#cursor), [Factory Droid](https://github.com/obra/Superpowers#factory-droid), [Gemini CLI](https://github.com/obra/Superpowers#gemini-cli), [GitHub Copilot CLI](https://github.com/obra/Superpowers#github-copilot-cli), [Kimi Code](https://github.com/obra/Superpowers#kimi-code), [OpenCode](https://github.com/obra/Superpowers#opencode), [Pi](https://github.com/obra/Superpowers#pi).

## How it works

[](https://github.com/obra/Superpowers#how-it-works)

It starts from the moment you fire up your coding agent. As soon as it sees that you're building something, it _doesn't_ just jump into trying to write code. Instead, it steps back and asks you what you're really trying to do.

Once it's teased a spec out of the conversation, it shows it to you in chunks short enough to actually read and digest.

After you've signed off on the design, your agent puts together an implementation plan that's clear enough for an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing to follow. It emphasizes true red/green TDD, YAGNI (You Aren't Gonna Need It), and DRY.

Next up, once you say "go", it launches a _subagent-driven-development_ process, having agents work through each engineering task, inspecting and reviewing their work, and continuing forward. It's not uncommon for your agent to work autonomously for a couple hours at a time without deviating from the plan you put together.

There's a bunch more to it, but that's the core of the system. And because the skills trigger automatically, you don't need to do anything special. Your coding agent just has Superpowers.

## Commercial Services

[](https://github.com/obra/Superpowers#commercial-services)

If you're using Superpowers in enterprise and could benefit from commercial support, additional tooling, or managed spending, please don't hesitate to drop us a line at [sales@primeradiant.com](mailto:sales@primeradiant.com).

## Installation

[](https://github.com/obra/Superpowers#installation)

Installation differs by harness. If you use more than one, install Superpowers separately for each one.

### Claude Code

[](https://github.com/obra/Superpowers#claude-code)

Superpowers is available via the [official Claude plugin marketplace](https://claude.com/plugins/superpowers)

#### Official Marketplace

[](https://github.com/obra/Superpowers#official-marketplace)

- Install the plugin from Anthropic's official marketplace:
    
    ```shell
    /plugin install superpowers@claude-plugins-official
    ```
    

#### Superpowers Marketplace

[](https://github.com/obra/Superpowers#superpowers-marketplace)

The Superpowers marketplace provides Superpowers and some other related plugins for Claude Code.

- Register the marketplace:
    
    ```shell
    /plugin marketplace add obra/superpowers-marketplace
    ```
    
- Install the plugin from this marketplace:
    
    ```shell
    /plugin install superpowers@superpowers-marketplace
    ```
    

### Antigravity

[](https://github.com/obra/Superpowers#antigravity)

Install Superpowers as a plugin from this repository:

```shell
agy plugin install https://github.com/obra/superpowers
```

Antigravity runs the plugin's session-start hook, so Superpowers is active from the first message. Reinstall with the same command to update.

### Codex App

[](https://github.com/obra/Superpowers#codex-app)

Superpowers is available via the [official Codex plugin marketplace](https://github.com/openai/plugins).

- In the Codex app, click on Plugins in the sidebar.
- You should see `Superpowers` in the Coding section.
- Click the `+` next to Superpowers and follow the prompts.

### Codex CLI

[](https://github.com/obra/Superpowers#codex-cli)

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
    

### Cursor

[](https://github.com/obra/Superpowers#cursor)

- In Cursor Agent chat, install from marketplace:
    
    ```
    /add-plugin superpowers
    ```
    
- Or search for "superpowers" in the plugin marketplace.
    

### Factory Droid

[](https://github.com/obra/Superpowers#factory-droid)

- Register the marketplace:
    
    ```shell
    droid plugin marketplace add https://github.com/obra/superpowers
    ```
    
- Install the plugin:
    
    ```shell
    droid plugin install superpowers@superpowers
    ```
    

### Gemini CLI

[](https://github.com/obra/Superpowers#gemini-cli)

- Install the extension:
    
    ```shell
    gemini extensions install https://github.com/obra/superpowers
    ```
    
- Update later:
    
    ```shell
    gemini extensions update superpowers
    ```
    

### GitHub Copilot CLI

[](https://github.com/obra/Superpowers#github-copilot-cli)

- Register the marketplace:
    
    ```shell
    copilot plugin marketplace add obra/superpowers-marketplace
    ```
    
- Install the plugin:
    
    ```shell
    copilot plugin install superpowers@superpowers-marketplace
    ```
    

### Kimi Code

[](https://github.com/obra/Superpowers#kimi-code)

Superpowers is available in Kimi Code's plugin marketplace.

- Open Kimi Code's plugin manager:
    
    ```
    /plugins
    ```
    
- Go to `Marketplace` > `Superpowers` and install it.
    
- Or install directly from this repository:
    
    ```
    /plugins install https://github.com/obra/superpowers
    ```
    
- Detailed docs: [docs/README.kimi.md](https://github.com/obra/superpowers/blob/main/docs/README.kimi.md)
    

### OpenCode

[](https://github.com/obra/Superpowers#opencode)

OpenCode uses its own plugin install; install Superpowers separately even if you already use it in another harness.

- Tell OpenCode:
    
    ```
    Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
    ```
    
- Detailed docs: [docs/README.opencode.md](https://github.com/obra/superpowers/blob/main/docs/README.opencode.md)
    

### Pi

[](https://github.com/obra/Superpowers#pi)

Install Superpowers as a Pi package from this repository:

```shell
pi install git:github.com/obra/superpowers
```

For local development, run Pi with this checkout loaded as a temporary package:

```shell
pi -e /path/to/superpowers
```

The Pi package loads the Superpowers skills and a small extension that injects the `using-superpowers` bootstrap at session startup and again after compaction. Pi has native skills, so no compatibility `Skill` tool is required. Subagent and task-list tools remain optional Pi companion packages.

## The Basic Workflow

[](https://github.com/obra/Superpowers#the-basic-workflow)

1. **brainstorming** - Activates before writing code. Refines rough ideas through questions, explores alternatives, presents design in sections for validation. Saves design document.
    
2. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.
    
3. **writing-plans** - Activates with approved design. Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps.
    
4. **subagent-driven-development** or **executing-plans** - Activates with plan. Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality), or executes in batches with human checkpoints.
    
5. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR: write failing test, watch it fail, write minimal code, watch it pass, commit. Deletes code written before tests.
    
6. **requesting-code-review** - Activates between tasks. Reviews against plan, reports issues by severity. Critical issues block progress.
    
7. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, presents options (merge/PR/keep/discard), cleans up worktree.
    

**The agent checks for relevant skills before any task.** Mandatory workflows, not suggestions.

## What's Inside

[](https://github.com/obra/Superpowers#whats-inside)

### Skills Library

[](https://github.com/obra/Superpowers#skills-library)

**Testing**

- **test-driven-development** - RED-GREEN-REFACTOR cycle (includes testing anti-patterns reference)

**Debugging**

- **systematic-debugging** - 4-phase root cause process (includes root-cause-tracing, defense-in-depth, condition-based-waiting techniques)
- **verification-before-completion** - Ensure it's actually fixed

**Collaboration**

- **brainstorming** - Socratic design refinement
- **writing-plans** - Detailed implementation plans
- **executing-plans** - Batch execution with checkpoints
- **dispatching-parallel-agents** - Concurrent subagent workflows
- **requesting-code-review** - Pre-review checklist
- **receiving-code-review** - Responding to feedback
- **using-git-worktrees** - Parallel development branches
- **finishing-a-development-branch** - Merge/PR decision workflow
- **subagent-driven-development** - Fast iteration with two-stage review (spec compliance, then code quality)

**Meta**

- **writing-skills** - Create new skills following best practices (includes testing methodology)
- **using-superpowers** - Introduction to the skills system

## Philosophy

[](https://github.com/obra/Superpowers#philosophy)

- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success

Read [the original release announcement](https://blog.fsck.com/2025/10/09/superpowers/).

## Contributing

[](https://github.com/obra/Superpowers#contributing)

The general contribution process for Superpowers is below. Keep in mind that we don't generally accept contributions of new skills and that any updates to skills must work across all of the coding agents we support.

1. Fork the repository
2. Switch to the 'dev' branch
3. Create a branch for your work
4. Follow the `writing-skills` skill for creating and testing new and modified skills
5. Submit a PR, being sure to fill in the pull request template.

Skill-behavior tests use the drill eval harness from [superpowers-evals](https://github.com/prime-radiant-inc/superpowers-evals/), cloned into `evals/` — see `evals/README.md` for setup. Plugin-infrastructure tests live at `tests/` and run via the relevant `run-*.sh` or `npm test`.

See `skills/writing-skills/SKILL.md` for the complete guide.

## Updating

[](https://github.com/obra/Superpowers#updating)

Superpowers updates are somewhat coding-agent dependent, but are often automatic.

## License

[](https://github.com/obra/Superpowers#license)

MIT License - see LICENSE file for details

## Visual companion telemetry

[](https://github.com/obra/Superpowers#visual-companion-telemetry)

Because skills and plugins don't provide any feedback to creators, we have no idea how many of you are using Superpowers. By default, the Prime Radiant logo on brainstorming's optional visual companion feature is loaded from our website. It includes the version of Superpowers in use. It does not include any details about your project, prompt, or coding agent. We don't see your clicks or anything about what you're building. This helps us have a rough idea of how many folks are using Superpowers and which version of Superpowers they're using. It's 100% optional. To disable this, set the environment variable `SUPERPOWERS_DISABLE_TELEMETRY` to any true value. Superpowers also honors Claude Code's `DISABLE_TELEMETRY` and `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` opt-outs.

## Community

[](https://github.com/obra/Superpowers#community)

Superpowers is built by [Jesse Vincent](https://blog.fsck.com/) and the rest of the folks at [Prime Radiant](https://primeradiant.com/).

- **Discord**: [Join us](https://discord.gg/35wsABTejz) for community support, questions, and sharing what you're building with Superpowers
- **Issues**: [https://github.com/obra/superpowers/issues](https://github.com/obra/superpowers/issues)
- **Release announcements**: [Sign up](https://primeradiant.com/superpowers/) to get notified about new versions