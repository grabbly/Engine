# this.view and this.display

## this.display

The PIXI Container that this script is physically attached to.

```typescript
this.display.visible = true
this.display.visible = false
this.display.on("someEvent", handler)  // subscribe to events on this container
```

`LocalExtension.emit("eventName")` fires **on `this.display`** — not on the script object.
This means external scripts can listen on the container directly:

```typescript
// Inside Lobby script:
this.emit("startGamePressed")

// Inside Run script (which holds a reference to the Lobby container):
this.view.UI_Lobby.on("startGamePressed", () => { ... })
```

---

## this.view

A proxy that gives access to **named child objects** of `this.display` by their scene name.

```typescript
// Access child named "StartBtn" in the scene hierarchy:
this.view.StartBtn.on("pointertap", () => { ... })

// Access child named "UI_Lobby":
this.view.UI_Lobby.open()
this.view.UI_Lobby.close()
this.view.UI_Lobby.on("startGamePressed", () => { ... })
```

**The name must match exactly** the object's name in the editor scene tree.

### Typing this.view (optional, for the mirror workspace)

In the real engine `this.view` is dynamically resolved. In the mirror you can use `any` or cast:

```typescript
(this.view as any).UI_Lobby.open()
```

---

## Accessing children: this.view vs @Configurable

| Approach | How | When to use |
|---|---|---|
| `this.view.ChildName` | By **scene name** | Child is always a direct descendant, name is stable |
| `@Configurable` | Assigned in **inspector** | Reference to any object, not necessarily a child |

`this.view` is simpler and requires no inspector wiring — use it for direct children with fixed names.

---

## Example: Run as a Scene Controller

```typescript
export default class Script extends LocalExtension {
    @Init
    private Init() {
        // Open lobby, close others — by scene object name
        this.view.UI_Lobby.open()
        this.view.UI_Gameplay.close()

        // Subscribe to Lobby's event
        this.view.UI_Lobby.on("startGamePressed", () => {
            this.view.UI_Lobby.close()
            this.view.UI_Gameplay.open()
        })
    }
}
```

No imports, no `@Configurable`, no type annotations needed — just scene names.
