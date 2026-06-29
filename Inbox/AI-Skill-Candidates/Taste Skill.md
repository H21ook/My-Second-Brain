# Taste Skill

[](https://github.com/Leonxlnx/taste-skill#taste-skill)

_The Anti-Slop Frontend Framework for AI Agents_

[![Visit tasteskill.dev](https://github.com/Leonxlnx/taste-skill/raw/main/assets/readme-buttons/btn-site.webp)](https://tasteskill.dev/ "Visit tasteskill.dev")

### Sponsors

[](https://github.com/Leonxlnx/taste-skill#sponsors)

[![animations.dev](https://github.com/Leonxlnx/taste-skill/raw/main/assets/sponsors/animations-dev.webp)](https://animations.dev/)[**Emil Kowalski**](https://github.com/emilkowalski) · [animations.dev](https://animations.dev/)

[![Vercel OSS Program](https://github.com/Leonxlnx/taste-skill/raw/main/assets/vercel-oss-program-badge.svg)](https://vercel.com/open-source-program)

[Become a sponsor](https://github.com/sponsors/Leonxlnx)

Portable **Agent Skills** that upgrade AI-built interfaces: stronger layout, typography, motion, and spacing instead of boilerplate-looking UIs. This repo also includes **image-generation skills** for reference boards (web, mobile, brand kits). Pair them with **ChatGPT Images** or similar generators, then hand the frames to Codex, Cursor, or Claude Code for implementation.

[![MIT License](https://github.com/Leonxlnx/taste-skill/raw/main/assets/readme-buttons/btn-mit.webp)](https://github.com/Leonxlnx/taste-skill/blob/main/LICENSE)   [![Agent Skills compatible](https://github.com/Leonxlnx/taste-skill/raw/main/assets/readme-buttons/btn-agent-skills.webp)](https://github.com/vercel-labs/agent-skills)   [![Codex, Cursor, Claude](https://github.com/Leonxlnx/taste-skill/raw/main/assets/readme-buttons/btn-tools.webp)](https://github.com/Leonxlnx/taste-skill#installing)   [![Changelog](https://github.com/Leonxlnx/taste-skill/raw/main/assets/readme-buttons/btn-changelog.webp)](https://www.tasteskill.dev/changelog)

## Disclaimer

[](https://github.com/Leonxlnx/taste-skill#disclaimer)

Taste Skill has no official token, coin, or crypto project. Any token using my name, image, or project is unaffiliated and not endorsed by me.

[Disclaimer](https://github.com/Leonxlnx/taste-skill#disclaimer) · [Install](https://github.com/Leonxlnx/taste-skill#installing) · [Skills](https://github.com/Leonxlnx/taste-skill#skills) · [Settings](https://github.com/Leonxlnx/taste-skill#settings-taste-skill-only) · [Examples](https://github.com/Leonxlnx/taste-skill#examples) · [Sponsors](https://github.com/Leonxlnx/taste-skill#sponsors) · [Research](https://github.com/Leonxlnx/taste-skill#research) · [FAQ](https://github.com/Leonxlnx/taste-skill#common-questions) · [License](https://github.com/Leonxlnx/taste-skill#license)

## Feedback & Contributions

[](https://github.com/Leonxlnx/taste-skill#feedback--contributions)

We would love your feedback. Suggestions and bug reports:

- Open a Pull Request or Issue on GitHub
- DM [@lexnlin](https://x.com/lexnlin) or [@blueemi99](https://x.com/blueemi99)
- Email us at [hello@tasteskill.dev](mailto:hello@tasteskill.dev)

## Installing

[](https://github.com/Leonxlnx/taste-skill#installing)

The [`npx skills add`](https://github.com/vercel-labs/agent-skills) CLI scans the `skills/` folder in this repo, so **all skills below (code and image-generation) install the same way.**

```shell
npx skills add https://github.com/Leonxlnx/taste-skill
```

Install a single skill by its **install name** (the `name:` field inside the SKILL frontmatter, not the folder name):

```shell
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"
```

You can also copy any `SKILL.md` into your project or paste it into ChatGPT / Codex conversations.

### Updating from the previous version

[](https://github.com/Leonxlnx/taste-skill#updating-from-the-previous-version)

The default `taste-skill` (install name `design-taste-frontend`) is now **v2 (experimental)**, a substantial rewrite of the original v1. If you already have v1 installed, just re-run the install command and you will be upgraded:

```shell
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"
```

The install name did not change, so no script updates are needed. The newer SKILL.md replaces the older one in place.

If you depend on the exact behavior of v1 and want to pin to it explicitly:

```shell
npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend-v1"
```

See [CHANGELOG.md](https://github.com/Leonxlnx/taste-skill/blob/main/CHANGELOG.md) for the full v1 to v2 diff and the rationale.

## Skills

[](https://github.com/Leonxlnx/taste-skill#skills)

Each skill does one job; you do not need all of them at once. **Implementation skills** output code. **Image-generation skills** output reference images only.

The `Install name` column is the exact value you pass to `--skill`.

|Skill (folder)|Install name|Description|
|---|---|---|
|**taste-skill**|`design-taste-frontend`|🆕 **v2 (experimental)** - substantial rewrite of the default skill. Reads the brief, infers the design language, tunes three dials (VARIANCE / MOTION / DENSITY). Brief inference, design-system map, hard em-dash ban, canonical GSAP code skeletons, redesign-audit protocol, strict pre-flight check. Actively iterating toward v2.0.0 stable.|
|**taste-skill-v1**|`design-taste-frontend-v1`|The original v1 of taste-skill, preserved for projects depending on its exact behavior. Use only if the v2 default breaks something specific in your workflow.|
|**gpt-tasteskill**|`gpt-taste`|Stricter variant for GPT/Codex: higher layout variance, stronger GSAP direction, aggressive anti-slop.|
|**image-to-code-skill**|`image-to-code`|Image-first pipeline: generate site references, analyze them, then implement the frontend to match.|
|**redesign-skill**|`redesign-existing-projects`|Existing projects: audit the UI first, then fix layout, spacing, hierarchy, styling.|
|**soft-skill**|`high-end-visual-design`|Polished, calm, expensive UI with softer contrast, whitespace, premium fonts, spring motion.|
|**output-skill**|`full-output-enforcement`|When the model ships half-finished work: full output, no placeholder comments.|
|**minimalist-skill**|`minimalist-ui`|Editorial product UI (Notion/Linear vibes), restrained palette, crisp structure.|
|**brutalist-skill**|`industrial-brutalist-ui`|Hard mechanical language: Swiss type, sharp contrast, experimental layout.|
|**stitch-skill**|`stitch-design-taste`|Google Stitch-compatible rules, including optional `DESIGN.md` export format.|

### Image generation skills

[](https://github.com/Leonxlnx/taste-skill#image-generation-skills)

These produce design images only (no code). Use with ChatGPT Images, Codex image mode, or any agent that generates images.

|Skill (folder)|Install name|Description|
|---|---|---|
|**imagegen-frontend-web**|`imagegen-frontend-web`|Website comps: hero, landing, multi-section with strong typography, spacing, anti-slop art direction.|
|**imagegen-frontend-mobile**|`imagegen-frontend-mobile`|Mobile screens and flows: iOS/Android/cross-platform, mockups, readable type, coherent sets.|
|**brandkit**|`brandkit`|Brand-kit boards: logo directions, palettes, type, identity applications across categories.|

### Which one should I use?

[](https://github.com/Leonxlnx/taste-skill#which-one-should-i-use)

- Start with **taste-skill** for the safest general default. (Now v2 experimental - see what changed in the [CHANGELOG](https://github.com/Leonxlnx/taste-skill/blob/main/CHANGELOG.md).)
- If you depend on the exact behavior of the original taste-skill, install **taste-skill-v1** instead.
- Use **gpt-taste** when you want the stricter GPT/Codex-oriented rules and motion/layout enforcement.
- Use **image-to-code-skill** for image → analyze → code website workflows.
- Use **redesign-skill** to improve an existing codebase instead of greenfield styling.
- Add **soft-skill**, **minimalist-skill**, or **brutalist-skill** when the visual direction is already chosen.
- Add **output-skill** if the agent keeps truncating output.
- Use **imagegen-frontend-web**, **imagegen-frontend-mobile**, or **brandkit** when the deliverable is **images** (comps, flows, identity boards), then pass results to your coding agent.

### Image-first tip

[](https://github.com/Leonxlnx/taste-skill#image-first-tip)

For **image-to-code-skill**, state the pipeline in the prompt, e.g.: `follow the skill: generate images, then analyze, then code`.

### ChatGPT Images and Codex

[](https://github.com/Leonxlnx/taste-skill#chatgpt-images-and-codex)

Attach or paste **`imagegen-frontend-web`**, **`imagegen-frontend-mobile`**, or **`brandkit`** and ask for the frames you need, then feed the renders to Codex, Cursor, or Claude Code. Use **image-to-code-skill** when you want one workflow that both generates references and implements the site in code.

## Settings (taste-skill only)

[](https://github.com/Leonxlnx/taste-skill#settings-taste-skill-only)

Numbers at the top of the file are 1-10 dials:

- **DESIGN_VARIANCE**: Layout experimentation (lower: centered/clean · higher: asymmetric/modern).
- **MOTION_INTENSITY**: Animation depth (lower: hover · higher: scroll/magnetic).
- **VISUAL_DENSITY**: Information per viewport (lower: spacious · higher: dense dashboards).