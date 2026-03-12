# Script Communication — Events

## How emit Works

`LocalExtension.emit("eventName", ...args)` fires the event **on the display container** (`this.display`).

This means the event travels on the PIXI EventEmitter built into the container — not on the script class itself.

```
Script.emit("startGamePressed")
       ↓
this.display.emit("startGamePressed")   ← fires here
       ↑
anyone holding a reference to this container can listen with .on(...)
```

---

## Emitting an Event

```typescript
type LobbyEvents = {
    startGamePressed: []
}

export default class Script extends LocalExtension<LobbyEvents> {
    @Init
    private Init() {
        this.view.StartBtn.on("pointertap", () => {
            this.emit("startGamePressed")   // fires on this.display
        })
    }
}
```

---

## Listening to an Event (from another script)

The listener must have a reference to the **container** of the emitting script.
The easiest way is `this.view.ChildName` if the emitting object is a child:

```typescript
export default class Script extends LocalExtension {
    @Init
    private Init() {
        // UI_Lobby is a child object whose script emits "startGamePressed"
        this.view.UI_Lobby.on("startGamePressed", () => {
            this.view.UI_Lobby.close()
            this.view.UI_Gameplay.open()
        })
    }
}
```

Or via `@Configurable` if the object is not a direct child:

```typescript
@Configurable
private lobby: ExtendEvents<LobbyDisplay, LobbyEvents>

// then:
this.lobby.on("startGamePressed", () => { ... })
```

---

## Listening on this.display (incoming events)

A script can also receive events from the outside world via its own display:

```typescript
// Someone outside does: lobbyContainer.emit("bet", 100)
this.display.on("bet", (bet) => {
    this.view.betPanel.setBet(bet)
})
```

This is how parent scripts send data down to children.

---

## Communication Patterns — Summary

> **Scope rule:** a script can only see its own direct children via `this.view`.
> Two children of different parents are invisible to each other.
> Cross-subtree communication must go through a common parent or via `@Configurable`.

| Pattern | Code | Use case |
|---|---|---|
| Child → Parent | `this.emit("event")` in child, `this.view.Child.on(...)` in parent | Button press, screen done |
| Parent → Child | `this.view.Child.emit("event", data)` or call public method | Update state, send data |
| Sibling via parent | Parent listens to A, then calls B — **only way siblings can interact** | Screen switching in Run |
| Cross-subtree | `@Configurable` field assigned in inspector | Objects in different branches of the scene tree |
| Public method | `this.view.Child.open()` | Direct control with typed interface |

---

## Bridge Pattern: LocalExtension ↔ GlobalExtension on Same Component

`this.view` only exposes `LocalExtension` methods — `GlobalExtension` is invisible to it.
When you need engine/PIXI access (`this.context`) but also `this.view` visibility, put **both scripts on the same component** and connect via `this.display`:

```typescript
// LocalExtension — the public API, callable via this.view.myComp.show()
export default class Script extends LocalExtension {
    public show() { this.display.emit('menu:show') }
    public hide() { this.display.emit('menu:hide') }
}

// GlobalExtension — has this.context, does the real work
export default class Script extends GlobalExtension<DisplayInterface> {
    @Init
    private init() {
        this.display.on('menu:show', () => this.show())
        this.display.on('menu:hide', () => this.hide())
    }
    public show() { /* render HTML, load assets via this.context */ }
    public hide() { /* cleanup */ }
}
```

Both scripts share the same `this.display` Container — it acts as the event bus between them.

> ⚠️ `LocalExtension` does **NOT** have `this.context` — no `displayManager`, no `application.renderer`.
> If you need engine access, that code must live in the `GlobalExtension`.

---

## Bridge Pattern: LocalExtension ↔ GlobalExtension on Same Component

`this.view` only exposes `LocalExtension` methods — `GlobalExtension` is invisible to it.
When you need engine/PIXI access (`this.context`) but also `this.view` visibility, put **both scripts on the same component** and connect via `this.display`:

```typescript
// LocalExtension — the public API, callable via this.view.myComp.show()
export default class Script extends LocalExtension {
    public show() { this.display.emit('menu:show') }
    public hide() { this.display.emit('menu:hide') }
}

// GlobalExtension — has this.context, does the real work
export default class Script extends GlobalExtension<DisplayInterface> {
    @Init
    private init() {
        this.display.on('menu:show', () => this.show())
        this.display.on('menu:hide', () => this.hide())
    }
    public show() { /* render HTML, load assets via this.context */ }
    public hide() { /* cleanup */ }
}
```

Both scripts share the same `this.display` Container — it acts as the event bus between them.

> ⚠️ `LocalExtension` does **NOT** have `this.context` — no `displayManager`, no `application.renderer`.
> If you need engine access, that code must live in the `GlobalExtension`.
