# pythonx-game

Declarative 2D and 3D graphics for Python apps on Kotlin Multiplatform: Compose for 2D, Filament for
3D, and a scene built for AI agents as well as people.

[한국어](docs/locale/README_ko.md)

```python
@app
@Composable
def Game():
    Scene3D(
        camera=Camera(position=(0, 1.5, 4), look_at=(0, 0, 0)),
        content=lambda: {
            Sun(intensity=100_000, direction=(-1, -1, -1)),
            Model("models/chair.glb", name="chair", behaviours=[Spin(axis=(0, 1, 0), speed=30)]),
        },
    )
```

- **One model for 2D and 3D.** Scenes are declared like pythonx-compose UI: functions, `state(...)`,
  redeclaration. A 3D scene is a Compose composition whose nodes are Filament entities.
- **The frame loop stays native.** Motion, physics and behaviour are declared as data; Python runs on
  events, not every frame.
- **Agentic by design.** Every scene is a typed document an agent can describe, query, ground in pixels
  (entity-id buffer), edit through validated and undoable transactions, render headless and replay.

Status: specification. There is no code yet. Read [`docs/INTENT.md`](docs/INTENT.md) and
[`docs/SPEC.md`](docs/SPEC.md).

Once published:

```bash
uv add --prerelease allow pythonx-game
```

Licensed under Apache-2.0.
