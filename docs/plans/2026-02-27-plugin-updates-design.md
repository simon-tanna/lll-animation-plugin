# Plugin Updates Design — 2026-02-27

## Context

The LLL Animation Workshop plugin (`lll-animation`) was reviewed against three Notion documents:

- **Original spec:** LLL Animation Spec (primary workshop task definitions)
- **Crash Course guide:** Crash Course: 3D cube animations (CSS vs Three.js) — new facilitator material
- **Performance guide:** 3D Cube [CSS vs Three.js] — performance comparison

The review identified 22 issues across 9 files: factual errors, architectural gaps, missing content, and housekeeping. This document records the approved design for resolving all 22.

---

## Approach

**Approach B — React skill first, then file-by-file.**

A new `react-integration` skill is created first to centralise React/JSX patterns. All other changes are then applied file-by-file. Correctness fixes (wrong values, wrong names) are addressed before additions are built on top of them.

---

## Section 1 — New `react-integration` Skill

### Rationale

The workshop uses Vite + React TypeScript (`Cube.tsx`, `Cube3D.tsx`). No existing skill addresses how GSAP and Three.js integrate with React — `useRef` for DOM targeting, `useEffect` for scene initialisation, `className` vs `class`, cleanup on unmount. This gap affects all participants from Task 1.1 onward.

A dedicated skill follows the existing plugin pattern of centralising cross-cutting concerns (cf. `gsap-expert` for all GSAP API questions).

### Design

**Location:** `skills/react-integration/SKILL.md`

**Frontmatter:**
```yaml
name: react-integration
description: >
  Use when a participant asks how to use GSAP or Three.js inside a React
  component, how to target DOM elements with GSAP in React, how to set up
  a Three.js scene in a useEffect, or why their animation works in vanilla
  JS but not in their .tsx file.
user-invocable: true
argument-hint: "e.g. 'GSAP with useRef', 'Three.js in useEffect'"
```

**Content sections:**
1. **Why React changes things** — GSAP needs a stable DOM ref, not `querySelector`; Three.js scene must live in `useEffect` to run after mount; cleanup prevents memory leaks on re-renders
2. **GSAP + React (Phase 1)** — `useRef` on the position container div; pass `.current` to `gsap.to()`; `useEffect` with a cleanup `tween.kill()`
3. **Three.js + React (Phase 2)** — `useRef` for the `<canvas>` element; `useEffect` for renderer/scene/camera init; return a cleanup function calling `renderer.dispose()`
4. **JSX differences** — `className` not `class`; inline styles as objects (`style={{ transform: '...' }}`); event handlers as camelCase props

**Delegation in:** Referenced from `phase1-css-cube`, `phase-transition-helper`, and `workshop-guide` with delegation lines pointing participants here for React-specific questions.

---

## Section 2 — Correctness Fixes

These are confirmed wrong values or names, corrected before any additions are built on top of them.

### 2a — Isometric rotation formula

**Wrong:** `rotateX(35.264deg) rotateZ(45deg)`
**Correct:** `rotateX(-35.264deg) rotateY(45deg)` (confirmed by Crash Course facilitator guide)

Every occurrence replaced in:
- `skills/phase1-css-cube/SKILL.md`
- `skills/phase1-css-cube/references/desandro-cube-technique.md`
- `skills/workshop-guide/SKILL.md` (Phase Transition table)
- `skills/css-3d-cube/SKILL.md`

### 2b — Tech stack label

**Wrong:** "Vite + TypeScript" / "vanilla JS/TS"
**Correct:** "Vite + React TypeScript"

Updated in:
- `skills/workshop-guide/SKILL.md` (line 18 + task hints)
- `skills/phase1-css-cube/SKILL.md` (HTML Structure intro)

### 2c — CSS direction naming in `isometric-rolling-cube`

**Wrong:** "Right / Left / Forward / Backward"
**Correct:** "top-right / bottom-right / bottom-left / top-left" (canonical per CLAUDE.md and original spec)

Updated in `skills/isometric-rolling-cube/SKILL.md` CSS direction table only (~6 lines). The Three.js table in the same file already uses correct naming.

### 2d — Render loop preferred option

**Wrong:** `phase-transition-helper.md` marks `gsap.ticker` as "recommended"
**Correct:** Continuous RAF loop is the spec's preferred option (labeled "1-preferred" in original spec gotchas)

Updated in:
- `agents/phase-transition-helper.md` (Render Loop section — correct ranking)
- `agents/phase-transition-helper.md` (Build Order step 3 — changed from "Set up the render loop" to "Verify the render loop — the boilerplate provides a RAF loop already; confirm it is running")

---

## Section 3 — Architecture Additions

New content confirmed by the Crash Course guide and original spec.

### 3a — 3-Container CSS Architecture

**Gap:** The Crash Course introduces a critical design pattern: each CSS container owns exactly one transform responsibility. Without this separation, GSAP's position animation overwrites the isometric rotation on the same element, breaking both.

**Design:**

A dedicated section "The 3-Container Architecture" is added to `skills/phase1-css-cube/SKILL.md` before the HTML Structure block:

```
Position container   → GSAP animates x/y translation only (never rotated)
Perspective container → static isometric rotation, never animated
Modeled cube         → GSAP animates rolling rotation only (never translated)

Why: GSAP overwrites the entire `transform` property when it animates.
Mixing position and rotation on one element means GSAP's position tween
destroys the isometric rotation.
```

HTML structure example updated from 2 containers to 3, with named comments.

Same explanation (shorter summary) added to:
- `skills/workshop-guide/SKILL.md` Task 1.1 key concepts
- `skills/phase1-css-cube/references/phase1-tasks.md` Task 1.1 hints
- `skills/phase1-css-cube/references/desandro-cube-technique.md` — updated to show 3-container structure throughout (workshop-specific adaptation, not faithful DeSandro replication)

### 3b — `THREE.Group` as Isometric Container

**Gap:** The Crash Course establishes that the CSS 3-container architecture maps directly to Three.js: the CSS perspective container (static isometric rotation) maps to a `THREE.Group` with equivalent static rotation applied once.

**Design:**

New row in `agents/phase-transition-helper.md` concept mapping table:

| CSS | Three.js |
|-----|----------|
| Position Container + Perspective Container (two nested elements) | `THREE.Group` — consolidates both: holds static isometric tilt (`rotateX(-35.264deg) rotateY(45deg)`) and manages X/Y position. Three.js does not require two separate parent objects. |

Note: Three.js collapses the two CSS containers into a single Group. This is a simplification — participants who built 3 CSS containers should understand they need only 1 Group (not 2) in Three.js.

`skills/workshop-guide/SKILL.md` Phase Transition table "Isometric view" row expanded to note that the Group consolidates both CSS position and perspective containers — not a one-to-one replacement of the perspective container alone.

`skills/isometric-rolling-cube/SKILL.md` Phase 2 section gets a brief note on the Group as the unified isometric + position container.

### 3c — `yoyo` Jump Effect

**Gap:** The Crash Course describes `yoyo: true` as the mechanism for the characteristic cube landing jump. Entirely absent from all skills.

**Design:**

Added to `skills/isometric-rolling-cube/SKILL.md` GSAP Animation Patterns section:

```js
// The landing jump: yoyo plays forward then reverses automatically
gsap.to(positionContainer, {
  y: -jumpHeight,
  duration: rollDuration / 2,
  yoyo: true,
  repeat: 1,
  ease: "power2.out"
})
```

With explanation: "`yoyo: true` with `repeat: 1` plays the tween forward then automatically reverses — producing the up-then-down bounce at each landing."

Also added to `skills/workshop-guide/SKILL.md` Task 1.3 key concepts as a bullet.

### 3d — Starter Repository Description

**Gap:** `workshop-guide/SKILL.md` references "the starter repo" and "the boilerplate" repeatedly with no description of what they contain.

**Design:**

New opening section in `skills/workshop-guide/SKILL.md`:

```
## Starter Repository
Pre-configured: Vite + React TypeScript, isometric diamond grid background,
OrthographicCamera (Phase 2 boilerplate), RAF render loop (Phase 2 boilerplate),
all dependencies installed (GSAP, Three.js).
Do not modify: the camera, the grid, or the boilerplate render loop.
Component files: Cube.tsx (Phase 1), Cube3D.tsx (Phase 2).
```

---

## Section 4 — Agent Updates

### 4a — `animation-reviewer.md`

**Trigger timing:** Frontmatter description updated to trigger after **Task 1.3 alone** (rolling working in one direction), with an optional second pass after Task 1.5/2.3. Rationale: catching pivot and reset bugs before the participant builds direction selection (Task 1.4) on a broken foundation.

**New Check — CSS `transform-origin` (before current Check 1):**
- Is `transform-origin` set as a 3D value (three components), not a 2D value?
- Does it target the specific bottom edge for the current roll direction?
- Does the value change per direction?

**New Check — CSS `preserve-3d` override:**
- Are there any ancestor elements with `overflow`, `opacity < 1`, `filter`, or `clip-path` that would silently disable `transform-style: preserve-3d` on the rolling container?

**Summary line** updated from "Checks passed: N/8" to "N/10".

### 4b — `phase-transition-helper.md`

**Concept mapping additions:**
- CSS perspective container → `THREE.Group` (see Section 3b)
- CSS `backface-visibility: hidden` → `material.side = THREE.FrontSide` (Three.js default; `DoubleSide` renders both faces)

**`updateMatrixWorld()` note** added to pivot cycle section: "After positioning the pivot in the same frame as `attach()`, call `pivot.updateMatrixWorld()` first — stale world matrices cause the same visual jump as using `add()` instead of `attach()`."

**Closing handoff:** New final line after build order — "Once a single roll works in Three.js, run the `animation-reviewer` to catch common failure modes before building direction selection."

**Delegation to `react-integration`:** Build order step 1 notes — "For Three.js scene setup inside a React component, see the `react-integration` skill."

---

## Section 5 — Remaining Skill Updates

### 5a — `skills/isometric-rolling-cube/SKILL.md`

- `updateMatrixWorld()` note in Phase 2 pivot cycle Step 2 (see Section 4b)
- `yoyo` jump pattern in GSAP Animation Patterns (see Section 3c)
- CSS direction table renamed (see Section 2c)
- New `references/` subdirectory created with two files:
  - `references/direction-data.md` — full CSS and Three.js direction tables (rotation axes, translation vectors, edge names)
  - `references/boundary-algorithm.md` — boundary enforcement and cursor-following algorithm detail
  - SKILL.md body links to both; tables removed from body to bring it under the 2,500-word limit

### 5b — `skills/workshop-guide/SKILL.md`

- New "Starter Repository" section (see Section 3d)
- Task 1.1 key concepts: 3-container architecture summary (see Section 3a)
- Task 1.3 key concepts: `yoyo` jump bullet added (see Section 3c)
- Phase Transition table: `THREE.Group` note on isometric view row; corrected rotation formula
- Task 2.3 key concepts: clarifying note — "When using the pivot technique, GSAP targets `pivot.rotation`. The spec's `gsap.to(mesh.rotation, ...)` example illustrates sub-object targeting syntax, not the specific pivot target."
- Gotcha table: new entry — "`transform-origin` misconfiguration (Phase 1)": distinct from rotation axis confusion; using a 2D or fixed value breaks rolling for 3 of the 4 directions
- Tech stack label corrected (see Section 2b)

### 5c — `skills/phase1-css-cube/SKILL.md`

- 3-container architecture section added (see Section 3a)
- HTML structure example updated to 3 containers
- Isometric rotation formula corrected (see Section 2a)
- Description scope narrowed to Tasks 1.1–1.2 only
- Delegation line added: "For GSAP targeting inside React components, see the `react-integration` skill."

---

## Section 6 — Housekeeping

### 6a — `skills/css-3d-cube/SKILL.md` (deprecated)

Body collapsed to redirect:
```
This skill is deprecated. Use `phase1-css-cube` for CSS cube
construction and isometric projection, and `isometric-rolling-cube`
for rolling animation.
```
All technique content removed. `user-invocable: false` retained.

### 6b — `skills/threejs-animation/SKILL.md`

Opening callout added to body:
```
> Workshop note: This workshop drives all animation with GSAP — see the
> `gsap-expert` skill. This skill covers Three.js-native animation
> (keyframes, skeletal animation, morph targets) which are not used here.
```

### 6c — `skills/threejs-fundamentals/SKILL.md`

Quick Start material swapped from `MeshStandardMaterial` + ambient/directional lights to `MeshBasicMaterial` (no lights). Comment added: `// MeshBasicMaterial needs no lighting — matches workshop Task 2.1`.

### 6d — `skills/workshop-guide/references/resources.md`

Two links added:
- Crash Course: 3D cube animations (CSS vs Three.js) — `https://www.notion.so/labrys/Crash-Course-3D-cube-animations-CSS-vs-Three-js-313ced2a7e13801eb0f8e5ed0057e568`
- 3D Cube [CSS vs Three.js] (performance comparison) — `https://www.notion.so/labrys/3D-Cube-CSS-vs-Three-js-30bced2a7e138077b50ce60c97faddd2`

---

## Change Summary

| Step | Action | Files |
|------|--------|-------|
| 1 | Create `react-integration` skill | 1 new file |
| 2 | Correctness fixes (rotation formula, stack name, direction naming, render loop ranking) | 5 files |
| 3 | Architecture additions (3-container, `THREE.Group`, `yoyo`, starter repo) | 6 files |
| 4 | Agent updates (reviewer checks + timing, transition helper mapping + handoff) | 2 files |
| 5 | Remaining skill updates + new `references/` files for `isometric-rolling-cube` | 3 files + 2 new ref files |
| 6 | Housekeeping (deprecated skill, disclaimers, Quick Start material, resource links) | 4 files |

**Total: 22 issues resolved across 13 existing files, 1 new skill file, 2 new reference files.**

---

## Sources

- LLL Animation Spec: `https://www.notion.so/LLL-Animation-Spec-30dced2a7e13804eb784d06ef1b9e6a9`
- Crash Course guide: `https://www.notion.so/labrys/Crash-Course-3D-cube-animations-CSS-vs-Three-js-313ced2a7e13801eb0f8e5ed0057e568`
- CSS vs Three.js performance: `https://www.notion.so/labrys/3D-Cube-CSS-vs-Three-js-30bced2a7e138077b50ce60c97faddd2`
- Local plugin files: `/Users/simontanna/repos/github/lll-animation-plugin/`
