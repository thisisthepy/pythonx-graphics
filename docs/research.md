# Research

Evidence behind `docs/SPEC.md`. Each claim says where it comes from: a cited source, or
`(inferred)` when it is reasoning, or `(unverified)` when it still has to be checked. Nothing here was
measured in this repository yet.

## 1. Why Filament, not wgpu

| | wgpu / WebGPU | Filament |
|---|---|---|
| Layer | GPU API (buffers, textures, pipelines, shaders) | Scene-level renderer (entities, cameras, lights, PBR materials, glTF) |
| GPU-side speed | Shaders compile to the platform's native form (MSL, SPIR-V, HLSL) through naga; GPU work runs at native speed (inferred) | Native C++ renderer on Vulkan, Metal, OpenGL ES (Filament README) |
| CPU-side cost | Per-call validation and resource tracking cost CPU, most visible with many draw calls (inferred from WebGPU's validation model) | A renderer built for mobile; batching and culling inside the engine (inferred) |
| What it leaves to us | The whole scene layer (scene graph, camera, materials, lights, loaders), the part three.js provides | A declarative layer and the agent layer |
| Kotlin access | none official | Android Java/Kotlin API is official (filament-android); desktop Java bindings exist in the Filament source tree (unverified for current releases) |
| Licence | MIT / Apache-2.0 | Apache-2.0 |

Conclusion recorded in python-multiplatform `docs/design/ecosystem.md` §7: Filament for 3D, Compose
(Skia) for 2D, wgpu only if custom GPU compute is ever needed.

## 2. Prior art

- **SceneView** (Android, Apache-2.0): Filament inside Jetpack Compose with composable nodes. Shows that
  a Compose `Applier` over Filament entities is practical on Android. Android only; no iOS or desktop.
  (unverified: current API and maintenance state)
- **three.js / react-three-fiber**: a declarative scene (JSX) over a retained scene graph, with
  reconciliation. The closest model to G1; the agent layer of §7 and §8 has no counterpart there.
- **Compose runtime**: tree-agnostic; any node type can be managed through a custom `Applier`
  (Compose runtime documentation). This is what lets 3D nodes share pythonx-compose's model.
- **Game engines' ECS** (Bevy, Unity DOTS): data-oriented per-frame systems. G2.3 takes the idea
  (per-frame logic over arrays, compiled) without exposing an ECS as the primary API.

## 3. Per-platform feasibility

| Capability | Android | iOS | Desktop | wasm |
|---|---|---|---|---|
| Filament renderer | official (Vulkan/GLES) | official C++ (Metal) | official C++ (Metal/Vulkan/GL) | official WebGL build |
| Kotlin access to Filament | official Java/Kotlin API | none: Filament's API is C++, and Kotlin/Native cinterop needs a C API, so a C shim is required (inferred) | Java bindings (unverified for current releases); else JNI shim | Kotlin/Wasm to Filament JS (unverified) |
| Filament view inside Compose | `AndroidView` with `SurfaceView`/`TextureView` | `UIKitView` with an `MTKView` (inferred) | native surface or offscreen + texture copy into Compose (inferred; cost to measure) | canvas element (unverified) |
| Entity-id buffer (picking) | Filament `View.pick` (Filament API) | same | same | same (unverified) |
| Headless rendering | not a goal | not a goal | offscreen swap chain (Filament supports headless swap chains, unverified per backend) | no |
| Python on the frame path | not needed by design (G2) | same | same | same |

## 4. What is hard

1. **iOS needs a C shim** around Filament's C++ API before Kotlin/Native can call it. Size unknown.
2. **Desktop composition**: drawing Filament into a Compose Desktop window either needs a native
   child surface (z-order with Compose content above it is hard) or an offscreen render copied into a
   Skia image each frame (simple, costs a copy). To measure.
3. **Determinism across GPUs** is not achievable; G8.2 promises same pixels on the same machine only,
   and per-entity masks (id buffer) are the robust comparison across machines.
4. **Source patches (G7.6)** are exact only for declarations with literal arguments and names; computed
   declarations get a state-change suggestion instead.
5. **Physics determinism** depends on the engine chosen (INTENT open question 3); Jolt advertises
   deterministic simulation (unverified).
