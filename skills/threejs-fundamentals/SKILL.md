---
name: threejs-fundamentals
description: Three.js scene setup, cameras, renderer, Object3D hierarchy, coordinate systems. Use when setting up 3D scenes, creating cameras, configuring renderers, managing object hierarchies, or working with transforms.
user-invocable: true
argument-hint: "[topic] e.g. camera, Object3D, attach, pivot, quaternion"
---

# Three.js Fundamentals

## Quick Start

```javascript
import * as THREE from "three";

// Create scene, camera, renderer
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000,
);
const renderer = new THREE.WebGLRenderer({ antialias: true });

renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
document.body.appendChild(renderer.domElement);

// Add a mesh
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// Add light
scene.add(new THREE.AmbientLight(0xffffff, 0.5));
const dirLight = new THREE.DirectionalLight(0xffffff, 1);
dirLight.position.set(5, 5, 5);
scene.add(dirLight);

camera.position.z = 5;

// Animation loop — Three.js does NOT render automatically
function animate() {
  requestAnimationFrame(animate);
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  renderer.render(scene, camera);
}
animate();

// Handle resize
window.addEventListener("resize", () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

## Core Classes

### Scene

Container for all 3D objects, lights, and cameras.

```javascript
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x000000);
```

### Cameras

**PerspectiveCamera** — Most common, simulates human eye.

```javascript
// PerspectiveCamera(fov, aspect, near, far)
const camera = new THREE.PerspectiveCamera(
  75, // Field of view (degrees)
  window.innerWidth / window.innerHeight, // Aspect ratio
  0.1, // Near clipping plane
  1000, // Far clipping plane
);

camera.position.set(0, 5, 10);
camera.lookAt(0, 0, 0);
camera.updateProjectionMatrix(); // Call after changing fov, aspect, near, far
```

**OrthographicCamera** — No perspective distortion, good for 2D/isometric. Objects stay the same size regardless of distance from camera.

```javascript
// OrthographicCamera(left, right, top, bottom, near, far)
// These define a rectangular viewing box (frustum) in world units
const aspect = window.innerWidth / window.innerHeight;
const frustumSize = 10;
const camera = new THREE.OrthographicCamera(
  (frustumSize * aspect) / -2, // left
  (frustumSize * aspect) / 2, // right
  frustumSize / 2, // top
  frustumSize / -2, // bottom
  0.1, // near
  1000, // far
);
```

**Isometric setup:** Position the camera along a diagonal and use `lookAt` to point at the origin. The `zoom` property adjusts how many world units are visible. In an isometric view, world +X appears as a diagonal going up-right on screen, and +Z goes down-right.

```javascript
camera.position.set(10, 10, 10);
camera.lookAt(0, 0, 0);
camera.zoom = 50;
camera.updateProjectionMatrix();
```

### WebGLRenderer

```javascript
const renderer = new THREE.WebGLRenderer({
  canvas: document.querySelector("#canvas"), // Optional existing canvas
  antialias: true, // Smooth edges
  alpha: true, // Transparent background
});

renderer.setSize(width, height);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setClearColor(0x000000, 1);

// Shadows
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;

renderer.render(scene, camera);
```

### Object3D

Base class for all 3D objects. Mesh, Group, Light, Camera all extend Object3D.

#### Scene Graph and Transform Hierarchy

Every Object3D has a local transform (position, rotation, scale) relative to its parent. Three.js composes these down the hierarchy when rendering:

```
worldMatrix = parent.worldMatrix × child.localMatrix
```

This means:
- Moving a parent moves all its children in world space
- A child's `.position` is in **parent-local** coordinates, not world coordinates
- Reparenting changes which coordinate space the local transform is relative to

```javascript
const obj = new THREE.Object3D();

// Transform (all in parent-local space)
obj.position.set(x, y, z);
obj.rotation.set(x, y, z); // Euler angles (radians)
obj.quaternion.set(x, y, z, w); // Quaternion rotation
obj.scale.set(x, y, z);

// World-space queries
obj.getWorldPosition(targetVector);
obj.getWorldQuaternion(targetQuaternion);
obj.getWorldDirection(targetVector);
```

#### Hierarchy — add() vs attach()

```javascript
// add() — keeps child's LOCAL transform unchanged
// If the new parent is at a different position, the child visually jumps
parent.add(child);

// attach() — recomputes child's local transform to preserve WORLD position
// The child appears to stay in place
parent.attach(child);

parent.remove(child);
parent.children; // Array of children
child.parent; // Reference to parent
```

**When to use which:**
- `add()` — placing a new object into a parent (it has no meaningful world position yet)
- `attach()` — reparenting an existing object that should stay visually in place (e.g., the pivot pattern below)

`attach()` depends on up-to-date world matrices. If you just created or repositioned the parent in the same frame, call `parent.updateMatrixWorld()` before `attach()`.

```javascript
// Example: child at world (5,0,0), new parent at world (3,0,0)
parent.add(child); // child.position stays (5,0,0) local → world (8,0,0) — JUMPS
parent.attach(child); // child.position becomes (2,0,0) local → world (5,0,0) — STAYS
```

#### Other Object3D Features

```javascript
obj.visible = false;

// Layers (for selective rendering/raycasting)
obj.layers.set(1);
obj.layers.enable(2);

// Traverse hierarchy
obj.traverse((child) => {
  if (child.isMesh) child.material.color.set(0xff0000);
});

// Force-update world matrix (needed before attach() if parent just moved)
obj.updateMatrixWorld(true);
```

### Group

Empty container for organizing objects.

```javascript
const group = new THREE.Group();
group.add(mesh1);
group.add(mesh2);
scene.add(group);

// Transform entire group
group.position.x = 5;
group.rotation.y = Math.PI / 4;
```

### Mesh

Combines geometry and material.

```javascript
const mesh = new THREE.Mesh(geometry, material);

// Per-face materials (array of 6 for BoxGeometry — one per face)
const mesh = new THREE.Mesh(geometry, [mat0, mat1, mat2, mat3, mat4, mat5]);

mesh.castShadow = true;
mesh.receiveShadow = true;
mesh.frustumCulled = true; // Default: skip rendering if outside camera view
mesh.renderOrder = 10; // Higher = rendered later
```

## Coordinate System

Three.js uses a **right-handed, Y-up** coordinate system:

- **+X** points right
- **+Y** points up
- **+Z** points toward the viewer (out of screen)

The ground plane is **XZ** (not XY).

**CSS contrast:** In CSS, +Y points *down* the screen and there is no Z depth by default. In Three.js, +Y points *up* and the ground plane is XZ.

```javascript
const axesHelper = new THREE.AxesHelper(5);
scene.add(axesHelper); // Red=X, Green=Y, Blue=Z
```

## Math Essentials

Most commonly needed operations. For the full API (all Vector3, Matrix4, Quaternion, Euler, Color methods), see `references/math-utilities.md`.

```javascript
// Vector3
const v = new THREE.Vector3(x, y, z);
v.set(x, y, z);
v.copy(other);
v.clone();
v.add(v2);
v.sub(v2);
v.multiplyScalar(2);
v.normalize();
v.length();
v.distanceTo(v2);
v.lerp(target, alpha);

// Angle conversions
THREE.MathUtils.degToRad(degrees);
THREE.MathUtils.radToDeg(radians);

// Useful utilities
THREE.MathUtils.clamp(value, min, max);
THREE.MathUtils.lerp(start, end, alpha);
```

## Common Patterns

### The Render Loop

Three.js does not render automatically — you must call `renderer.render(scene, camera)` whenever the scene should update. The standard pattern uses `requestAnimationFrame`:

```javascript
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
animate();
```

When using GSAP with Three.js, GSAP tweens property values but does not trigger renders. Options:
- Run a continuous `requestAnimationFrame` loop (simplest)
- Add `renderer.render(scene, camera)` in GSAP's `onUpdate` callback
- Use `gsap.ticker.add(() => renderer.render(scene, camera))`

### Pivot Pattern

Three.js has no equivalent to CSS `transform-origin`. To rotate an object around an arbitrary point, use an empty Object3D as a temporary pivot:

1. Create an empty Object3D positioned at the desired pivot point
2. `attach()` the object to the pivot (preserves world position)
3. Rotate the pivot — the object orbits around it
4. `attach()` the object back to the scene
5. Remove the pivot

```javascript
const pivot = new THREE.Object3D();
pivot.position.set(edgeX, edgeY, edgeZ);
scene.add(pivot);

pivot.attach(cube); // Cube stays in place visually

// Animate the pivot rotation (e.g., with GSAP)
// gsap.to(pivot.rotation, { z: -Math.PI / 2, onUpdate: render })

// After animation completes:
scene.attach(cube); // Reparent cube back to scene
scene.remove(pivot);
```

For workshop-specific direction mappings and the full rolling cycle, see the `isometric-rolling-cube` skill.

### Euler Angles and Rotation Drift

Three.js stores rotation as Euler angles (`obj.rotation.x/y/z` in radians) with a default `XYZ` order. When combining 90° rotations across multiple axes (common in rolling animations), floating-point imprecision accumulates — after 3-4 rolls the cube visibly drifts.

**Fix:** After each animation completes, snap to clean multiples of π/2:

```javascript
function snapRotation(obj) {
  const HALF_PI = Math.PI / 2;
  obj.rotation.x = Math.round(obj.rotation.x / HALF_PI) * HALF_PI;
  obj.rotation.y = Math.round(obj.rotation.y / HALF_PI) * HALF_PI;
  obj.rotation.z = Math.round(obj.rotation.z / HALF_PI) * HALF_PI;
}
```

### Proper Cleanup

```javascript
function dispose() {
  mesh.geometry.dispose();

  if (Array.isArray(mesh.material)) {
    mesh.material.forEach((m) => m.dispose());
  } else {
    mesh.material.dispose();
  }

  texture.dispose();
  scene.remove(mesh);
  renderer.dispose();
}
```

### Clock for Animation

```javascript
const clock = new THREE.Clock();

function animate() {
  const delta = clock.getDelta(); // Seconds since last frame
  const elapsed = clock.getElapsedTime(); // Total seconds

  mesh.rotation.y += delta * 0.5; // Framerate-independent speed

  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
```

### Responsive Canvas

```javascript
// Perspective camera
function onResize() {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

// Orthographic camera
function onResize() {
  const aspect = window.innerWidth / window.innerHeight;
  camera.left = (frustumSize * aspect) / -2;
  camera.right = (frustumSize * aspect) / 2;
  camera.top = frustumSize / 2;
  camera.bottom = frustumSize / -2;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
}

window.addEventListener("resize", onResize);
```

## Reference Files

- **`references/math-utilities.md`** — Full API for Vector3, Matrix4, Quaternion, Euler, Color, and MathUtils. Load when participants need specific math operations beyond the essentials above.

## See Also

- `threejs-materials` — Material types and properties
- `threejs-animation` — Keyframe animation, skeletal animation, morph targets
- `isometric-rolling-cube` — Rolling animation technique using the pivot pattern
