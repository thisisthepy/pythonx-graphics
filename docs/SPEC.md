# Specification

What `pythonx-game` will do: the behavioural contract. Every item stays inside `docs/INTENT.md`.
Anything INTENT does not cover is under "Outside intent: needs a decision".

| Status | Meaning |
|---|---|
| `implemented` | Behaviour exists in this repository and a test that was read for this document passes. |
| `partial` | Some of it exists, or its only evidence is outside this repository. |
| `planned` | Required by INTENT; nothing delivers it yet. |

Every entry below is `planned`. The maintainer ranks the pythonx packages after torchnative,
python-multiplatform, pypackpack/toolchain and pythonx-compose (2026-10-04): the spec is written now,
implementation comes in its turn. Issue markers read `Issue: to be opened`. A line starting
`needs binder:` is a request to python-multiplatform; `needs kotlin layer:` is a request to wherever
the Kotlin layer lives (INTENT open question 1). Both are collected in section 11.

Examples install with uv, ppp or tcl once the package is published:

```bash
uv add --prerelease allow pythonx-game
ppp core add "pythonx-game==0.1.0a1"
tcl install pythonx-game
```

## 0. The shape in one example

```python
from pythonx.compose.runtime import Composable, app, state
from pythonx.compose.material3 import Text
from pythonx.game import Scene3D, Camera, Sun, Model, Spin, Box, Material

angle_speed = state(30.0)              # degrees per second, shared by HUD and scene

@app
@Composable
def Game():
    Scene3D(
        camera=Camera(position=(0, 1.5, 4), look_at=(0, 0, 0)),
        content=lambda: {
            Sun(intensity=100_000, direction=(-1, -1, -1)),
            Model("models/chair.glb", name="chair", tags={"furniture"},
                  behaviours=[Spin(axis=(0, 1, 0), speed=angle_speed)]),
            Box(size=(4, 0.1, 4), name="floor", material=Material(color=0xFF808080), static=True),
        },
    )
    Text(f"speed {angle_speed.value:.0f} deg/s")   # 2D Compose HUD over the 3D view
```

`Spin` is declared once; Filament rotates the chair every frame natively. Python runs again only when
`angle_speed` changes.

---

## 1. Declaration model

### G1.1 One model for 2D and 3D: `planned`

A 3D scene is a Compose composition whose nodes are Filament entities (a Compose `Applier` over the
Filament scene). Python declares it with the same functions, `state(...)` and `@app` redeclaration as
pythonx-compose UI. `Scene3D(...)` is a composable that occupies a rectangle of the 2D UI and holds a
3D composition inside it. needs kotlin layer: the `Applier`, `Scene3D`, and node composables.
Issue: to be opened.

### G1.2 Nodes: `planned`

Node composables: `Camera`, `Sun`, `PointLight`, `SpotLight`, `Environment` (IBL and skybox), `Model`
(glTF/GLB through Filament's gltfio), primitives (`Box`, `Sphere`, `Plane`, `Cylinder`), `Group`
(transform parent), `Instances` (G9.2), `Text3D` (later). Every node accepts `position`, `rotation`,
`scale`, `name`, `tags`, `visible`, `behaviours`. Arguments are snake_case. Issue: to be opened.

### G1.3 Redeclaration replaces the scene, state survives: `planned`

Redeclaring a root (a notebook cell, a hot reload) recomposes: nodes with the same key keep their
Filament entity, new ones are created, missing ones are destroyed. A node's key is its `name` if
given, else its position in the composition. Loaded assets are cached by path, so redeclaring does not
reload a model. Issue: to be opened.

### G1.4 State drives both sides: `planned`

A `state(...)` read in a 3D node's arguments recomposes only that node when it changes. A state read
inside a behaviour's parameters (as `Spin(speed=angle_speed)`) updates the native behaviour without
recomposition. Issue: to be opened.

### G1.5 2D is Compose: `planned`

2D games and HUDs use pythonx-compose's surface and Compose's `Canvas` drawing (Skia). This package
adds 2D game helpers that Compose lacks: a sprite node, sprite sheets and tile maps drawn in one
canvas pass, and the same behaviour and agent model as 3D (G2, G8). Issue: to be opened.

---

## 2. Behaviour declared, not looped

The frame loop is native. Python declares what should happen; it is not called every frame.

### G2.1 Behaviours are data: `planned`

`behaviours=[...]` attaches native behaviours to a node: `Tween(property, to, duration, easing)`,
`Spring(property, target, stiffness, damping)`, `Keyframes(property, frames)`, `FollowPath(points)`,
`Spin(axis, speed)`, `LookAt(target)`, `Orbit(center, radius, speed)`. Each runs on the native frame
clock (Compose `withFrameNanos`). needs kotlin layer: the behaviour runtime. Issue: to be opened.

### G2.2 Events call Python, frames do not: `planned`

`on_click`, `on_hover`, `on_collision(other)`, `on_enter(trigger)`, `on_finished` (of a behaviour) and
`on_tick(every=seconds)` call Python. There is no per-frame Python callback by default. `on_frame` exists
for prototyping, is documented as costly, and a measurement test (G9.1) reports its cost.
Issue: to be opened.

### G2.3 Compiled per-frame logic: `planned`

Logic that must run every frame is written as a function over arrays and compiled with TypedPython; it
runs natively and, on free-threaded CPython, in parallel across entities when the compiler proves the
iterations independent (python-multiplatform #151). Issue: to be opened.

---

## 3. Rendering

### G3.1 Filament for 3D: `planned`

PBR materials, image-based lighting, shadows, post-processing (bloom, tone mapping, anti-aliasing) as
Filament provides them, configured by `Scene3D(quality=...)` presets and explicit options.
Issue: to be opened.

### G3.2 Backends per platform: `planned`

| Platform | Filament backend | Host view |
|---|---|---|
| Android | Vulkan or OpenGL ES | `SurfaceView`/`TextureView` inside Compose |
| iOS | Metal | an `MTKView`-backed `UIView` inside Compose |
| Desktop | Metal (macOS), Vulkan or OpenGL (Linux, Windows) | a native surface or an offscreen target composited into Compose |
| wasm | WebGL | INTENT open question 4 |

Feasibility and what is unverified per platform: `docs/research.md` §3. Issue: to be opened.

### G3.3 2D over 3D and 3D in 2D: `planned`

Compose content can sit over a `Scene3D` (HUD), and several `Scene3D`s can sit in one UI. Order and
hit-testing follow Compose. Issue: to be opened.

---

## 4. Assets

### G4.1 Loading: `planned`

`Model(path_or_url)` loads glTF/GLB asynchronously through pythonx-concurrent; a placeholder (bounds
box) is shown until loaded. Textures (KTX2, PNG, JPEG) and environment maps likewise. Assets are
cached and reference-counted across redeclarations. Issue: to be opened.

### G4.2 Generative asset sources: `planned`

An asset argument may be a provider instead of a path: `Model(generate("a wooden chair"))`,
`Material(texture=generate_texture("worn brick"))`. A provider is an async function the app supplies
(local model through torchnative, or a remote service); this package defines the interface, the
placeholder and the cache, and records provenance (provider, prompt, seed) in the scene (G7.1).
No provider ships with the package. Issue: to be opened.

---

## 5. Input

### G5.1 Pointer, touch, keyboard, gamepad: `planned`

Pointer and touch through Compose; ray picking of 3D nodes through Filament's picking (G7.3) feeds
`on_click`/`on_hover`. Keyboard and gamepad as state (`input.keys`, `input.gamepad(0)`), readable in
behaviours without Python per frame. Issue: to be opened.

---

## 6. Physics

### G6.1 Declared bodies: `planned`

`body=RigidBody(mass, friction, restitution)`, `static=True`, `collider=Auto|Box|Sphere|Mesh`,
`trigger=True` on a node. Simulation runs natively at a fixed step; collisions and triggers raise the
events of G2.2. The engine is INTENT open question 3. Issue: to be opened.

---

## 7. The scene as data (the agent's view)

The innovation of this package is that every scene is also a **document**: typed, addressable,
serialisable, diffable. An agent works on the document; the renderer shows it.

### G7.1 Scene IR: `planned`

Every node has a stable id, a kind, its declared arguments, its resolved transform and bounds, its
`name`, `tags` and asset provenance. `scene.snapshot()` returns this as a plain Python structure and
as JSON with a published JSON Schema. `Scene.from_snapshot(...)` declares the same scene back
(round trip: snapshot, declare, snapshot is identical). Issue: to be opened.

### G7.2 Description and query: `planned`

`scene.describe()` returns a compact text and structured summary meant for a language model: counts,
named entities, hierarchy, and **spatial relations computed from bounds** (`on_top_of`, `left_of`,
`inside`, `near`, `facing`) relative to the camera or the world. `scene.query(kind=..., tags=...,
where=lambda n: ...)` selects nodes; `scene.relations(a, b)` answers one pair. Issue: to be opened.

### G7.3 Grounding in pixels: `planned`

`scene.capture()` returns, for one frame: the colour image, a per-pixel **entity id buffer**, a depth
buffer and the camera matrices. `scene.pick(x, y)` returns the node under a pixel; `scene.mask(node)`
returns its pixels. With these, a vision model's answer ("the red chair is at 412, 300") becomes a node
id, and an agent's edit can be checked in pixels. needs kotlin layer: an id pass (Filament's picking
renders entity ids). Issue: to be opened.

### G7.4 Edits are transactions: `planned`

`with scene.edit(author="agent") as tx:` collects changes (`tx.move(node, to=...)`, `tx.set(node,
material=...)`, `tx.add(...)`, `tx.remove(...)`). On exit the transaction is **validated** (argument
types against the node schema, user constraints G7.5), then applied atomically as one recomposition,
recorded with its author, and undoable (`scene.undo()`). `tx.preview()` renders the result without
applying it. A failed validation applies nothing and reports why. Issue: to be opened.

### G7.5 Constraints and permissions: `planned`

The app declares rules the agent cannot break: `scene.constrain(no_overlap(tags={"furniture"}),
inside(bounds=room), above(floor))`, and which nodes and properties an agent may change
(`editable=...` on a node, default not editable for nodes marked `locked`). Violations reject the
transaction. Issue: to be opened.

### G7.6 Declarative source stays the truth: `planned`

An agent edit to a scene that was declared in code produces a **patch to the declaration** as well as
the live change: `tx.as_source_patch()` returns the change as Python source edits against the
declaring module (by node name), so an agent's change can be committed, reviewed and replayed.
Where the declaration is computed (a loop), the patch says so and offers a state change instead.
Issue: to be opened.

---

## 8. Agents as first-class users

### G8.1 Tool schemas generated from the components: `planned`

`pythonx.game.tools()` returns tool definitions (JSON Schema) for `describe`, `query`, `capture`,
`pick`, `edit` and every node kind's arguments, generated from the node signatures (the binder's
`describe()` metadata for Kotlin-backed arguments). An app hands them to any agent framework. Whether
an MCP server adapter ships here is INTENT open question 5. Issue: to be opened.

### G8.2 Headless and deterministic: `planned`

On desktop a scene renders offscreen with no window (`Scene3D.headless(width, height)`), with a fixed
clock (`clock=FixedClock(fps=60)`), a fixed physics step and a seed. The same declaration, inputs and
seed render the same pixels on the same machine, so an agent or a CI job can compare frames with a
reference (pixel and per-entity mask diffs, as python-multiplatform's render tests already do).
Issue: to be opened.

### G8.3 Record and replay: `planned`

`scene.record()` logs declarations, state changes, inputs and transactions with timestamps;
`replay(log)` reproduces the session headless. An agent can bisect a regression or evaluate many
candidate edits offline. Issue: to be opened.

### G8.4 Observation stream: `planned`

`scene.observe(every=seconds or on=events)` yields snapshots or diffs (G7.1) asynchronously, so an
agent loop watches a running game without polling and without per-frame Python. Issue: to be opened.

---

## 9. Performance

### G9.1 Measurement tests: `planned`

Measured and published per platform: frame time of an idle scene, cost of a state change that
recomposes one node, cost of a behaviour parameter update (no recomposition), cost of `on_frame`
per entity, instancing cost per 10,000 instances, `capture()` cost. These guard AGENTS.md rule 12.7.
Issue: to be opened.

### G9.2 Instancing from arrays: `planned`

`Instances(mesh, transforms=array)` draws N copies from one buffer-protocol array (NumPy,
`array.array`, memoryview) with no per-instance Python object. Updating the array updates the GPU
buffer once per frame. needs binder: zero-copy buffer-protocol arguments (shared with TypedPython
#149 in python-multiplatform). Issue: to be opened.

### G9.3 No Python on the frame path: `planned`

With no `on_frame` and no state change, a running scene performs zero Python calls per frame. Test:
the `sys.monitoring` counter used by python-multiplatform's notebook E2E reads 0 over 120 frames.
Issue: to be opened.

---

## 10. Testing strategy

- **Pixel tests against Kotlin controls**, as python-multiplatform's render tests do: a Python
  declaration and the same scene written in Kotlin render the same image (within a stated tolerance
  for GPU differences).
- **Round-trip tests** for G7.1 and G7.6.
- **Agent scenario tests**: scripted agent tasks (find, move, validate, undo) run headless (G8.2).
- **Red first, with a cause** (AGENTS.md §5): each test fails before its feature exists, and the
  failure message names the missing feature.

## 11. Requests to other repositories

needs binder:
1. Zero-copy buffer-protocol arguments for array parameters (G9.2).
2. Python callables invoked from native events on the UI thread without per-call setup cost (G2.2).

needs kotlin layer:
1. Compose `Applier` over Filament entities, `Scene3D` and node composables (G1.1, G1.2).
2. The native behaviour runtime on the frame clock (G2.1).
3. Entity-id pass, depth readback and offscreen rendering (G7.3, G8.2).
4. Physics integration (G6.1).
5. Platform host views (G3.2).

## 12. Outside intent: needs a decision

1. Networking and multiplayer.
2. Audio (positional audio would fit G1.2's node model).
3. A visual editor beyond notebooks and agents.
4. Custom GPU compute and custom shaders beyond Filament's material system.
