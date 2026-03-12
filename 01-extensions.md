# LocalExtension vs GlobalExtension

## The Core Rule

Where you **create the script** in the editor determines which class to use.

| Where created | Class | Lifetime |
|---|---|---|
| On a **scene object** | `LocalExtension` | Lives and dies with the object |
| In the **library** | `GlobalExtension` | Singleton, survives scene changes |

---

## LocalExtension

A script attached to a specific object in the scene hierarchy.

```typescript
import { LocalExtension } from "@miratech278/editor-kit"

export default class Script extends LocalExtension {
    // ...
}
```

**With typed events:**
```typescript
type Events = {
    startGamePressed: []        // no arguments
    gameOver: [score: number]   // with argument
}

export default class Script extends LocalExtension<Events> {
    // now this.emit("startGamePressed") and this.emit("gameOver", 42) are typed
}
```

**Key properties available:**
- `this.display` — the PIXI Container this script is attached to
- `this.view` — named children of the display object (see [02-view-and-display.md](02-view-and-display.md))

> ⚠️ `LocalExtension` does **NOT** have `this.context`. No `displayManager`, no `application.renderer`.
> Everything that needs engine/PIXI access must live in a `GlobalExtension`.

---

## GlobalExtension

A script created in the **library panel** of the editor, not on a scene object.

```typescript
import { GlobalExtension } from "@miratech278/editor-kit"

interface DisplayInterface {
    setText(text: string): void
}

type Events = {
    meterFinished: []
}

export default class Script extends GlobalExtension<DisplayInterface, Events> {
    // DisplayInterface — shape of the display object
    // Events — events this script can emit
}
```

**Key difference from LocalExtension:**
- Takes **two** generics: `<DisplayInterface, Events>`
- `this.display` is typed as `DisplayInterface`
- Created in the library, then **placed on a scene by dragging** from the library onto any scene object or onto the scene itself
- Behaves as a reusable component — similar to a Unity prefab

---

## Why This Matters

This is why `GlobalExtension` was unavailable in our scene scripts — the engine enforces the distinction at the editor level. A script created on a scene object simply cannot be a `GlobalExtension`.

**Practical rule:**
- Building a screen, button, panel, controller → `LocalExtension`
- Building a reusable widget (win meter, slider, timer) placed from the library → `GlobalExtension`

---

## this.view — Only Sees LocalExtension

`this.view.childName` only exposes methods of `LocalExtension` scripts. If a component has only a `GlobalExtension` on it, `this.view.childName.someMethod()` will fail at runtime — the method won't be found.

```
this.view.menuPopup.show()  ✅ — works if menuPopup.comp has a LocalExtension with show()
this.view.menuPopup.show()  ❌ — fails if menuPopup.comp only has a GlobalExtension
```

---

## Bridge Pattern: LocalExtension + GlobalExtension on the Same Component

When you need a component accessible via `this.view` (LocalExtension) but also needs engine/PIXI access (GlobalExtension), put **both scripts on the same component** and connect them via `this.display` events:

**LocalExtension** — public API, called from outside via `this.view`:
```typescript
export default class Script extends LocalExtension {
    public show() {
        this.display.emit('menu:show')  // passes to GlobalExtension
    }
    public hide() {
        this.display.emit('menu:hide')
    }
}
```

**GlobalExtension** — has `this.context`, does real work, listens on the same container:
```typescript
export default class Script extends GlobalExtension<DisplayInterface> {
    @Init
    private init() {
        this.display.on('menu:show', () => this.show())
        this.display.on('menu:hide', () => this.hide())
    }
    public show() { /* render HTML, load assets, etc */ }
    public hide() { /* cleanup */ }
}
```

Both scripts share the same `this.display` Container — it acts as the event bus between them.

| | LocalExtension | GlobalExtension |
|---|---|---|
| Accessible via `this.view` | ✅ | ❌ |
| Has `this.context` / PIXI access | ❌ | ✅ |
| Role in bridge pattern | Public API (emits events) | Implementation (listens + acts) |
