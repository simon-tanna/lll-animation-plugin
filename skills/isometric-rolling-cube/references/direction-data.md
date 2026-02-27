# Direction Data Reference

Full direction tables for both CSS (Phase 1) and Three.js (Phase 2) implementations.

## Three.js Direction Data (Y-up, XZ ground plane)

For a cube centred at `(gx, S/2, gz)`:

| Direction    | World Axis | Translation | Pivot Offset from Centre | Rotation Axis | Angle |
| ------------ | ---------- | ----------- | ------------------------ | ------------- | ----- |
| Top-Right    | +X        | (S, 0, 0)   | (+S/2, -S/2, 0)          | (0, 0, -1)    | PI/2  |
| Bottom-Left  | -X        | (-S, 0, 0)  | (-S/2, -S/2, 0)          | (0, 0, +1)    | PI/2  |
| Bottom-Right | +Z        | (0, 0, S)   | (0, -S/2, +S/2)          | (+1, 0, 0)    | PI/2  |
| Top-Left     | -Z        | (0, 0, -S)  | (0, -S/2, -S/2)          | (-1, 0, 0)    | PI/2  |

The rotation axis can be derived: `cross(direction, downVector)` where `downVector = (0, -1, 0)`.

The pivot offset is always: half a cube-width toward the roll direction on the ground axis, plus half a cube-width downward (to the bottom edge).

## CSS Direction Data (pre-isometric flat plane)

The isometric rotation (`rotateX(-35.264deg) rotateY(45deg)`) is applied to the perspective container. The cube rolls in the container's local coordinate system (flat plane before rotation):

| Direction    | `transform-origin`  | Rotation        | Translation    |
| ------------ | ------------------- | --------------- | -------------- |
| Top-Right    | bottom-right edge   | rotateZ(-90deg) | translateX(+S) |
| Bottom-Left  | bottom-left edge    | rotateZ(+90deg) | translateX(-S) |
| Bottom-Right | bottom-front edge   | rotateX(+90deg) | translateY(-S) |
| Top-Left     | bottom-back edge    | rotateX(-90deg) | translateY(+S) |

### CSS `transform-origin` Values (3D)

For a cube of side `S` centred at its own origin, the bottom edge origins in 3D space:

| Direction    | `transform-origin`       |
| ------------ | ------------------------ |
| Top-Right    | `S S/2 0`                |
| Bottom-Left  | `0 S/2 0`                |
| Bottom-Right | `S/2 S/2 calc(S / -2)`   |
| Top-Left     | `S/2 S/2 calc(S / 2)`    |

The `transform-origin` must be a **3D value (three components)**. A 2D value places the pivot at Z=0, which is wrong for Bottom-Right and Top-Left (those need a non-zero Z component). The exact values depend on the cube construction approach (specifically how faces are positioned and whether a `translateZ` centering transform is applied to the cube container). The principle is always the same: origin goes to the bottom edge in the roll direction, in 3D space.
