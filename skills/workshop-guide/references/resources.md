# Workshop External Resources

Key reference materials for the LLL Animation Workshop, organised by phase and topic.

## Phase 1 — CSS + GSAP

### CSS 3D Transforms

- **[Intro to CSS 3D Transforms — Cube (David DeSandro)](https://3dtransforms.desandro.com/cube)** — Primary reference for Phase 1 cube construction. Covers the 6-div technique, face positioning with rotate + translateZ, perspective, and show-face transitions.
- **[Intro to CSS 3D Transforms — Perspective (DeSandro)](https://3dtransforms.desandro.com/perspective)** — Supplementary reading on how CSS perspective works. Useful for understanding why isometric projection requires removing perspective.
- **[Intro to CSS 3D Transforms — 3D Transform Functions (DeSandro)](https://3dtransforms.desandro.com/3d-transform-functions)** — Reference for rotateX, rotateY, rotateZ, translateZ, and transform function ordering.

### Isometric Projection

- **[Isometric Projection — Wikipedia](https://en.wikipedia.org/wiki/Isometric_projection)** — Reference for understanding the math behind isometric rotation angles. The key angle is `atan(1 / sqrt(2))` = 35.264 degrees.

## Phase 2 — Three.js + GSAP

### Three.js

- **[Three.js Documentation](https://threejs.org/docs/)** — Official API docs. Key pages for the workshop:
  - [BoxGeometry](https://threejs.org/docs/#api/en/geometries/BoxGeometry) — Cube shape
  - [MeshBasicMaterial](https://threejs.org/docs/#api/en/materials/MeshBasicMaterial) — Simple material, no lighting needed
  - [OrthographicCamera](https://threejs.org/docs/#api/en/cameras/OrthographicCamera) — The camera type used for isometric projection
  - [Object3D](https://threejs.org/docs/#api/en/core/Object3D) — Base class, crucial for the pivot technique (`.attach()`, `.add()`, `.position`, `.rotation`)

## Both Phases

### GSAP

- **[GSAP Documentation](https://gsap.com/docs/v3/)** — Animation library used in both phases. Key pages:
  - [gsap.to()](https://gsap.com/docs/v3/GSAP/gsap.to()) — Core tween method
  - [gsap.timeline()](https://gsap.com/docs/v3/GSAP/Timeline/) — Composing roll animations
  - [gsap.set()](https://gsap.com/docs/v3/GSAP/gsap.set()) — Immediate property setting (used for grid snapping)
  - [gsap.delayedCall()](https://gsap.com/docs/v3/GSAP/gsap.delayedCall()) — Scheduling pauses between rolls
  - [Eases](https://gsap.com/docs/v3/Eases/) — `power1.out`, `power2.inOut` for roll feel

## Workshop Spec

- **[LLL Animation Spec (Notion)](https://www.notion.so/LLL-Animation-Spec-30dced2a7e13804eb784d06ef1b9e6a9)** — The canonical workshop specification with all task requirements, hints, and acceptance criteria.

## Facilitator Guides

These documents were added by the workshop facilitator and provide additional architectural context beyond the spec:

- **[Crash Course: 3D cube animations (CSS vs Three.js)](https://www.notion.so/labrys/Crash-Course-3D-cube-animations-CSS-vs-Three-js-313ced2a7e13801eb0f8e5ed0057e568)** — Explains the 3-container CSS architecture (position/perspective/modeled-cube separation), the `THREE.Group` isometric container, and the `yoyo` jump effect. Good conceptual overview of how both phases work architecturally.

- **[3D Cube CSS vs Three.js — Performance Comparison](https://www.notion.so/labrys/3D-Cube-CSS-vs-Three-js-30bced2a7e138077b50ce60c97faddd2)** — Explains why CSS is fine for a single cube but Three.js/WebGL is needed for 1,000+ cubes. Useful context for Phase 2 motivation.
