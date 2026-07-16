# claude-webapp-toolkit

A shared library of Claude Code agents and skills for web application development. Designed to be added as a git submodule in consuming repositories, making these agents and skills available to Claude across machines and projects.

## Contents

### Agents (`agents/`)

Specialist agents that Claude invokes to handle specific categories of work:

| Agent                     | Model  | Description                                                                                 |
| ------------------------- | ------ | ------------------------------------------------------------------------------------------- |
| `tech-lead-architect`     | Opus   | Architecture decisions, system design, technical planning, performance strategy             |
| `typescript-code-writer`  | Sonnet | Writing, refactoring, and testing TypeScript with strict type safety and clean architecture |
| `ux-design-accessibility` | Sonnet | UI components, page layouts, CSS, semantic HTML, and WCAG 2.2 accessibility                 |
| `sveltekit-developer`     | Sonnet | SvelteKit routes, Svelte 5 components, form actions, load functions, and API routes         |

### Skills (`skills/`)

Reference documents that agents load at the start of relevant tasks to apply consistent standards:

| Skill                 | Description                                                                                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `typescript.md`       | Type safety rules, naming conventions, error handling, design patterns, async guidelines, and a self-review checklist                                               |
| `web-optimization.md` | Core Web Vitals targets (LCP/INP/CLS), bundle strategy, image pipeline, caching, critical path, and performance budgets                                             |
| `ui-design.md`        | Visual hierarchy, spacing (8-point grid), typography, colour tokens, component states, WCAG 2.2 AA accessibility, motion, responsive design, and a polish checklist |
| `sveltekit.md`        | Svelte 5 runes, SvelteKit routing, data loading, form actions, API routes, TypeScript integration, and Vitest testing patterns                                      |

## Usage

### Add as a submodule

```bash
git submodule add https://github.com/shanelucy/claude-webapp-toolkit .claude/toolkit
```

Then create symlinks so Claude Code discovers the agents, skills, and delegation instructions under `.claude/`:

**macOS / Linux**

```bash
ln -s toolkit/agents .claude/agents
ln -s toolkit/skills .claude/skills
```

If `.claude/CLAUDE.md` does not already exist, symlink it too:

```bash
ln -s toolkit/CLAUDE.md .claude/CLAUDE.md
```

If `.claude/CLAUDE.md` already exists, append the delegation instructions from `toolkit/CLAUDE.md` to it instead of symlinking.

**Windows (PowerShell — requires Developer Mode or admin)**

```powershell
# Remove existing directories if present, then create symlinks
if (Test-Path .claude\agents) { Remove-Item -Recurse -Force .claude\agents }
if (Test-Path .claude\skills) { Remove-Item -Recurse -Force .claude\skills }
New-Item -ItemType SymbolicLink -Path .claude\agents -Target .claude\toolkit\agents
New-Item -ItemType SymbolicLink -Path .claude\skills -Target .claude\toolkit\skills

# Symlink CLAUDE.md only if one doesn't already exist
if (-not (Test-Path .claude\CLAUDE.md)) {
    New-Item -ItemType SymbolicLink -Path .claude\CLAUDE.md -Target .claude\toolkit\CLAUDE.md
}
# If .claude\CLAUDE.md already exists, manually append the delegation table from toolkit\CLAUDE.md
```

Commit the symlinks alongside the submodule so all contributors get the same layout automatically:

```bash
git add .claude/agents .claude/skills .gitmodules .claude/toolkit
git commit -m "Add claude-webapp-toolkit submodule with agent and skill symlinks"
```

### How agents and skills are loaded

Claude Code discovers agents in `.claude/agents/` and skills in `.claude/skills/`. The toolkit's `CLAUDE.md` (read as `.claude/CLAUDE.md`) tells Claude to delegate tasks to the appropriate specialist agent rather than handling them inline.

**This delegation instruction is the critical piece.** Without it, Claude may handle TypeScript, SvelteKit, or UI tasks itself instead of invoking the specialist agents — especially in projects that already have their own agents or CLAUDE.md. Claude Code reads CLAUDE.md files from the project root, `.claude/`, and subdirectories additively, so the toolkit's `CLAUDE.md` stacks with any existing project instructions without conflict.

Once delegated, each agent reads its relevant skill files as a mandatory first step before producing any output.

### Updating the submodule

```bash
git submodule update --remote .claude
```

## Agent memory

Each agent maintains persistent memory in a per-project directory (`.claude/agent-memory/<agent-name>/`). This memory is project-local and not stored in this repository — it accumulates as agents work in each consuming project, building up context about that project's conventions, decisions, and patterns.
