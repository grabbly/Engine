# SpinCraft Engine — Developer Guide

Community-written documentation for the **SpinCraft engine** (`@miratech278/editor-kit` + PIXI v8).

Built from hands-on exploration and real project code. Contributions and corrections welcome.

---

## Contents

| # | File | Topic |
|---|---|---|
| 01 | [01-extensions.md](01-extensions.md) | `LocalExtension` vs `GlobalExtension` |
| 02 | [02-view-and-display.md](02-view-and-display.md) | `this.view`, `this.display`, `this.context` |
| 03 | [03-communication.md](03-communication.md) | Events — `emit`, `on`, communication patterns |
| 04 | [04-configurable.md](04-configurable.md) | `@Configurable`, `ExtendEvents`, `ChildReference` |
| 05 | [05-imports-and-limits.md](05-imports-and-limits.md) | Import rules and isolation model |
| 06 | [06-decorators.md](06-decorators.md) | `@Init`, `@Update`, `@FixedUpdate` |

---

## Quick Reference

### The two script types

```
Script created on a scene object  →  LocalExtension
Script created in the library     →  GlobalExtension
```

### Accessing children

```typescript
// By scene name — no imports, no @Configurable needed
this.view.UI_Lobby.open()
this.view.StartBtn.on("pointertap", handler)
```

### Events between scripts

```typescript
// Child emits
this.emit("startGamePressed")

// Parent listens (child is in this.view)
this.view.UI_Lobby.on("startGamePressed", () => {
    this.view.UI_Lobby.close()
    this.view.UI_Gameplay.open()
})
```

### Typed external reference

```typescript
interface LobbyDisplay extends Container {
    open(): void
    close(): void
}
type LobbyEvents = { startGamePressed: [] }

@Configurable
private lobby: ExtendEvents<LobbyDisplay, LobbyEvents>
```

### Game loop

```typescript
@Init    private Init()           { /* setup */        }
@Update  private Update()         { /* every frame */  }
@FixedUpdate private Update(dt: number) { /* fixed step, dt in ms */ }
```

---

## Stack

- **Renderer**: PIXI.js v8
- **Component system**: `@miratech278/editor-kit`
- **Language**: TypeScript
- **Editor**: SpinCraft web editor at [spincraft.io](https://spincraft.io)

---

## Status

This is an **unofficial community guide** — not the official engine documentation.
Written based on code examples and experimentation. Some sections may be incomplete or inaccurate.
