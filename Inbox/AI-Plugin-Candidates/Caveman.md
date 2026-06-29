A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill/plugin (also Codex, Gemini, Cursor, Windsurf, Cline, Copilot, 30+ more) that makes agent talk like caveman — cuts **~75% of output tokens**, keeps full technical accuracy. Brain still big. Mouth small.

## Before / After

[](https://github.com/juliusbrussee/caveman#before--after)

|   |   |
|---|---|
|### 🗣️ Normal Claude (69 tokens)<br><br>[](https://github.com/juliusbrussee/caveman#%EF%B8%8F-normal-claude-69-tokens)<br><br>> "The reason your React component is re-rendering is likely because you're creating a new object reference on each render cycle. When you pass an inline object as a prop, React's shallow comparison sees it as a different object every time, which triggers a re-render. I'd recommend using useMemo to memoize the object."|### [![rock](https://github.com/JuliusBrussee/caveman/raw/main/docs/assets/dancing-rock.svg)](https://github.com/JuliusBrussee/caveman/blob/main/docs/assets/dancing-rock.svg) Caveman Claude (19 tokens)<br><br>[](https://github.com/juliusbrussee/caveman#-caveman-claude-19-tokens)<br><br>> "New object ref each render. Inline object prop = new ref = re-render. Wrap in `useMemo`."|
|### 🗣️ Normal Claude<br><br>[](https://github.com/juliusbrussee/caveman#%EF%B8%8F-normal-claude)<br><br>> "Sure! I'd be happy to help you with that. The issue you're experiencing is most likely caused by your authentication middleware not properly validating the token expiry. Let me take a look and suggest a fix."|### [![rock](https://github.com/JuliusBrussee/caveman/raw/main/docs/assets/dancing-rock.svg)](https://github.com/JuliusBrussee/caveman/blob/main/docs/assets/dancing-rock.svg) Caveman Claude<br><br>[](https://github.com/juliusbrussee/caveman#-caveman-claude)<br><br>> "Bug in auth middleware. Token expiry check use `<` not `<=`. Fix:"|

**Same fix. 75% less word. Brain still big.**

```
┌─────────────────────────────────────┐
│  TOKENS SAVED          ████████ 75% │
│  TECHNICAL ACCURACY    ████████ 100%│
│  SPEED INCREASE        ████████ ~3x │
│  VIBES                 ████████ OOG │
└─────────────────────────────────────┘
```

Pick your level of grunt — `lite` (drop filler), `full` (default caveman), `ultra` (telegraphic), or `wenyan` (classical Chinese, even shorter). One command switch. Cost go down forever.

**Speak your tongue.** Caveman keep your language. You write Portuguese, caveman grunt Portuguese. Spanish, French, same. Compress the _style_, not the language. Code, command, error string stay exact.

> "Novo ref de objeto cada render. Prop inline = novo ref = re-render. Envolva com `useMemo`."

|   |
|---|
|### [![rock](https://github.com/JuliusBrussee/caveman/raw/main/docs/assets/dancing-rock.svg)](https://github.com/JuliusBrussee/caveman/blob/main/docs/assets/dancing-rock.svg) Like this trick? Now get whole agent — **caveman-code**<br><br>[](https://github.com/juliusbrussee/caveman#-like-this-trick-now-get-whole-agent--caveman-code)<br><br>This skill shrink what agent **say**. **[caveman-code](https://github.com/JuliusBrussee/caveman-code)** shrink **everything** — full terminal coding agent, caveman top to bottom. **~2× fewer tokens than Codex** on identical tasks. 20+ providers · plan mode · autopilot goal loop · MIT.<br><br>```shell<br>npm install -g @juliusbrussee/caveman-code<br>```<br><br>[**▶ Try caveman-code now →**](https://github.com/JuliusBrussee/caveman-code) — _why use many token when whole agent save_|

## Install

[](https://github.com/juliusbrussee/caveman#install)

One line. Find every agent. Install for each.

```shell
# macOS / Linux / WSL / Git Bash
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash

# Windows (PowerShell 5.1+)
irm https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.ps1 | iex
```

~30 seconds. Needs Node ≥18. Skip agent you no have. Safe to re-run.

**Trigger:** type `/caveman` or say "talk like caveman". Stop with "normal mode".

One agent only, manual command, or any of 30+ other agents → [**INSTALL.md**](https://github.com/JuliusBrussee/caveman/blob/main/INSTALL.md). Install break? Open agent, say _"Read CLAUDE.md and INSTALL.md, install caveman for me."_ Agent fix own brain.

## What You Get

[](https://github.com/juliusbrussee/caveman#what-you-get)

|Skill|What|
|---|---|
|`/caveman [lite\|full\|ultra\|wenyan]`|Compress every reply. Levels stick until session end.|
|`/caveman-commit`|Conventional Commit messages, ≤50 char subject. Why over what.|
|`/caveman-review`|One-line PR comments: `L42: 🔴 bug: user null. Add guard.`|
|`/caveman-stats`|Real session token usage + lifetime savings + USD. Tweetable line via `--share`.|
|`/caveman-compress <file>`|Rewrite memory file (e.g. `CLAUDE.md`) into caveman-speak. Cuts ~46% input tokens every session. Code/URLs/paths byte-preserved.|
|`caveman-shrink`|MCP middleware. Wraps any MCP server, compresses tool descriptions. [npm](https://www.npmjs.com/package/caveman-shrink).|
|`cavecrew-*`|Caveman subagents (investigator/builder/reviewer). ~60% fewer tokens than vanilla, main context lasts longer.|

**Statusline badge** — Claude Code shows `[CAVEMAN] ⛏ 12.4k` (lifetime tokens saved). Updates every `/caveman-stats` run. Set `CAVEMAN_STATUSLINE_SAVINGS=0` to silence.

Auto-activate every session: Claude Code, Codex, Gemini (built-in). Cursor / Windsurf / Cline / Copilot get always-on rule files via `--with-init`. Other agents trigger with `/caveman` per session. Full feature matrix in [INSTALL.md](https://github.com/JuliusBrussee/caveman/blob/main/INSTALL.md#what-you-get).

## Benchmarks

[](https://github.com/juliusbrussee/caveman#benchmarks)

Real token counts from the Claude API. Average **65% output reduction** across 10 prompts (range 22-87%).

|Task|Normal|Caveman|Saved|
|---|--:|--:|--:|
|Explain React re-render bug|1180|159|87%|
|Fix auth middleware token expiry|704|121|83%|
|Set up PostgreSQL connection pool|2347|380|84%|
|Explain git rebase vs merge|702|292|58%|
|Refactor callback to async/await|387|301|22%|
|Architecture: microservices vs monolith|446|310|30%|
|Review PR for security issues|678|398|41%|
|Docker multi-stage build|1042|290|72%|
|Debug PostgreSQL race condition|1200|232|81%|
|Implement React error boundary|3454|456|87%|
|**Average**|**1214**|**294**|**65%**|

Raw data and reproduction script: [`benchmarks/`](https://github.com/JuliusBrussee/caveman/blob/main/benchmarks). Three-arm eval harness (baseline / terse / skill) lives in [`evals/`](https://github.com/JuliusBrussee/caveman/blob/main/evals) — caveman compared against `Answer concisely.` not against verbose default, so the delta is honest.

**caveman-compress receipts** (real memory files):

|File|Original|Compressed|Saved|
|---|--:|--:|--:|
|`claude-md-preferences.md`|706|285|**59.6%**|
|`project-notes.md`|1145|535|**53.3%**|
|`claude-md-project.md`|1122|636|**43.3%**|
|`todo-list.md`|627|388|**38.1%**|
|`mixed-with-code.md`|888|560|**36.9%**|
|**Average**|**898**|**481**|**46%**|

Important

Caveman only affects output tokens — thinking/reasoning tokens untouched. Caveman no make brain smaller. Caveman make _mouth_ smaller. Biggest win is **readability and speed**, cost savings a bonus.

A March 2026 paper ["Brevity Constraints Reverse Performance Hierarchies in Language Models"](https://arxiv.org/abs/2604.00025) found that constraining large models to brief responses **improved accuracy by 26 points** on certain benchmarks. Verbose not always better. Sometimes less word = more correct.

## How It Work

[](https://github.com/juliusbrussee/caveman#how-it-work)

1. Install drop skill file in agent.
2. Skill tell agent: drop filler, keep substance, use fragments.
3. For Claude Code, hook also write tiny flag file each session — agent see flag, talk caveman from message one. No need say `/caveman`.
4. Stats command read Claude Code session log, count tokens saved, write number to statusline.
5. Caveman-compress sub-skill rewrite memory files (CLAUDE.md, project notes) so each session start with smaller context. Save tokens forever, not just one reply.

Maintainer detail (hook architecture, file ownership, CI sync) live in [CLAUDE.md](https://github.com/JuliusBrussee/caveman/blob/main/CLAUDE.md).

## Lobster, Meet Rock 🦞 [![rock](https://github.com/JuliusBrussee/caveman/raw/main/docs/assets/dancing-rock.svg)](https://github.com/JuliusBrussee/caveman/blob/main/docs/assets/dancing-rock.svg)

[](https://github.com/juliusbrussee/caveman#lobster-meet-rock--)

[**OpenClaw**](https://openclaw.ai/) the self-host gateway. One box, many agent inside (Claude Code, Codex, Pi, OpenCode), wired to your Slack / Discord / iMessage / Telegram / whatever. Tagline: _"The lobster way."_ Lobster strong. Lobster smart. Lobster also talk a lot.

Caveman teach lobster brevity — same canonical installer, scoped to one agent:

```shell
# macOS / Linux / WSL
curl -fsSL https://raw.githubusercontent.com/JuliusBrussee/caveman/main/install.sh | bash -s -- --only openclaw

# Windows (PowerShell): no Node? install Node ≥18 first, then
npx -y github:JuliusBrussee/caveman -- --only openclaw
```

Two thing happen, no more:

1. **Skill drop** at `~/.openclaw/workspace/skills/caveman/SKILL.md` — spec-correct frontmatter (`version`, `always: true`), discoverable by `openclaw skills list`. Skill not auto-inject (OpenClaw load skill on demand) — that why we also do step 2.
2. **SOUL.md nudge.** Tiny marker-fenced block appended to `~/.openclaw/workspace/SOUL.md`. OpenClaw inject SOUL.md into _every_ turn under "Project Context" (12K-per-file, 60K total — block well under). Lobster terse from message one. No `/caveman` per session. No nag.

```
~/.openclaw/workspace/
├── skills/caveman/SKILL.md   ← full ruleset, on-demand load
└── SOUL.md                    ← <!-- caveman-begin --> ... <!-- caveman-end -->
                                  ↑ auto-inject every turn
```

Custom workspace path? `OPENCLAW_WORKSPACE=/your/path` before the command. Uninstall: same one-liner with `--uninstall` — skill folder gone, SOUL.md block ripped out cleanly, your other workspace content stay untouched. Idempotent re-runs (frontmatter not double-prepended, marker block not duplicated).

Lobster claw still sharp. Lobster mouth now small. Brain still big.

## Caveman Ecosystem

[](https://github.com/juliusbrussee/caveman#caveman-ecosystem)

Five tools. One philosophy: **agent do more with less**.

|Repo|What|
|---|---|
|[**caveman**](https://github.com/JuliusBrussee/caveman) _(you here)_|Output compression — _why use many token when few do trick_|
|[**caveman-code**](https://github.com/JuliusBrussee/caveman-code)|Whole terminal coding agent — _why use many token when whole agent can save_|
|[**cavemem**](https://github.com/JuliusBrussee/cavemem)|Cross-agent memory — _why agent forget when agent can remember_|
|[**cavekit**](https://github.com/JuliusBrussee/cavekit)|Spec-driven build loop — _why agent guess when agent can know_|
|[**cavegemma**](https://github.com/JuliusBrussee/finetune-caveman)|Gemma 4 31B fine-tuned on caveman pairs — _why prompt every turn when weight remember_|

Compose: cavekit drive build, caveman compress what agent _say_, cavemem compress what agent _remember_, cavegemma bake compression into weight, caveman-code ship it all as one terminal agent. One rock. Two rock. Three rock. Four rock. Five rock. That it.

## More Skill From Same Cave

[](https://github.com/juliusbrussee/caveman#more-skill-from-same-cave)

Caveman has siblings. [**JuliusBrussee/skills**](https://github.com/JuliusBrussee/skills) — five skills, one install, works in Claude Code, Cursor, Gemini, Cline, Copilot, 40+ agents:

|Skill|What|
|---|---|
|[**caveman**](https://github.com/JuliusBrussee/skills/tree/main/skills/caveman)|This one. Speak less, say more.|
|[**grill-me**](https://github.com/JuliusBrussee/skills/tree/main/skills/grill-me)|Agent grill your plan _before_ you build wrong thing. Checks how much you know first — no condescend, no coddle.|
|[**interface-kit**](https://github.com/JuliusBrussee/skills/tree/main/skills/interface-kit)|Build UI that look good, load fast, work for everyone.|
|[**junior-to-senior**](https://github.com/JuliusBrussee/skills/tree/main/skills/junior-to-senior)|Adversarial review pass. Junior output go in, senior output come out.|
|[**loop-factory**](https://github.com/JuliusBrussee/skills/tree/main/skills/loop-factory)|Spec-driven task loop — inbox → active → archive, review gate between.|

```shell
npx skills@latest add JuliusBrussee/skills
```

One command. Five skill. Cave well stocked.

## Links

[](https://github.com/juliusbrussee/caveman#links)

- [INSTALL.md](https://github.com/JuliusBrussee/caveman/blob/main/INSTALL.md) — full install matrix, all flags, per-agent detail
- [CONTRIBUTING.md](https://github.com/JuliusBrussee/caveman/blob/main/CONTRIBUTING.md) — how to send patch
- [CLAUDE.md](https://github.com/JuliusBrussee/caveman/blob/main/CLAUDE.md) — maintainer guide (file ownership, hook architecture, CI)
- [docs/](https://github.com/JuliusBrussee/caveman/blob/main/docs) — extra guides (Windows install, etc.)
- [Issues](https://github.com/JuliusBrussee/caveman/issues) — bug, feature, weird behavior