# Phase 1 — Task Specifications

Full task requirements, hints, and acceptance criteria for Phase 1 (CSS + GSAP) of the LLL Animation Workshop. Sourced from the workshop Notion spec.

---

## Task 1.1 — Construct the 3D Cube (HTML + CSS)

**Goal:** Create a static 3D cube using CSS transforms. The cube is built from HTML elements (divs), not a canvas.

**Requirements:**
- Use a 4-level HTML hierarchy: a position container (x/y movement), a perspective container (static isometric rotation), a cube container (rolling rotation), and 6 child divs — one for each face (front, back, right, left, top, bottom)
- The cube container must establish a shared 3D space for all faces
- Each face is positioned by first rotating it to its orientation, then translating it outward from the centre by half the cube's size
- Each face needs a distinct shade/tint to convey depth — participants choose their own colour palette
- The cube should have equal width, height, and depth (a true cube)
- Use CSS custom properties for dimensions so the cube size is easily tuneable

**Hints:**
- `transform-style: preserve-3d` on the cube container is what makes the faces exist in a shared 3D space rather than being flattened
- The construction pattern for each face is: rotate to the face's orientation, then `translateZ(halfSize)` to push it outward. Transform function order matters — rotate first, then translate
- Side faces (front, back, left, right) rotate around Y. Top and bottom faces rotate around X
- Consider pushing the entire cube back with a negative `translateZ` on the cube container — this centres the cube's visual position on its transform origin, which simplifies animation pivot math later
- `color-mix()` in CSS is one approach for generating face shading from a single base colour
- **3-container design:** Use three nested containers, each with one transform responsibility — a position container (x/y movement), a perspective container (static isometric rotation), and the cube itself (rolling rotation). This separation is essential: GSAP overwrites the full `transform` property when animating, so mixing position and rotation on one element breaks the animation.

**Acceptance Criteria:** A static, correctly-formed 3D cube is visible in the browser with all 6 faces in the right positions and distinguishable shading.

**Reference:** [Intro to CSS 3D Transforms — Cube (David DeSandro)](https://3dtransforms.desandro.com/cube)

---

## Task 1.2 — Apply Isometric Projection

**Goal:** Rotate the cube into a true isometric viewing angle so it aligns with the pre-existing isometric grid in the starter repo.

**Requirements:**
- The cube must be viewed from an isometric perspective — the classic "corner-on" 3D view where all three visible faces are equally foreshortened
- The cube should visually sit on and align with the diamond grid lines provided by the boilerplate
- No perspective distortion — isometric projection is orthographic (parallel lines stay parallel)

**Hints:**
- Isometric projection involves specific rotation angles on two axes. The Wikipedia article on isometric projection describes the math behind these angles
- The Desandro reference uses `perspective` on the scene container to create a natural vanishing-point 3D look. Isometric projection specifically requires NO perspective — parallel lines must stay parallel. Remove or omit the perspective property
- The rotation is applied to a container/scene element, not individual cube faces

**Acceptance Criteria:** The cube appears in the correct isometric orientation matching the grid. No perspective warping — edges that should be parallel remain parallel.

---

## Task 1.3 — Implement Rolling Animation with GSAP

**Goal:** Animate the cube so it "rolls" — tipping over one of its bottom edges to land on an adjacent face, translating one grid cell in the corresponding direction.

**Requirements:**
- Each roll should rotate the cube exactly 90 degrees around one of its bottom edges
- The cube must translate during the roll so it lands precisely on the next grid cell
- The animation should look like a physical box tipping over, not sliding or spinning in place
- Use GSAP to drive the animation (not CSS keyframes)
- The roll duration should feel "pleasant" — smooth and readable, not jarring or sluggish
- After each roll completes, the cube should briefly pause before the next roll begins

**Hints:**
- The critical challenge is managing rotation state. Each roll is a basic 90-degree rotation — the key insight is **resetting** after each roll: zero the rotation and update the element's position to the new grid cell in the `onComplete` callback. Without this reset, cumulative transforms break the animation after 2-3 rolls
- After each 90-degree roll, the cube's orientation has changed — account for cumulative rotation state
- A `gsap.timeline()` is useful for composing each individual roll (combining rotation + translation with easing). Since directions are chosen randomly after each roll, chain rolls dynamically using `onComplete` callbacks — call a `rollNext()` function from `onComplete` to pick a new direction and build the next roll. Use `delay` or `gsap.delayedCall()` for the pause between rolls
- GSAP's default ease (`"power1.out"`) has natural deceleration that suits a "landing" feel. `"power2.inOut"` gives a more physical roll sensation
- There are 4 possible roll directions in isometric space: top-right, bottom-right, bottom-left, top-left. Each corresponds to a different rotation axis and translation vector

**Acceptance Criteria:** The cube rolls convincingly — tipping over its edge, not rotating in place. It moves one grid cell per roll and lands aligned to the grid.

**Primary skill:** `isometric-rolling-cube` for rolling technique details.

---

## Task 1.4 — Random Direction Selection

**Goal:** After each roll, randomly choose the next direction from the 4 possible isometric directions.

**Requirements:**
- The direction for each roll is chosen randomly from: top-right, bottom-right, bottom-left, top-left
- Randomisation should feel natural — the cube wanders across the grid
- The direction logic must account for the isometric coordinate system (diagonal movement on screen corresponds to axis-aligned movement on the grid)

**Hints:**
- Define the 4 directions as data (rotation axis + translation offset per direction)
- Separate animation concerns from direction/movement logic

**Acceptance Criteria:** The cube randomly picks a new direction each roll and wanders the grid unpredictably.

---

## Task 1.5 — Window Boundary Enforcement

**Goal:** Prevent the cube from rolling off-screen.

**Requirements:**
- Before each roll, check whether the chosen direction would move the cube outside the visible viewport
- If the chosen direction would go out of bounds, select a different valid direction
- The cube should never be partially or fully off-screen after a roll

**Hints:**
- Track the cube's logical grid position (not just pixel position) to simplify boundary checks
- The isometric grid means "out of bounds" checks need to consider diagonal screen-space movement
- Consider corner cases — filter the available directions list rather than just re-rolling once

**Acceptance Criteria:** The cube never leaves the visible viewport. It naturally changes direction when it reaches edges or corners.

---

## Task 1.6 — OPTIONAL: Cursor Following

**Goal:** Bias the random direction selection toward the user's cursor position.

**Requirements:**
- Track the mouse/cursor position on the page
- When choosing the next roll direction, prefer directions that move the cube closer to the cursor
- Random direction remains the fallback — this is a bias, not a deterministic path
- If the cursor is not moving or is not over the viewport, revert to fully random behaviour

**Hints:**
- Calculate the vector from the cube's current position to the cursor, then determine which of the 4 isometric directions best aligns with that vector
- A weighted random selection (higher probability for cursor-aligned directions) feels more natural than always picking the closest direction

**Acceptance Criteria:** The cube visibly trends toward the cursor over time but still has organic/random movement. Removing the cursor reverts to random wandering.
