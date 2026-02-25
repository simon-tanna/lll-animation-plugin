---
name: isometric-rolling-cube
description: >
  This skill should be used when the user is working on rolling cube animation, edge-pivoting,
  isometric cube movement, transform-origin animation, Object3D reparenting for rotation,
  grid-based cube wandering, window boundary enforcement, direction selection logic,
  cursor following, or weighted random direction bias — even if they don't explicitly
  mention "rolling cube." Covers both CSS transform-origin and Three.js Object3D pivot
  approaches, GSAP animation chaining, cumulative rotation tracking, grid snapping,
  boundary checking, and cursor-biased direction selection.
user-invocable: true
argument-hint: "[topic] e.g. pivot, directions, reparenting, snapping"
---

# Isometric Rolling Cube

A cube "rolls" by tipping 90 degrees over one of its bottom edges, translating one grid cell in that direction. This is fundamentally different from rotating around the centre — the pivot point must be at the contact edge, not the object's origin.

This skill covers the technique for both CSS (Phase 1) and Three.js (Phase 2) implementations, with GSAP driving the animation in both cases.

## Invocation Behaviour

When invoked without a specific topic, prompt for context:

> Which aspect of rolling animation are you working on?
>
> | Topic           | Covers                                                    |
> | --------------- | --------------------------------------------------------- |
> | Rolling / pivot | CSS transform-origin or Three.js Object3D pivot technique |
> | Directions      | The 4 isometric roll directions and data tables           |
> | Boundaries      | Window boundary enforcement and direction filtering       |
> | Cursor          | Cursor-following bias and weighted random selection       |
> | Snapping        | Grid snapping and floating-point drift prevention         |
> | Reparenting     | Three.js `attach()` vs `add()` and the pivot cycle        |
>
> Or describe what you need help with.

When a specific topic is identified, jump directly to the relevant section. For CSS cube construction (faces, preserve-3d, isometric projection), delegate to the `phase1-css-cube` skill. For GSAP API specifics, delegate to the `gsap-expert` skill.

## The 4 Isometric Roll Directions

In isometric view, the cube moves diagonally on screen. Each screen direction maps to a world-axis movement:

| Screen Direction | World Direction (XZ) | Grid Delta |
| ---------------- | -------------------- | ---------- |
| Top-Right        | +X                   | (1, 0)     |
| Bottom-Right     | +Z                   | (0, 1)     |
| Bottom-Left      | -X                   | (-1, 0)    |
| Top-Left         | -Z                   | (0, -1)    |

For a unit cube (side length `S`), each direction has a corresponding **rotation axis**, **pivot offset**, and **translation vector**.

### Direction Data (Three.js — Y-up, XZ ground plane)

For a cube centred at `(gx, S/2, gz)`:

| Direction | Translation | Pivot Offset from Centre | Rotation Axis | Angle |
| --------- | ----------- | ------------------------ | ------------- | ----- |
| +X        | (S, 0, 0)   | (+S/2, -S/2, 0)          | (0, 0, -1)    | PI/2  |
| -X        | (-S, 0, 0)  | (-S/2, -S/2, 0)          | (0, 0, +1)    | PI/2  |
| +Z        | (0, 0, S)   | (0, -S/2, +S/2)          | (+1, 0, 0)    | PI/2  |
| -Z        | (0, 0, -S)  | (0, -S/2, -S/2)          | (-1, 0, 0)    | PI/2  |

The rotation axis can be derived: `cross(direction, downVector)` where `downVector = (0, -1, 0)`.

The pivot offset is always: half a cube-width toward the roll direction on the ground axis, plus half a cube-width downward (to the bottom edge).

### Direction Data (CSS — pre-isometric flat plane)

In the CSS approach, the isometric rotation (`rotateX(35.264deg) rotateZ(45deg)`) is applied to a **container** element. The cube rolls in the container's local coordinate system (flat plane before rotation), so:

| Direction            | transform-origin  | Rotation        | Translation    |
| -------------------- | ----------------- | --------------- | -------------- |
| Right (+X)           | bottom-right edge | rotateZ(-90deg) | translateX(+S) |
| Left (-X)            | bottom-left edge  | rotateZ(+90deg) | translateX(-S) |
| Forward (-Y screen)  | bottom-front edge | rotateX(+90deg) | translateY(-S) |
| Backward (+Y screen) | bottom-back edge  | rotateX(-90deg) | translateY(+S) |

The `transform-origin` values need to target the specific bottom edge in 3D space. For a cube of side `S` centred at its own origin:

- Right: `transform-origin: S S/2 0` (right face, vertical centre, front)
- Left: `transform-origin: 0 S/2 0`
- Forward: `transform-origin: S/2 S/2 -S/2`
- Backward: `transform-origin: S/2 S/2 S/2`

The exact values depend on how the cube's faces are positioned (which in turn depends on the construction approach). The principle is always the same: origin goes to the bottom edge in the roll direction.

---

## Phase 1: CSS Rolling Technique

### How It Works

Each roll is a simple 90-degree rotation animated with GSAP. The key is the **reset step** after each roll — this keeps each animation starting from a clean state.

For each roll:

1. **Animate** a 90-degree rotation in the roll direction (using GSAP)
2. **On complete (the reset):** move the element to its new grid position, zero out the rotation, and the cube is ready for the next roll

### The Reset Step (Core Technique)

After a roll completes, the cube has rotated 90 degrees. Absorb this into the base state:

- Move the element to its new grid position (CSS `left`/`top` or `translate`)
- Zero out the roll rotation
- This prevents cumulative transform stacking

Without this reset, each successive roll compounds on the previous rotation and the animation breaks after 2-3 rolls. The reset is what makes the entire approach work — each roll starts from a clean, predictable state.

### Cumulative State

Because each roll resets to a clean state, no complex rotation tracking is needed. The logical grid position `(gridX, gridY)` is the only state to maintain between rolls — update it in the `onComplete` callback after each reset.

---

## Phase 2: Three.js Pivot Technique

### The Object3D Reparenting Cycle

Three.js has no `transform-origin` equivalent. Instead, create a temporary pivot `Object3D` at the rolling edge and make the cube a child of it. Rotating the pivot then rotates the cube around that edge.

The full cycle for one roll:

**Step 1 — Create pivot at the rolling edge:**
Position a new `Object3D` at the cube's bottom edge in the roll direction.

**Step 2 — Attach cube to pivot:**
Use `pivot.attach(cube)` — this preserves the cube's world-space position while making it a child of the pivot. The cube does not visually move.

**Step 3 — Animate pivot rotation:**
Use GSAP to rotate the pivot 90 degrees around the appropriate axis. The cube, as a child, orbits around the pivot point.

**Step 4 — Detach cube from pivot:**
Use `scene.attach(cube)` — this moves the cube back to the scene's children while preserving its current world position and rotation.

**Step 5 — Clean up:**
Remove the pivot from the scene. Snap the cube's position to exact grid coordinates.

### Why `attach()` Not `add()`

This is the single most important API distinction in the entire implementation.

- `Object3D.add(child)` — reparents the child but **keeps its local transform unchanged**. Since the local transform is now interpreted relative to the new parent, the child's world position changes. The cube would visually jump.
- `Object3D.attach(child)` — reparents the child and **recalculates its local transform** so that its world position stays the same. Internally it computes: `newLocal = inverse(newParent.worldMatrix) * oldParent.worldMatrix * oldLocal`.

Use `attach()` for both directions: cube-to-pivot and pivot-back-to-scene.

### Rotation Reset After Each Roll

After each roll's reparenting cleanup, reset the cube's rotation to clean values. Since each roll is exactly 90 degrees, all rotation components should be exact multiples of PI/2.

**Recommended approach:** Reset Euler angles to clean values after each roll in the `onComplete` callback. Round each rotation component to the nearest multiple of PI/2. This keeps every roll starting from a predictable state — no cumulative drift, no gimbal lock issues.

**Optional advanced approach:** Use quaternions (`Quaternion.premultiply()`) to track cumulative rotation if you need precise face-orientation tracking (e.g., knowing which face is on top). This is not needed for the workshop's rolling behaviour.

---

## GSAP Animation Patterns

### Per-Roll Timeline with onComplete Chaining

Each roll is an independent animation. Since directions are chosen randomly, you cannot pre-build a long timeline. Instead, chain rolls dynamically:

1. Pick a random valid direction
2. Create a short timeline (or single tween) for one roll
3. In the `onComplete` callback, clean up the pivot/transform state, then call the roll function again (possibly after a delay)

Use `gsap.delayedCall(pauseDuration, rollNext)` to insert pauses between rolls — this is cleaner than adding empty delays to timelines.

### Targeting Three.js Objects with GSAP

Target the sub-object, not the mesh itself:

- Position: `gsap.to(mesh.position, { x: ..., z: ... })`
- Rotation: `gsap.to(pivot.rotation, { z: angle })`

Do NOT use dot-path syntax on the mesh: `gsap.to(mesh, { "rotation.x": angle })` — this does not work with Three.js objects.

### Easing

- `"power2.inOut"` gives a natural roll-and-settle feel
- `"power1.out"` (GSAP default) also works well for a lighter landing
- Experiment encouraged — the "right" ease is subjective

### Render Loop

GSAP tweens values but does not trigger Three.js renders. Three options:

1. **Continuous rAF loop** — `requestAnimationFrame` that calls `renderer.render()` every frame. Simple, always works.
2. **GSAP onUpdate** — call `renderer.render()` inside each tween's `onUpdate`. Renders only during animation but misses static scene updates.
3. **gsap.ticker** — `gsap.ticker.add(() => renderer.render(scene, camera))`. Synchronises rendering with GSAP's update cycle. Good middle ground.

---

## Grid Snapping

Floating-point arithmetic causes gradual drift after many rolls. After each roll completes (in the `onComplete` callback), snap the cube's position to exact grid coordinates:

- **Three.js:** `gsap.set(mesh.position, { x: Math.round(mesh.position.x), z: Math.round(mesh.position.z) })` — using `gsap.set()` rather than direct assignment keeps GSAP's internal state consistent
- **CSS:** Set the element's position to `gridX * cellSize`, `gridY * cellSize` and clear any transform residue

Track position as integer grid coordinates `(gridX, gridY)` alongside the rendered position. Update the grid coordinates after each roll, and snap the rendered position to match.

---

## Boundary Enforcement

Prevent the cube from rolling off-screen by filtering valid directions before each roll.

### Direction Filtering Algorithm

Before each roll, check which of the 4 directions would keep the cube in bounds. The steps:

1. Calculate the candidate position for each direction: `(gridX + deltaX, gridY + deltaY)`
2. Check each candidate against the grid bounds
3. Filter the directions list to only valid options
4. Pick randomly from the filtered list

Filtering is essential — re-rolling a single rejected direction is not enough, because in corners multiple directions may be invalid simultaneously.

### CSS (Phase 1) — Viewport Pixel Bounds

Track the cube's logical grid position `(gridX, gridY)` and convert to pixel coordinates to check against `window.innerWidth` / `window.innerHeight`. The isometric rotation means screen-space movement is diagonal — a single grid step moves the cube diagonally on screen, so bounds checks must account for both the X and Y pixel displacement per step.

### Three.js (Phase 2) — Grid Coordinate Bounds

The simplest approach: define bounds in grid coordinates and check the logical position directly, avoiding camera-to-world projection math entirely. The boilerplate's `OrthographicCamera` frustum properties (`left`, `right`, `top`, `bottom`) are in **camera-space**, not world-space — for the rotated isometric camera, these don't map directly to ground-plane boundaries.

Alternative: unproject the frustum corners onto the ground plane for precise world-space bounds, but grid-coordinate bounds are simpler and sufficient for the workshop.

---

## Cursor Following

Bias direction selection toward the cursor position. This is a weighted preference, not a deterministic path — the cube should still wander organically.

### Direction Bias Algorithm

1. Calculate the vector from the cube's current position to the cursor position
2. For each of the 4 isometric directions, compute the dot product between the cursor vector and the direction's movement vector
3. Assign higher probability to directions with larger positive dot products
4. Use weighted random selection — directions aligned with the cursor get higher weight, but all valid directions remain possible

A simple weighting scheme: give cursor-aligned directions 3x or 4x the weight of non-aligned directions. This produces a visible trend toward the cursor while preserving organic, random-looking movement.

### Fallback to Random

When the cursor is inactive or outside the viewport, revert to fully random direction selection (all valid directions weighted equally).

### CSS (Phase 1) — Screen-Space Vector Math

Both the cube position and cursor position are in screen-space pixels. Calculate the direction vector directly from pixel coordinates and compare against the 4 isometric screen-space direction vectors.

### Three.js (Phase 2) — Screen-to-World Conversion

Convert the screen-space cursor position to world-space coordinates before computing the direction vector. With an `OrthographicCamera`, the conversion is simpler than with a perspective camera — use `Vector3.unproject()` or `Raycaster` to project the cursor onto the ground plane (Y=0). The direction-biasing logic from Phase 1 carries over directly once the cursor position is in world-space.

---

## Common Pitfalls

| Pitfall                                      | Symptom                               | Fix                                                                      |
| -------------------------------------------- | ------------------------------------- | ------------------------------------------------------------------------ |
| Pivot at centre, not edge                    | Cube spins in place                   | Position pivot at bottom edge using offset formula                       |
| Using `add()` instead of `attach()`          | Cube jumps when reparented            | Always use `attach()` for both directions                                |
| Euler angle accumulation                     | Unexpected rotation after 3-4 rolls   | Reset rotation to clean multiples of PI/2 after each roll                |
| No grid snap                                 | Cube drifts off grid over time        | `gsap.set()` to round position in `onComplete`                           |
| Pre-built timeline for random directions     | Can't randomise; timeline is fixed    | Use `onComplete` chaining, one roll per timeline                         |
| GSAP directional shortcuts (`_cw`, `_short`) | Ignored or breaks on Three.js objects | These only work on DOM targets — use raw angle values                    |
| No render loop                               | Scene doesn't update visually         | Add `gsap.ticker.add(() => renderer.render(...))`                        |
| Modifying boilerplate camera/scene           | Breaks grid alignment                 | The starter repo's camera and grid are pre-configured — don't touch them |

## See Also

- `phase1-css-cube` — CSS cube construction (Desandro 6-div approach, face transforms, preserve-3d, isometric projection)
- `workshop-guide` — Task progression, acceptance criteria, phase transitions
- `gsap-expert` — Full GSAP API reference, timeline patterns, ScrollTrigger
- `threejs-fundamentals` — Scene setup, Object3D hierarchy, coordinate system, math utilities
