# Boundary and Cursor Algorithm Reference

## Direction Filtering Algorithm

Before each roll, check which of the 4 directions would keep the cube in bounds. The steps:

1. Calculate the candidate position for each direction: `(gridX + deltaX, gridY + deltaY)`
2. Check each candidate against the grid bounds
3. Filter the directions list to only valid options
4. Pick randomly from the filtered list

Filtering is essential — re-rolling a single rejected direction is not enough, because in corners multiple directions may be invalid simultaneously.

```js
function getValidDirections(gridX, gridY, bounds) {
  return DIRECTIONS.filter(dir => {
    const nx = gridX + dir.deltaX
    const ny = gridY + dir.deltaY
    return nx >= bounds.minX && nx <= bounds.maxX
        && ny >= bounds.minY && ny <= bounds.maxY
  })
}

function rollNext() {
  const valid = getValidDirections(gridX, gridY, bounds)
  if (valid.length === 0) return // should not happen if bounds are set correctly
  const dir = valid[Math.floor(Math.random() * valid.length)]
  // ... animate roll in dir
}
```

## CSS (Phase 1) — Viewport Pixel Bounds

Track the cube's logical grid position `(gridX, gridY)` and convert to pixel coordinates to check against `window.innerWidth` / `window.innerHeight`. The isometric rotation means screen-space movement is diagonal — a single grid step moves the cube diagonally on screen, so bounds checks must account for both the X and Y pixel displacement per step.

## Three.js (Phase 2) — Grid Coordinate Bounds

The simplest approach: define bounds in grid coordinates and check the logical position directly, avoiding camera-to-world projection math entirely.

The boilerplate's `OrthographicCamera` frustum properties (`left`, `right`, `top`, `bottom`) are in **camera-space**, not world-space — for the rotated isometric camera, these don't map directly to ground-plane boundaries.

Alternative: unproject the frustum corners onto the ground plane for precise world-space bounds, but grid-coordinate bounds are simpler and sufficient for the workshop.

## Cursor Following — Direction Bias Algorithm

1. Calculate the vector from the cube's current position to the cursor position
2. For each of the 4 isometric directions, compute the dot product between the cursor vector and the direction's movement vector
3. Assign higher probability to directions with larger positive dot products
4. Use weighted random selection — directions aligned with the cursor get higher weight, but all valid directions remain possible

A simple weighting scheme: give cursor-aligned directions 3x or 4x the weight of non-aligned directions. This produces a visible trend toward the cursor while preserving organic movement.

### Fallback to Random

When the cursor is inactive or outside the viewport, revert to fully random direction selection (all valid directions weighted equally).

### CSS (Phase 1) — Screen-Space Vector Math

Both the cube position and cursor position are in screen-space pixels. Calculate the direction vector directly from pixel coordinates and compare against the 4 isometric screen-space direction vectors.

### Three.js (Phase 2) — Screen-to-World Conversion

Convert the screen-space cursor position to world-space coordinates before computing the direction vector. With an `OrthographicCamera`, the conversion is simpler than with a perspective camera — use `Vector3.unproject()` or `Raycaster` to project the cursor onto the ground plane (Y=0). The direction-biasing logic from Phase 1 carries over directly once the cursor position is in world-space.
