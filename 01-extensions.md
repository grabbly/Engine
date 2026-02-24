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
- `this.context` — engine context (sound manager, etc.)

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
