---
name: workshop-guide
description: >
  LLL Animation Workshop navigator. Provides task-by-task guidance, acceptance criteria,
  gotcha warnings, and phase transition help for the 3-hour rolling cube workshop.
  Use this skill whenever a participant asks "what do I do next?", "I'm stuck",
  "what should I work on?", asks about a specific task number (e.g. "help with 1.3"),
  wants to check if their work is done, or needs to understand the workshop structure.
  Also use when someone mentions Phase 1, Phase 2, or any task by number.
user-invocable: true
argument-hint: "[task-id] e.g. 1.3, 2.1, or blank for overview"
---

# LLL Animation Workshop Guide

A 3-hour workshop building a rolling 3D isometric cube in two phases. Participants receive a starter repo with Vite + TypeScript, an isometric diamond grid background, and all dependencies pre-installed.

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

**Goal:** Create a static 3D cube using CSS transforms with minimal HTML elements.

**Acceptance criteria:**
- A static, correctly-formed 3D cube visible in the browser
- All 6 faces in the right positions
- Distinguishable shading on each face
- Uses CSS custom properties for dimensions

**Key concepts:** `transform-style: preserve-3d` (not inherited — think about which elements need it), `transform-origin`, `color-mix()` for face shading from a single base colour.

**Reference:** Julia Miocene — 3D Cube With CSS (2-div + pseudo-element approach)

---

### Task 1.2 — Apply Isometric Projection

**Goal:** Rotate the cube into true isometric orientation matching the starter grid.

**Acceptance criteria:**
- Classic "corner-on" 3D view with three equally foreshortened visible faces
- Aligns with the diamond grid lines from the boilerplate
- No perspective distortion — parallel lines stay parallel

**Key concepts:** Rotation angles derived from `atan(1/sqrt(2))` = 35.264 degrees. Applied to a container element, not individual faces. No CSS `perspective` property (that creates vanishing-point distortion).

**Gotcha:** Watch out for — Rotation axis confusion (Task 1.2 gotcha)

---

### Task 1.3 — Implement Rolling Animation with GSAP

**Goal:** Animate the cube tipping over its bottom edge, translating one grid cell per roll.

**Acceptance criteria:**
- Each roll rotates 90 degrees around one bottom edge
- Cube translates during the roll (lands on next grid cell)
- Looks like physical tipping, not sliding or spinning
- Uses GSAP (not CSS keyframes)
- Pleasant timing with pause between rolls

**Key concepts:** Each roll is a basic 90-degree rotation. After each roll, **reset** — zero the rotation and update the element's position to the new grid cell. This reset-per-roll pattern is the core technique. Chain rolls with `onComplete` callbacks — don't pre-build a long timeline since directions are random.

**Gotchas:** Watch out for — Rotation axis confusion, Cumulative rotation state, Grid snapping after rolls

---

### Task 1.4 — Random Direction Selection

**Goal:** Randomly choose from 4 isometric directions after each roll.

**Acceptance criteria:**
- Direction chosen randomly from: up-right, down-right, down-left, up-left
- Cube wanders the grid unpredictably
- Direction logic accounts for the isometric coordinate system

**Key concepts:** Define directions as data (rotation axis + translation offset per direction). Separate animation concerns from direction/movement logic.

---

### Task 1.5 — Window Boundary Enforcement

**Goal:** Prevent the cube from rolling off-screen.

**Acceptance criteria:**
- Cube never partially or fully off-screen after a roll
- Naturally changes direction at edges and corners

**Key concepts:** Track logical grid position. Filter available directions before each roll — in corners, multiple directions may be invalid. Don't just re-roll once; filter the list.

---

### Task 1.6 — OPTIONAL: Cursor Following

**Goal:** Bias direction selection toward the cursor position.

**Acceptance criteria:**
- Cube visibly trends toward the cursor over time
- Still has organic/random movement (bias, not deterministic path)
- Reverts to random when cursor is inactive

**Key concepts:** Calculate vector from cube to cursor, find the closest isometric direction, use weighted random selection.

---

### Task 2.1 — Create the Cube (Geometry + Material)

**Goal:** Create a 3D cube in the pre-initialised Three.js scene.

**Acceptance criteria:**
- Shaded 3D cube visible in the Three.js scene, positioned on the grid

**Key concepts:** `BoxGeometry` + `MeshBasicMaterial`. Use an array of 6 `MeshBasicMaterial` instances for per-face colour — this matches the CSS approach and keeps things simple (no scene lighting needed).

---

### Task 2.2 — Understand the Isometric Camera

**Goal:** Understand the boilerplate's `OrthographicCamera` setup. **No code required.**

**Acceptance criteria:**
- Can articulate how camera maps world-space axes to screen-space diagonals

**Key concepts:** Unlike CSS (rotate the scene), Three.js positions the camera for isometric view. World X/Z axes appear diagonal on screen. The camera has no perspective distortion.

---

### Task 2.3 — Implement Rolling Animation with GSAP

**Goal:** Animate rolling using Three.js objects and GSAP.

**Acceptance criteria:**
- Cube rolls convincingly — tipping over edges, landing on grid cells
- Random wandering with pauses between rolls
- Same visual quality as Phase 1

**Key concepts:** The Object3D pivot technique — create a temporary pivot at the rolling edge, `attach()` the cube to it, rotate the pivot, then `scene.attach(cube)` to detach. Target `mesh.rotation` and `mesh.position` directly with GSAP (not dot-path syntax on the mesh). Need a render loop (rAF, `onUpdate`, or `gsap.ticker`).

**Gotchas:** Watch out for — Pivot point in Three.js, GSAP + render loop, Euler rotation order, Grid snapping, `attach()` vs `add()`

---

### Task 2.4 — Window Boundary Enforcement

**Goal:** Keep the cube within the camera's visible area.

**Acceptance criteria:**
- Cube stays visible at all times

**Key concepts:** The camera's `left/right/top/bottom` are camera-space, not world-space. Simplest approach: track logical grid position and define bounds in grid coordinates. Alternative: unproject frustum corners onto the ground plane.

---

### Task 2.5 — OPTIONAL: Cursor Following

**Goal:** Bias direction toward mouse position, same as Phase 1.

**Acceptance criteria:**
- Same as Task 1.6

**Key concepts:** Use `Raycaster` or `Vector3.unproject()` to convert screen coords to world coords. With orthographic camera, the math is simpler. Direction-biasing logic from Phase 1 carries over directly.

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

**What you can reuse directly:**
- Direction selection logic and boundary filtering
- The `onComplete` chaining pattern for random roll sequencing
- Grid position tracking (integer coordinates)
- Cursor-bias weighting algorithm

**What changes fundamentally:**
- How you achieve the pivot point (biggest change)
- How rotation state is managed (reset rotation to clean values after each roll)
- You need to manage the render loop explicitly

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

---

## Responding to `/guide`

When invoked without arguments, give an overview of where the participant likely is (ask what they're working on if unclear) and suggest the next task.

When invoked with a task ID (e.g. `/guide 1.3`):
1. Show the task's goal and acceptance criteria
2. List relevant gotchas for that task
3. If they seem stuck, offer the key conceptual hint without giving away the implementation
4. If they think they're done, walk through acceptance criteria as a checklist

When a participant says "I'm stuck" without a task ID, ask what they're seeing (error? visual bug? doesn't move?) and use the gotcha table to diagnose.
