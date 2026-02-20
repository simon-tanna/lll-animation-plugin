---
name: isometric-rolling-cube
description: >
  Pivot-point rolling technique for animating a cube tipping over its bottom edge
  in isometric view. Covers both CSS transform-origin and Three.js Object3D pivot
  approaches, GSAP animation chaining, cumulative rotation tracking, and grid snapping.
  Use this skill whenever the user is working on rolling cube animation, edge-pivoting,
  isometric cube movement, transform-origin animation, Object3D reparenting for rotation,
  or grid-based cube wandering — even if they don't explicitly mention "rolling cube."
user-invocable: true
argument-hint: "[topic] e.g. pivot, directions, reparenting, snapping"
---

# Isometric Rolling Cube

A cube "rolls" by tipping 90 degrees over one of its bottom edges, translating one grid cell in that direction. This is fundamentally different from rotating around the centre — the pivot point must be at the contact edge, not the object's origin.

This skill covers the technique for both CSS (Phase 1) and Three.js (Phase 2) implementations, with GSAP driving the animation in both cases.

## The 4 Isometric Roll Directions

In isometric view, the cube moves diagonally on screen. Each screen direction maps to a world-axis movement:

| Screen Direction | World Direction (XZ) | Grid Delta |
|------------------|---------------------|------------|
| Up-Right | +X | (1, 0) |
| Down-Right | +Z | (0, 1) |
| Down-Left | -X | (-1, 0) |
| Up-Left | -Z | (0, -1) |

For a unit cube (side length `S`), each direction has a corresponding **rotation axis**, **pivot offset**, and **translation vector**.

### Direction Data (Three.js — Y-up, XZ ground plane)

For a cube centred at `(gx, S/2, gz)`:

| Direction | Translation | Pivot Offset from Centre | Rotation Axis | Angle |
|-----------|------------|--------------------------|---------------|-------|
| +X | (S, 0, 0) | (+S/2, -S/2, 0) | (0, 0, -1) | PI/2 |
| -X | (-S, 0, 0) | (-S/2, -S/2, 0) | (0, 0, +1) | PI/2 |
| +Z | (0, 0, S) | (0, -S/2, +S/2) | (+1, 0, 0) | PI/2 |
| -Z | (0, 0, -S) | (0, -S/2, -S/2) | (-1, 0, 0) | PI/2 |

The rotation axis can be derived: `cross(direction, downVector)` where `downVector = (0, -1, 0)`.

The pivot offset is always: half a cube-width toward the roll direction on the ground axis, plus half a cube-width downward (to the bottom edge).

### Direction Data (CSS — pre-isometric flat plane)

In the CSS approach, the isometric rotation (`rotateX(35.264deg) rotateZ(45deg)`) is applied to a **container** element. The cube rolls in the container's local coordinate system (flat plane before rotation), so:

| Direction | transform-origin | Rotation | Translation |
|-----------|-----------------|----------|-------------|
| Right (+X) | bottom-right edge | rotateZ(-90deg) | translateX(+S) |
| Left (-X) | bottom-left edge | rotateZ(+90deg) | translateX(-S) |
| Forward (-Y screen) | bottom-front edge | rotateX(+90deg) | translateY(-S) |
| Backward (+Y screen) | bottom-back edge | rotateX(-90deg) | translateY(+S) |

The `transform-origin` values need to target the specific bottom edge in 3D space. For a cube of side `S` centred at its own origin:

- Right: `transform-origin: S S/2 0` (right face, vertical centre, front)
- Left: `transform-origin: 0 S/2 0`
- Forward: `transform-origin: S/2 S/2 -S/2`
- Backward: `transform-origin: S/2 S/2 S/2`

The exact values depend on how the cube's faces are positioned (which in turn depends on the construction approach). The principle is always the same: origin goes to the bottom edge in the roll direction.

---

## Phase 1: CSS Pivot Technique

### How It Works

CSS `transform-origin` sets the point around which transforms are applied. For each roll:

1. **Set origin** to the bottom edge in the roll direction
2. **Animate** a 90-degree rotation around that edge (using GSAP)
3. **On complete:** reset the element — update its position by one grid cell, clear the rotation, and prepare the origin for the next roll

### The Reset Step

After a roll completes, the cube has rotated 90 degrees and translated one cell. You need to "absorb" this into the base state:

- Move the element to its new grid position (CSS `left`/`top` or `translate`)
- Zero out the roll rotation
- This prevents cumulative transform stacking

Without this reset, each successive roll compounds on the previous rotation, and the pivot point calculations break.

### Cumulative State

Track the cube's logical orientation separately from the CSS transform. After each roll, update a rotation tracker (which face is now on top, which face is forward, etc.). This matters because the roll axes are relative to the cube's current orientation — if the cube has already rolled right once, its local axes have shifted.

Two approaches:
- **Face tracking:** maintain which face index is in each position (top, front, right, etc.) — simpler for CSS where you reset transforms each roll
- **Quaternion accumulation:** multiply each roll's quaternion onto a running total — more robust but overkill for the CSS phase

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

### Quaternion vs Euler Rotation Tracking

Three.js defaults to `'XYZ'` Euler order. After multiple 90-degree rolls across different axes, Euler angles produce unexpected results due to gimbal lock and order-dependent composition.

**Recommended approach:** Use quaternions to track cumulative rotation state.

- Each roll produces a quaternion: `new Quaternion().setFromAxisAngle(axis, PI/2)`
- Accumulate with `cumulativeQuat.premultiply(rollQuat)` — `premultiply` applies the new rotation in world space (left-multiply)
- After each roll's reparenting cleanup, set the cube's quaternion to the accumulated value

**Simpler alternative:** Reset Euler angles to clean multiples of PI/2 after each roll. This works if you also maintain a face-tracking table that tells you what the "clean" rotation should be after N rolls in various directions. It avoids quaternion math but requires a lookup table.

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

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Pivot at centre, not edge | Cube spins in place | Position pivot at bottom edge using offset formula |
| Using `add()` instead of `attach()` | Cube jumps when reparented | Always use `attach()` for both directions |
| Euler angle accumulation | Unexpected rotation after 3-4 rolls | Use quaternions or reset to clean values |
| No grid snap | Cube drifts off grid over time | `gsap.set()` to round position in `onComplete` |
| Pre-built timeline for random directions | Can't randomise; timeline is fixed | Use `onComplete` chaining, one roll per timeline |
| GSAP directional shortcuts (`_cw`, `_short`) | Ignored or breaks on Three.js objects | These only work on DOM targets — use raw angle values |
| No render loop | Scene doesn't update visually | Add `gsap.ticker.add(() => renderer.render(...))` |
| Modifying boilerplate camera/scene | Breaks grid alignment | The starter repo's camera and grid are pre-configured — don't touch them |

## See Also

- `css-3d-cube` — CSS cube construction (2-div approach, face transforms, preserve-3d)
- `workshop-guide` — Task progression, acceptance criteria, phase transitions
- `gsap-expert` — Full GSAP API reference, timeline patterns, ScrollTrigger
- `threejs-fundamentals` — Scene setup, Object3D hierarchy, coordinate system, math utilities
