# claude-webapp-toolkit

A shared library of Claude Code agents and skills for web application development. Designed to be added as a git submodule in consuming repositories, making these agents and skills available to Claude across machines and projects.

## Contents

### Agents (`agents/`)

Specialist agents that Claude invokes to handle specific categories of work:

| Agent | Model | Description |
|---|---|---|
| `tech-lead-architect` | Opus | Architecture decisions, system design, technical planning, performance strategy |
| `typescript-code-writer` | Sonnet | Writing, refactoring, and testing TypeScript with strict type safety and clean architecture |
| `ux-design-accessibility` | Sonnet | UI components, page layouts, CSS, semantic HTML, and WCAG 2.2 accessibility |
| `sveltekit-developer` | Sonnet | SvelteKit routes, Svelte 5 components, form actions, load functions, and API routes |

### Skills (`skills/`)

Reference documents that agents load at the start of relevant tasks to apply consistent standards:

| Skill | Description |
|---|---|
| `typescript.md` | Type safety rules, naming conventions, error handling, design patterns, async guidelines, and a self-review checklist |
| `web-optimization.md` | Core Web Vitals targets (LCP/INP/CLS), bundle strategy, image pipeline, caching, critical path, and performance budgets |
| `ui-design.md` | Visual hierarchy, spacing (8-point grid), typography, colour tokens, component states, WCAG 2.2 AA accessibility, motion, responsive design, and a polish checklist |
| `sveltekit.md` | Svelte 5 runes, SvelteKit routing, data loading, form actions, API routes, TypeScript integration, and Vitest testing patterns |

## Usage

### Add as a submodule

```bash
git submodule add https://github.com/shanelucy/claude-webapp-toolkit .claude
```

Or if your repository already has a `.claude` directory, add the submodule into a subdirectory and symlink its contents:

```bash
git submodule add https://github.com/shanelucy/claude-webapp-toolkit .claude/toolkit
```

Then create symlinks so Claude Code discovers the agents and skills under `.claude/`:

**macOS / Linux**
```bash
ln -s toolkit/agents .claude/agents
ln -s toolkit/skills .claude/skills
```

**Windows (PowerShell — requires Developer Mode or admin)**
```powershell
# Remove existing directories if present, then create symlinks
if (Test-Path .claude\agents) { Remove-Item -Recurse -Force .claude\agents }
if (Test-Path .claude\skills) { Remove-Item -Recurse -Force .claude\skills }
New-Item -ItemType SymbolicLink -Path .claude\agents -Target .claude\toolkit\agents
New-Item -ItemType SymbolicLink -Path .claude\skills -Target .claude\toolkit\skills
```

Commit the symlinks alongside the submodule so all contributors get the same layout automatically:

```bash
git add .claude/agents .claude/skills .gitmodules .claude/toolkit
git commit -m "Add claude-webapp-toolkit submodule with agent and skill symlinks"
```

### How agents and skills are loaded

Claude Code automatically discovers agents in `.claude/agents/` and skills in `.claude/skills/`. Each agent's frontmatter declares which skills it reads at task start — no manual wiring required.

The agents are self-contained: each one describes when it should be used (its `description` field), what model to run on, and what skills to load. Claude selects the right agent automatically based on the task context.

### Updating the submodule

```bash
git submodule update --remote .claude
```

## Agent memory

Each agent maintains persistent memory in a per-project directory (`.claude/agent-memory/<agent-name>/`). This memory is project-local and not stored in this repository — it accumulates as agents work in each consuming project, building up context about that project's conventions, decisions, and patterns.
