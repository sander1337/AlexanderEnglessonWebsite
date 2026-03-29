# Custom Particle System — Technical Portfolio

## Overview

Built a fully custom GPU-friendly particle system from scratch inside a proprietary C++/DirectX11 game engine (TGA Engine). The system is entirely data-driven, editor-integrated, and supports real-time preview — no third-party particle middleware used.

---

## Curve-Based Over-Lifetime System

Replaced a legacy scalar min/max approach with a full **Catmull-Rom spline curve editor** for every particle property. Every aspect of a particle's life is now authored visually:

- **Size over lifetime** — smooth scaling driven by a bezier-like spline
- **Velocity over lifetime** — separate X/Y/Z curves, allowing complex motion paths
- **Color over lifetime** — per-channel R/G/B/A curves with a live gradient preview bar
- **Rotation over lifetime** — angular velocity driven by curve data

The curve editor supports:
- Right-click to add keyframes anywhere on the canvas
- Shift + right-click to remove keyframes
- Drag to reposition keys with neighbour-pushing collision
- Live time axis showing actual seconds based on particle lifetime
- Adjustable Y-axis scale per curve

---

## Physically Correct Gravity

Implemented proper gravity accumulation as a separate velocity integrator rather than baking it into curve keyframes. Gravity scales independently of the velocity curves, accumulates per-frame like real physics, and correctly interacts with the curve-driven movement system.

---

## Spawn Shape System

Seven fully implemented spawn shapes, each with correct spatial distribution:

| Shape | Notes |
|---|---|
| Point | Single origin |
| Sphere | Uniform volume distribution using cube-root for even density |
| Hollow Sphere | Surface-only spawning |
| Cone | Apex spawn, outward direction baked per-particle at spawn |
| Box | Uniform 3D volume |
| Circle | 2D disc with surface/volume toggle |
| Edge | Linear distribution |

All shapes support a **direction spread** parameter for randomising outward direction within a cone angle. Cone shape correctly computes spawn direction from apex to base ring using proper vector projection. All shapes have **real-time debug visualisation** drawn with the engine's debug drawer.

---

## Billboard Sprite Rendering

Particles render as **billboard 3D sprites** — always facing the camera regardless of viewing angle. Integrated with the engine's existing sprite rendering pipeline, supporting:

- Alpha blending
- No face culling
- Depth read-only for correct transparency sorting
- Back-to-front sort by camera distance each frame

---

## VFXInstance Runtime

Each particle is backed by a `VFXInstance` component that evaluates all curves every frame and writes results directly into the `Sprite3DInstanceData`:

- `mySize` updated directly — bypasses transform matrix scale which doesn't affect sprites
- `myColor` updated per channel from R/G/B/A curves
- Position integrated from velocity curves each frame
- Gravity accumulated as a proper separate velocity

The separation between sprite instance data and transform matrix was a non-obvious engine-specific issue that required understanding the rendering pipeline deeply to solve correctly.

---

## Burst Emission System

Full burst emission support with:
- Multiple bursts per emitter, each with configurable time, count, cycle count and interval
- Infinite cycle mode (`cycles = -1`)
- Independent of continuous spawn rate — bursts and rate emission run in parallel
- Editor UI for adding/removing bursts at runtime

Fixed a subtle bug where burst timing only advanced when `duration > 0`, causing bursts to never fire on infinite-duration emitters.

---

## Editor Integration

The entire particle system is **live-editable in the editor** with real-time preview in both the Object Definition document and the Scene document:

- Drag-and-drop texture assignment
- All curve editors update particles immediately
- Color tab shows a **live gradient bar** showing the full color lifecycle at a glance
- Shape debug wireframe rendered in editor viewport
- Warmup system pre-simulates particles on spawn so effects appear already running when placed in a scene
- Per-emitter preview in scene view with correct world-space positioning

The ImGui editor uses a `CopyOnWriteWrapper` pattern — data is only copied for mutation when an actual edit occurs, keeping the editor efficient.

---

## JSON Serialization

All particle data serializes cleanly to/from JSON with full forward compatibility — missing fields fall back to sensible defaults, so old saved files still load correctly after struct changes.

---

## What Makes This Hard

- Sprites don't use transform matrix scale — size must be written directly into instance data each frame, which requires knowing the rendering pipeline internals
- Velocity curves are sampled as instantaneous velocity, not accumulated — gravity had to be handled as a separate integrator to avoid the curve interpolation causing exponential acceleration
- The `CopyOnWriteWrapper` editor pattern means ImGui editors must call `makeEditable()` before passing references to lambdas, or edits silently write to discarded copies
- Burst timing must be tracked independently of duration so infinite emitters still fire bursts correctly
- Billboard sprites require back-to-front sorting per frame to composite correctly with other transparent objects

---

*Built in one session in a custom C++/DirectX11 engine with ImGui editor tooling.*
