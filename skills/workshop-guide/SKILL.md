---
name: workshop-guide
description: >
  LLL Animation Workshop navigator. Provides task-by-task guidance, acceptance criteria,
  gotcha warnings, and phase transition help for the 3-hour rolling cube workshop.
  This skill should be used when a participant asks "what do I do next?", "I'm stuck",
  "what should I work on?", asks about a specific task number (e.g. "help with 1.3"),
  wants to check if their work is done, or needs to understand the workshop structure.
  Also triggered when someone mentions Phase 1, Phase 2, or any task by number.
user-invocable: true
argument-hint: "[task-id] e.g. 1.3, 2.1, or blank for overview"
---

# LLL Animation Workshop Guide

A 3-hour workshop building a rolling 3D isometric cube in two phases.

**Tech stack:** Vite + TypeScript, HTML, CSS, vanilla JS/TS, GSAP, Three.js

Participants receive a starter repo with a working Vite/TS app, an isometric diamond grid background, and all dependencies pre-installed.

**End result:** A mint/teal 3D cube in isometric view that continuously rolls across the grid — tumbling face-to-face in random directions, bouncing off window boundaries.

## Workshop Structure

| Phase | Duration | Stack | Focus |
|-------|----------|-------|-------|
| Phase 1 | ~1.5 hours | CSS + GSAP | Build cube with CSS 3D transforms, animate rolling with GSAP |
| Phase 2 | ~1.5 hours | Three.js + GSAP | Rebuild cube with Three.js geometry, animate with GSAP |

Both phases share the same end behaviour — the difference is the rendering approach.

---

## Task Map

```
Phase 1 — CSS + GSAP
  1.1  Construct the 3D Cube (HTML + CSS)
  1.2  Apply Isometric Projection
  1.3  Implement Rolling Animation with GSAP
  1.4  Random Direction Selection
  1.5  Window Boundary Enforcement
  1.6  OPTIONAL: Cursor Following

Phase 2 — Three.js + GSAP
  2.1  Create the Cube (Geometry + Material)
  2.2  Understand the Isometric Camera (read-only, no code)
  2.3  Implement Rolling Animation with GSAP
  2.4  Window Boundary Enforcement
  2.5  OPTIONAL: Cursor Following
```

### Dependencies

- **1.1 → 1.2**: Cube must exist before it can be rotated into isometric view
- **1.2 → 1.3**: Isometric orientation must be set before rolling (roll directions depend on it)
- **1.3 → 1.4**: Rolling animation must work for one direction before randomising
- **1.4 → 1.5**: Random movement must exist before boundary logic makes sense
- **1.5 → 1.6**: Boundaries must work before cursor-biasing is layered on
- **2.1 → 2.2 → 2.3**: Need cube in scene, understand camera, then animate
- **2.3 → 2.4 → 2.5**: Same progression as Phase 1

---

## Task Details

### Task 1.1 — Construct the 3D Cube

**Goal:** Create a static 3D cube using CSS transforms with a 3-level HTML hierarchy (scene container, cube container, 6 face divs).

**Acceptance criteria:**
- A static, correctly-formed 3D cube visible in the browser
- All 6 faces in the right positions
- Distinguishable shading on each face
- Uses CSS custom properties for dimensions

**Key concepts:** `transform-style: preserve-3d` (not inherited — set on every element whose children participate in 3D space), rotate + `translateZ(halfSize)` face positioning pattern, `color-mix()` for face shading from a single base colour.

**Skill:** `phase1-css-cube` — full Desandro 6-div technique, code examples, and `preserve-3d` deep dive

**Reference:** [Intro to CSS 3D Transforms — Cube (David DeSandro)](https://3dtransforms.desandro.com/cube)

---

### Task 1.2 — Apply Isometric Projection

**Goal:** Rotate the cube into true isometric orientation matching the starter grid.

**Acceptance criteria:**
- Classic "corner-on" 3D view with three equally foreshortened visible faces
- Aligns with the diamond grid lines from the boilerplate
- No perspective distortion — parallel lines stay parallel

**Key concepts:** Rotation angles derived from `atan(1/sqrt(2))` = 35.264 degrees. Applied to a container element, not individual faces. No CSS `perspective` property — the Desandro tutorial uses perspective for a vanishing-point look, but isometric projection is orthographic (parallel lines stay parallel). The `.scene` container needs `preserve-3d` so the cube's 3D structure passes through.

**Skill:** `phase1-css-cube` — isometric projection section with full CSS

**Gotcha:** Watch out for — Rotation axis confusion, using `perspective` when isometric requires none

---

### Task 1.3 — Implement Rolling Animation with GSAP

**Goal:** Animate the cube tipping over its bottom edge, translating one grid cell per roll.

**Acceptance criteria:**
- Each roll rotates 90 degrees around one bottom edge
- Cube translates during the roll (lands on next grid cell)
- Looks like physical tipping, not sliding or spinning
- Uses GSAP (not CSS keyframes)
- Pleasant timing with pause between rolls

**Key concepts:** Each roll is a basic 90-degree rotation. There are 4 possible roll directions in isometric space: top-right, bottom-right, bottom-left, top-left — each corresponds to a different rotation axis and translation vector. After each roll, **reset** — zero the rotation and update the element's position to the new grid cell. This reset-per-roll pattern is the core technique. Chain rolls with `onComplete` callbacks — don't pre-build a long timeline since directions are random. Use `gsap.delayedCall(pauseDuration, rollNext)` for pauses between rolls. Easing: `"power1.out"` (GSAP default) gives natural deceleration; `"power2.inOut"` gives a more physical roll-and-settle feel.

**Skills:** `isometric-rolling-cube` for rolling technique, `gsap-expert` for GSAP API

**Gotchas:** Watch out for — Rotation axis confusion, Cumulative rotation state, Grid snapping after rolls

---

### Task 1.4 — Random Direction Selection

**Goal:** Randomly choose from 4 isometric directions after each roll.

**Acceptance criteria:**
- Direction chosen randomly from: top-right, bottom-right, bottom-left, top-left
- Cube wanders the grid unpredictably
- Direction logic accounts for the isometric coordinate system (diagonal movement on screen corresponds to axis-aligned movement on the grid)

**Key concepts:** Define directions as data (rotation axis + translation offset per direction). Separate animation concerns from direction/movement logic.

**Skill:** `isometric-rolling-cube` — direction data tables for both CSS and Three.js

---

### Task 1.5 — Window Boundary Enforcement

**Goal:** Prevent the cube from rolling off-screen.

**Acceptance criteria:**
- Cube never partially or fully off-screen after a roll
- Naturally changes direction at edges and corners

**Key concepts:** Track logical grid position (not just pixel position). Filter available directions before each roll — in corners, multiple directions may be invalid. Don't just re-roll once; filter the list. The isometric grid means "out of bounds" checks need to consider diagonal screen-space movement.

**Skill:** `isometric-rolling-cube` — grid snapping and boundary logic

---

### Task 1.6 — OPTIONAL: Cursor Following

**Goal:** Bias direction selection toward the cursor position.

**Acceptance criteria:**
- Cube visibly trends toward the cursor over time
- Still has organic/random movement (bias, not deterministic path)
- Reverts to random when cursor is inactive or not over the viewport

**Key concepts:** Calculate vector from cube to cursor, determine which of the 4 isometric directions best aligns with that vector, use weighted random selection (higher probability for cursor-aligned directions feels more natural than always picking the closest).

**Skill:** `isometric-rolling-cube` — direction selection logic

---

### Task 2.1 — Create the Cube (Geometry + Material)

**Goal:** Create a 3D cube in the pre-initialised Three.js scene.

**Acceptance criteria:**
- Shaded 3D cube visible in the Three.js scene, positioned on the grid

**Key concepts:** `BoxGeometry` + `MeshBasicMaterial`. Use an array of 6 `MeshBasicMaterial` instances for per-face colour (optional — can reuse the same material) — this matches the CSS approach and keeps things simple (no scene lighting needed). The boilerplate scene already includes a camera — just add the mesh.

**Skills:** `threejs-fundamentals` for scene setup and BoxGeometry, `threejs-materials` for MeshBasicMaterial

---

### Task 2.2 — Understand the Isometric Camera

**Goal:** Understand the boilerplate's `OrthographicCamera` setup. **No code required.**

**Acceptance criteria:**
- Can articulate how camera maps world-space axes to screen-space diagonals

**Key concepts:** Unlike CSS (where the scene is rotated), Three.js positions the camera for isometric view. `OrthographicCamera` has no perspective distortion — parallel lines stay parallel, matching the CSS phase. World X/Z axes appear diagonal on screen. Understanding the camera's orientation helps reason about which Three.js axes correspond to which screen-space directions.

**Skill:** `threejs-fundamentals` for camera and coordinate system concepts

---

### Task 2.3 — Implement Rolling Animation with GSAP

**Goal:** Animate rolling using Three.js objects and GSAP.

**Acceptance criteria:**
- Cube rolls convincingly — tipping over edges, landing on grid cells
- Random wandering with pauses between rolls
- Same visual quality as Phase 1

**Key concepts:** The Object3D pivot technique — create a temporary pivot at the rolling edge, `pivot.attach(cube)` to reparent while preserving world transform, rotate the pivot with GSAP, then `scene.attach(cube)` to detach. Target sub-objects directly: `gsap.to(mesh.rotation, { x: ... })` and `gsap.to(mesh.position, { x: ..., z: ... })` — not dot-path syntax on the mesh. Need a render loop (rAF, `onUpdate`, or `gsap.ticker.add(() => renderer.render(scene, camera))`).

**Skills:** `isometric-rolling-cube` for pivot technique and reparenting cycle, `gsap-expert` for GSAP API

**Gotchas:** Watch out for — Pivot point in Three.js, GSAP + render loop, Euler rotation order, Grid snapping, `attach()` vs `add()`

---

### Task 2.4 — Window Boundary Enforcement

**Goal:** Keep the cube within the camera's visible area.

**Acceptance criteria:**
- Cube stays visible at all times

**Key concepts:** The camera's `left/right/top/bottom` are camera-space, not world-space — for a rotated isometric camera, these don't directly correspond to ground-plane boundaries. Simplest approach: track logical grid position and define bounds in grid coordinates, avoiding camera-to-world projection math entirely. Alternative: unproject frustum corners onto the ground plane for precise world-space bounds.

**Skill:** `isometric-rolling-cube` — boundary logic and grid position tracking

---

### Task 2.5 — OPTIONAL: Cursor Following

**Goal:** Bias direction toward mouse position, same as Phase 1.

**Acceptance criteria:**
- Same as Task 1.6

**Key concepts:** Use Three.js `Raycaster` or `Vector3.unproject()` to convert screen coordinates to world coordinates — with an orthographic camera, the math is simpler than with a perspective camera. Direction-biasing logic from Phase 1 carries over almost directly once cursor position is in world-space.

**Skills:** `isometric-rolling-cube` for direction selection, `threejs-fundamentals` for coordinate conversion

---

## Phase Transition Guide (Phase 1 → Phase 2)

When starting Phase 2, here's what carries over and what changes:

| Concept | Phase 1 (CSS) | Phase 2 (Three.js) |
|---------|--------------|-------------------|
| Cube construction | HTML elements + CSS transforms | `BoxGeometry` + `Material` |
| Isometric view | Container rotation (`rotateX(35.264deg) rotateZ(45deg)`) | `OrthographicCamera` positioned at isometric angle |
| Pivot point | Basic rotation + reset in `onComplete` | Temporary `Object3D` at edge + `attach()` reparenting |
| Roll animation | GSAP tweens CSS properties | GSAP tweens `mesh.rotation` / `mesh.position` sub-objects |
| Render trigger | Browser handles CSS rendering | Need explicit render loop (rAF or `gsap.ticker`) |
| Direction data | Same 4 directions | Same 4 directions (but mapped to Three.js XZ axes) |
| Boundary logic | Viewport pixel bounds | Grid coordinate bounds or frustum unprojection |
| Cursor following | Screen-space vector math | Needs screen-to-world coordinate conversion |

**Reusable directly:**
- Direction selection logic and boundary filtering
- The `onComplete` chaining pattern for random roll sequencing
- Grid position tracking (integer coordinates)
- Cursor-bias weighting algorithm

**Changes fundamentally:**
- How the pivot point is achieved (biggest change)
- How rotation state is managed (reset rotation to clean values after each roll)
- The render loop must be managed explicitly

---

## Gotcha Quick Reference

| Gotcha | Relevant Tasks | What Goes Wrong |
|--------|---------------|-----------------|
| Rotation axis confusion | 1.2, 1.3, 2.3 | Visual "right" on screen is diagonal in world-space. Each roll direction maps to a different axis. |
| Cumulative rotation state | 1.3, 2.3 | After several rolls, local axes have shifted. Must track or reset orientation. |
| Euler rotation order | 2.3 | Three.js defaults to 'XYZ'. Multiple 90-degree rolls across axes → drift. Reset rotation to clean multiples of PI/2 after each roll. |
| Pivot point in Three.js | 2.3 | No `transform-origin` equivalent. Must use parent `Object3D` technique. Use `attach()` not `add()`. |
| GSAP + render loop | 2.3 | GSAP tweens values but doesn't trigger Three.js renders. Need rAF, `onUpdate`, or `gsap.ticker`. |
| Grid snapping after rolls | 1.3, 2.3 | Floating-point drift. Use `gsap.set()` to snap to exact grid coords in `onComplete`. |
| `preserve-3d` not inherited | 1.1 | Must be set on every ancestor that needs to participate in 3D space. |
| Pre-built timeline for random dirs | 1.3, 2.3 | Can't pre-build — directions are random. Use `onComplete` chaining instead. |
| GSAP directional shortcuts on Three.js | 2.3 | `_cw`, `_short` and other GSAP directional shortcuts only work on DOM targets — use raw angle values for Three.js objects. |

---

## Responding to `/workshop-guide`

When invoked without arguments, give an overview of where the participant likely is (ask what they're working on if unclear) and suggest the next task.

When invoked with a task ID (e.g. `/workshop-guide 1.3`):
1. Show the task's goal and acceptance criteria
2. List relevant gotchas for that task
3. Point to the specialized skill for that task (listed under each task's **Skill:** field)
4. If they seem stuck, offer the key conceptual hint without giving away the implementation
5. If they think they're done, walk through acceptance criteria as a checklist

When a participant says "I'm stuck" without a task ID, ask what they're seeing (error? visual bug? doesn't move?) and use the gotcha table to diagnose.

---

## Additional Resources

### Reference Files

- **`references/resources.md`** — External reference links and descriptions for all workshop technologies (Desandro, GSAP, Three.js, isometric projection)

### Related Skills

- `phase1-css-cube` — Desandro 6-div CSS cube technique, face positioning, `preserve-3d`, isometric projection
- `isometric-rolling-cube` — Rolling animation, pivot technique, direction logic, grid snapping
- `gsap-expert` — GSAP timelines, easing, callbacks, ScrollTrigger
- `threejs-fundamentals` — Scene setup, Object3D hierarchy, coordinate systems, BoxGeometry
- `threejs-materials` — MeshBasicMaterial, per-face colours
