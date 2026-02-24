# Decorators

All decorators are imported from `@miratech278/editor-kit`.

```typescript
import { Init, Update, FixedUpdate, Configurable } from "@miratech278/editor-kit"
```

---

## @Init

Runs once when the script is attached to the scene object. Use for setup, event binding, initial state.

```typescript
@Init
private Init() {
    this.view.StartBtn.on("pointertap", () => this.emit("startGamePressed"))
    this.display.visible = false
}
```

Equivalent to `Awake` / `Start` in Unity, or `componentDidMount` in React.

---

## @Update

Runs every frame. Use for visual updates tied to render rate.

```typescript
@Update
private Update() {
    // runs every rendered frame
}
```

Avoid heavy logic here — frame rate dependent.

---

## @FixedUpdate

Runs on a fixed time step. Receives `dt` (delta time in **milliseconds**).

```typescript
@FixedUpdate
private Update(dt: number) {
    this.time -= dt

    if (this.time <= 0) {
        this.snap()
        this.emit("meterFinished")
        return
    }

    this.value += this.delta * dt
}
```

Use for game logic, physics, timers, animations. `dt` is in ms — divide by 1000 for seconds if needed.

The fixed step is set in the engine ticker (default: 50ms = 20 ticks/sec).

---

## @Configurable

Exposes a field to the inspector. The inspector lets you drag-assign any scene object or asset.

```typescript
@Configurable
private startButton: ChildReference<Button>

@Configurable
private lobby: ExtendEvents<LobbyDisplay, LobbyEvents>
```

See [04-configurable.md](04-configurable.md) for full details.

---

## Naming Convention

Engine examples use `PascalCase` for decorated methods (`@Init private Init()`).
This mirrors C# Unity convention. Both `Init()` and `init()` work — pick one and stay consistent.
