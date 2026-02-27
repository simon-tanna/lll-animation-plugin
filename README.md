# LLL Animation Workshop Plugin

Claude Code plugin for the **Long Lunch Lab — Animations** workshop. Provides skills, subagents, and domain knowledge for building a rolling 3D isometric cube in two phases: CSS + GSAP, then Three.js + GSAP.

## Installation

```bash
claude /plugin marketplace add simon-tanna/lll-animation-plugin
claude /plugin install lll-animation@lll-animation-dev
```

Then restart Claude Code.

## What's Included

### Skills (8)

All skills are dual-mode: Claude auto-invokes them when the conversation topic matches, and participants can also invoke them directly as slash commands. Several skills include detailed reference files (GSAP API docs, Desandro technique walkthrough, external resource links) that load on demand — keeping context lean while providing deep documentation when needed.

| Skill | Slash Command | Description |
|-------|---------------|-------------|
| `workshop-guide` | `/workshop-guide [task-id]` | Task navigator with acceptance criteria, gotcha warnings, and phase transition help. e.g. `/workshop-guide 1.3` |
| `isometric-rolling-cube` | `/isometric-rolling-cube [topic]` | Rolling technique for both CSS and Three.js — pivot positioning, direction mappings, reparenting cycle, grid snapping, boundary enforcement, cursor-following logic. |
| `phase1-css-cube` | `/phase1-css-cube [topic]` | Desandro 6-div CSS cube construction, face positioning with rotate + `translateZ`, `preserve-3d`, `color-mix()` shading, isometric projection. |
| `gsap-expert` | `/gsap-expert [topic]` | Comprehensive GSAP v3 reference with 636KB of API documentation in `references/`. |
| `threejs-fundamentals` | `/threejs-fundamentals [topic]` | Scene setup, cameras, renderer, Object3D hierarchy, coordinate systems. |
| `threejs-animation` | `/threejs-animation [topic]` | Keyframe animation, skeletal animation, morph targets, animation mixing. |
| `threejs-materials` | `/threejs-materials [topic]` | PBR materials, Phong, shader materials, material properties and optimisation. |
| `react-integration` | `/react-integration [topic]` | GSAP and Three.js integration patterns for React — `useRef`, `useEffect`, cleanup, and JSX differences from HTML. |

### Agents (2)

| Agent | Trigger | Description |
|-------|---------|-------------|
| `animation-reviewer` | After Task 1.3 or 2.3 | Reviews rolling animation code against 8 failure modes: pivot points, reparenting, rotation axes, cumulative state, grid snapping, render loops, boundaries, chaining. |
| `phase-transition-helper` | Starting Task 2.1 | Maps Phase 1 CSS implementation to Phase 2 Three.js equivalents. Identifies reusable code and flags conceptual shifts. |

## Workshop Structure

```
Phase 1 — CSS + GSAP (~1.5 hours)
  1.1  Construct the 3D Cube (HTML + CSS)
  1.2  Apply Isometric Projection
  1.3  Implement Rolling Animation with GSAP
  1.4  Random Direction Selection
  1.5  Window Boundary Enforcement
  1.6  OPTIONAL: Cursor Following

Phase 2 — Three.js + GSAP (~1.5 hours)
  2.1  Create the Cube (Geometry + Material)
  2.2  Understand the Isometric Camera (read-only)
  2.3  Implement Rolling Animation with GSAP
  2.4  Window Boundary Enforcement
  2.5  OPTIONAL: Cursor Following
```

## Usage

Once installed, participants can:

- **Slash commands** — invoke any skill directly: `/workshop-guide 1.3`, `/gsap-expert timeline`, `/phase1-css-cube preserve-3d`
- **Auto-invocation** — just ask a question naturally ("my cube is spinning in place") and Claude pulls in the relevant skill automatically based on topic
- **Agents** — after completing the rolling animation (Task 1.3 or 2.3), the `animation-reviewer` agent can be invoked to check for common bugs. When starting Phase 2, the `phase-transition-helper` agent maps Phase 1 code to Three.js equivalents

## Optional: Notion Integration

The companion plugin `lll-animation-mcp` adds a Notion MCP server and `/fetch-spec` command for pulling the canonical workshop spec directly into your session.

```bash
claude /plugin install lll-animation-mcp@lll-animation-mcp-dev
```

This adds:
- **`/fetch-spec [task-or-section]`** — Fetches the workshop spec from Notion. Use `/fetch-spec 1.3` for a specific task or `/fetch-spec` for the full overview.
- **Notion MCP server** — Connects to [Notion MCP](https://mcp.notion.com/mcp) for reading the workshop spec. OAuth is handled on first use.

## Development

To test locally after making changes:

```bash
claude /plugin uninstall lll-animation@lll-animation-dev
claude /plugin install lll-animation@lll-animation-dev
```

Then restart Claude Code.
