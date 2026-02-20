# LLL Animation Workshop Plugin

Claude Code plugin for the **Long Lunch Lab — Animations** workshop. Provides skills, subagents, and domain knowledge for building a rolling 3D isometric cube in two phases: CSS + GSAP, then Three.js + GSAP.

## Installation

```bash
claude /plugin marketplace add simon-tanna/lll-animation-plugin
claude /plugin install lll-animation@lll-animation-dev
```

Then restart Claude Code.

## What's Included

### Skills (7)

| Skill | Type | Description |
|-------|------|-------------|
| `workshop-guide` | User-invocable (`/guide`) | Task navigator with acceptance criteria, gotcha warnings, and phase transition help. Accepts optional task ID (e.g. `/guide 1.3`). |
| `isometric-rolling-cube` | Claude-only | Pivot-point rolling technique for both CSS and Three.js — direction mappings, reparenting cycle, quaternion tracking, grid snapping. |
| `css-3d-cube` | Claude-only | 2-div + pseudo-element cube construction, `preserve-3d` rules, `color-mix()` shading, isometric projection angles. |
| `gsap-expert` | Claude-only | Comprehensive GSAP v3 reference with 636KB of API documentation in `references/`. |
| `threejs-fundamentals` | Claude-only | Scene setup, cameras, renderer, Object3D hierarchy, coordinate systems. |
| `threejs-animation` | Claude-only | Keyframe animation, skeletal animation, morph targets, animation mixing. |
| `threejs-materials` | Claude-only | PBR materials, Phong, shader materials, material properties and optimisation. |

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

- Type `/guide` to see where they are and what to do next
- Type `/guide 1.3` for task-specific help, acceptance criteria, and relevant gotchas
- Ask any question about rolling cubes, CSS 3D, GSAP, or Three.js — the relevant skills trigger automatically
- After completing the rolling animation (Task 1.3 or 2.3), the `animation-reviewer` agent can be invoked to check for common bugs
- When starting Phase 2, the `phase-transition-helper` agent maps their Phase 1 code to Three.js equivalents

## Development

To test locally after making changes:

```bash
claude /plugin uninstall lll-animation@lll-animation-dev
claude /plugin install lll-animation@lll-animation-dev
```

Then restart Claude Code.
