# Intent

What `pythonx-graphics` is for. This file is the boundary: `docs/SPEC.md` may not go beyond it. Anything
the maintainer has not said is under "Open questions" until confirmed.

## Sources

The maintainer's words, in the order they were given (2026-10-04):

> "게임 개발용 라이브러리 ... three.js 대체"

> "wgpu 말고 더 나은 추상화 방식은 없나? 아무래도 속도가 조금 걱정인데 추상화 품질이 어떤데?"

> "3D는 Filament를 쓰고 2D는 Compose로 하는거 좋아. 그 쪽으로 스펙 작성하고, 선언형 계층을 얹는
> 것으로 가자. 이것도 차세대 에이전틱 2d/3d 그래픽스를 지향하는 혁신적 설계를 담아야 해."

In English: a library for building games, replacing three.js. Speed matters, so the abstraction has
to be a good one. Use Filament for 3D and Compose for 2D, put a declarative layer on top, and design
it for the next generation of agentic 2D/3D graphics.

The comparison that led to Filament (wgpu is a GPU API with per-call validation cost and no scene
layer; Filament is a scene-level engine at the height of three.js) is recorded in
python-multiplatform `docs/design/ecosystem.md` §7 and in `docs/research.md` §1 here.

## 1. What this project is for

A Python app on Kotlin Multiplatform (Android, iOS, desktop) can draw interactive 2D and 3D scenes
and games by **declaring** them, the way pythonx-compose declares UI, and those scenes are built so
that an **AI agent** can work on them as well as a person can.

Four jobs:

1. **Declare** 2D and 3D content in Python with one model: functions that describe what is on screen,
   state that drives both UI and scene, and redeclaration that replaces what is shown.
2. **Render fast** on every platform: 2D through Compose (Skia), 3D through Filament (Vulkan, Metal,
   OpenGL ES), with the frame loop running natively and Python involved only when something changes.
3. **Animate and simulate by declaration**: motion, physics and behaviour are described as data that
   the native side executes every frame, not as Python code called every frame.
4. **Open the scene to agents**: a scene that can be described, queried, grounded in pixels, edited
   through validated transactions, rendered headless and replayed, so an agent can build, inspect,
   modify and test a scene end to end.

## 2. User stories

- As a Python developer, I write a 3D scene with a camera, lights and a glTF model in a few lines of
  declarative Python, and it runs at the platform's frame rate on a phone.
- As a game author, I mix a Compose HUD (2D) over a Filament world (3D) and drive both from the same
  `state(...)` values.
- As a notebook user, I redeclare the scene in a cell and the running window shows the new scene, as
  `UI.ipynb` does for UI.
- As an agent, I ask "what is in the scene?", get a structured answer with stable ids, names, bounds
  and spatial relations, find "the red chair" in a rendered frame through an id buffer, and move it
  with an edit that is validated, previewed and undoable.
- As an agent or a CI job, I render a scene headless on desktop with a fixed clock and seed, compare
  it with a reference, and replay a recorded session deterministically.
- As a performance-minded author, I put 10,000 instances in a scene from one array, and no Python runs
  per instance per frame.

## 3. What it is, structurally

- **A Python package** named `pythonx-graphics`, import package `pythonx.graphics` (decided, open
  question 2).
- **Real Python code on disk**, like `pythonx-compose`. It imports Kotlin modules under their own
  Kotlin names through the binder. It never asks the binder to rename anything.
- **A Kotlin layer** that hosts Filament inside Compose (a Compose `Applier` over Filament entities,
  a `Scene3D` composable, the per-frame native systems). Where that layer lives is open question 1.
- **Same declaration model as pythonx-compose**: `@Composable`-style functions, `state(...)`,
  redeclaration with `@app`. A 3D scene is a composition whose nodes are Filament entities.

## 4. How it relates to the other repositories

| Repository | Relationship |
|---|---|
| `python-multiplatform` | Interpreter, binder, upcalls; `PythonAppView` hosts the root. Missing binder features become issues there (`needs binder:`). |
| `pythonx-compose` | Sibling and base: the 2D side is pythonx-compose's Compose surface; this package adds the 3D surface and the agent layer on the same model. |
| `pythonx-concurrent` | Asset loading, generative asset providers and agent loops are async; they use its scopes and dispatchers. |
| TypedPython (in python-multiplatform) | Per-frame game logic that must run in Python is compiled; on free-threaded CPython it can run in parallel. |
| `torchnative` | Optional on-device models for generative asset providers and agents. Never a dependency of the core. |
| Filament (Google) | The 3D renderer. Used, not forked. |

## 5. What it is not

- **Not a GPU API.** No wgpu, Vulkan or Metal surface for users. Custom GPU compute is out of scope
  unless asked.
- **Not a full game engine editor.** No GUI editor; the agent layer and notebooks are the editing
  surfaces.
- **Not a Kotlin renamer.** No `androidx`-to-`pythonx` style mapping lives here or in the binder.
- **Not a bundled AI model.** Generative asset providers and agents are pluggable; this package
  defines the interfaces and ships none.
- **Not identical on every platform.** Where a platform lacks a capability (for example a Filament
  backend or headless rendering), the feature is absent there and says so.

## 6. Open questions

1. **Where the Kotlin layer lives.** Options: (a) a Kotlin module in this repository (a new top-level
   entry, needs approval); (b) `compose-multiplatform-core-extended`, as a multiplatform androidx-style
   library, like OS notifications (#13 there); (c) a separate Kotlin repository. Recommendation: (b)
   if its owners accept it, since that repository is where multiplatform Compose extensions live.
2. **Package and import name: decided.** `pythonx-graphics` / `pythonx.graphics` (maintainer,
   2026-10-04), replacing the first name `pythonx-game`: the scope is agentic 2D/3D graphics, games
   included.
3. **Physics engine.** Declared physics (SPEC §6) needs an engine. Candidates: Jolt Physics (MIT),
   Bullet (zlib), Box2D for 2D (MIT). Not chosen.
4. **Web.** Filament has a WebGL/wasm build and Compose has wasm, but python-multiplatform's wasm
   target is experimental. In scope for the first release or not?
5. **Agent tool protocol.** SPEC §8 generates tool schemas; whether to ship an MCP server adapter in
   this package or leave it to the app is open.
