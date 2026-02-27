# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin (`lll-animation`) for the Long Lunch Lab — Animations workshop. Participants build a rolling 3D isometric cube in two phases: CSS + GSAP (Phase 1), then Three.js + GSAP (Phase 2). The plugin provides skills (domain knowledge loaded into context) and agents (specialized reviewers) — no application code, no build system.

## Development Workflow

This is a pure-content plugin (markdown files only). To test changes locally:

```bash
claude /plugin uninstall lll-animation@lll-animation-dev
claude /plugin install lll-animation@lll-animation-dev
```

Then restart Claude Code. There are no build, lint, or test commands.

## Architecture

### Plugin Structure

```
.claude-plugin/plugin.json    — Plugin manifest (name, version, keywords)
skills/                        — 8 skills (7 active, 1 deprecated)
agents/                        — 2 specialized agents

lll-animation-mcp/             — Optional companion plugin (Notion integration)
  .claude-plugin/plugin.json
  .mcp.json                    — Notion MCP server
  commands/fetch-spec.md       — /fetch-spec command
```

Skills and agents are auto-discovered by Claude Code from their respective directories. Each skill is a `SKILL.md` with optional `references/` subdirectory. The Notion MCP server and `/fetch-spec` command live in a separate companion plugin so users can install them independently.

### Skill Delegation Graph

The skills form a directed graph — each skill handles its domain and delegates out-of-scope topics:

```
workshop-guide (top-level navigator for all 12 tasks)
  ├── phase1-css-cube      (Tasks 1.1–1.2: cube construction, isometric projection)
  ├── isometric-rolling-cube (Tasks 1.3–1.6, 2.3–2.5: rolling, boundaries, cursor)
  ├── threejs-fundamentals  (Tasks 2.1–2.2: scene, camera, Object3D)
  └── gsap-expert           (all phases: animation API reference)

phase1-css-cube ──delegates-to──> isometric-rolling-cube (rolling topics)
isometric-rolling-cube ──delegates-to──> phase1-css-cube (CSS construction topics)
```

Standalone reference skills (no delegation): `threejs-animation`, `threejs-materials`

Deprecated: `css-3d-cube` (replaced by `phase1-css-cube`, `user-invocable: false`)

### Agents

- `animation-reviewer` — Proactive code reviewer after Tasks 1.3/2.3. Checks 8 failure modes (pivot, reparenting, rotation axes, cumulative state, grid snapping, render loop, boundaries, chaining). Tools: Read, Grep, Glob.
- `phase-transition-helper` — Activated when starting Task 2.1. Maps Phase 1 CSS concepts to Three.js equivalents. Tools: Read, Grep, Glob.

## Authoring Conventions

### Skill SKILL.md

- **Frontmatter:** `name`, `description` (third-person: "This skill should be used when..."), `user-invocable`, `argument-hint`
- **Body:** Imperative/infinitive form ("Create the pivot" not "You should create the pivot")
- **Length:** SKILL.md body under 2,500 words; move detailed content to `references/`
- **Cross-references:** Use skill names in backticks (e.g., "delegate to the `gsap-expert` skill")
- **Direction naming:** Use "top-right, bottom-right, bottom-left, top-left" (Notion spec standard)

### Agent Markdown

- **Frontmatter:** `name`, `description` (when to trigger), `tools` (available tools)
- **Output format:** Structured checklists with PASS/FAIL/PARTIAL per check

## Key Domain Concepts

These concepts recur across multiple skills and are the most common source of participant bugs:

- **Reset-per-roll pattern:** After each 90-degree roll, zero the rotation and snap position to grid. Without this, cumulative transforms break after 2–3 rolls. This is the core technique for both phases.
- **`attach()` vs `add()` in Three.js:** `attach()` preserves world position when reparenting; `add()` does not. Using the wrong one causes the cube to visually jump.
- **`preserve-3d` not inherited:** Must be set explicitly on every ancestor whose children participate in 3D space. Silently broken by `overflow`, `opacity < 1`, `filter`, `clip-path`.
- **No CSS `perspective` for isometric:** Isometric projection is orthographic — the `perspective` property creates vanishing-point distortion.
- **GSAP render loop:** GSAP tweens Three.js values but doesn't trigger renders. Need `gsap.ticker.add(() => renderer.render(scene, camera))` or equivalent.

## Canonical Source

The workshop spec lives in Notion: `LLL Animation Spec` (linked in `skills/workshop-guide/references/resources.md`). When updating skills, verify against this spec. To fetch it programmatically, install the companion plugin `lll-animation-mcp`.
