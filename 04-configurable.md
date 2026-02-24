# @Configurable and ExtendEvents

## @Configurable

Exposes a field in the inspector so you can assign any scene object to it manually.

```typescript
import { Configurable } from "@miratech278/editor-kit"

@Configurable
private startButton: Button

@Configurable
private popup: Container
```

Use `@Configurable` when the referenced object is **not a direct child** or when you want explicit inspector control over the wiring.

---

## ExtendEvents\<Display, Events\>

Used to type a `@Configurable` reference when you want **both**:
- The shape of the object (its public methods)
- The events it can emit

Pattern copied directly from the engine's own code:

```typescript
interface BuyFeaturePopup extends Container {
    open(): void
    close(): void
    setBet(bet: number): void
}

type BuyFeaturePopupEvents = {
    opened: []
    closed: []
    start: []
}

@Configurable
private popup: ExtendEvents<BuyFeaturePopup, BuyFeaturePopupEvents>
```

Now `popup.open()`, `popup.close()`, `popup.on("start", ...)` are all typed.

---

## Defining interfaces for your own screens

Each screen script that will be referenced externally should define and export its interface + events:

```typescript
// UI_Lobby/Script.ts
interface LobbyDisplay extends Container {
    open(): void
    close(): void
}
type LobbyEvents = {
    startGamePressed: []
}
export default class Script extends LocalExtension<LobbyEvents> {
    public open() { this.display.visible = true }
    public close() { this.display.visible = false }
}
```

Then in the referencing script (same file, no import):

```typescript
interface LobbyDisplay extends Container { open(): void; close(): void }
type LobbyEvents = { startGamePressed: [] }

@Configurable
private lobby: ExtendEvents<LobbyDisplay, LobbyEvents>
```

**Important:** Because cross-component imports don't work, the interface must be re-declared in each file that uses it.

---

## ChildReference\<T\>

An alternative to raw typing — used specifically for `Button` and other engine built-in types:

```typescript
import { ChildReference, Button } from "@miratech278/editor-kit"

@Configurable
private startButton: ChildReference<Button>

// then:
this.startButton.on("pointertap", handler)
this.startButton.isEnabled()
this.startButton.enable()
this.startButton.disable()
```

`ChildReference<T>` makes the inspector show a typed object picker for type `T`.

---

## When to use what

| Situation | Use |
|---|---|
| Reference a built-in Button | `ChildReference<Button>` |
| Reference your own screen with events | `ExtendEvents<MyDisplay, MyEvents>` |
| Reference a plain container | `Container` (from pixi.js) |
| Child accessible by scene name | `this.view.ChildName` (no field needed) |
