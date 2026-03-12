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

## @Configurable with a plain data object

`@Configurable` works not only for display references but also for **plain data objects** (JSON-like structs assigned in the inspector).

```typescript
import { Configurable, Init } from '@miratech278/editor-kit';
import { LocalExtension } from "@miratech278/editor-kit";

type DataObjectType = {
    key: string,
    value: number
}

export default class Script extends LocalExtension {
    @Configurable
    private dataObject: DataObjectType

    @Init
    private init() {
        console.log(this.dataObject.key, this.dataObject.value)
    }
}
```

Rules:
- Define the type with `type` or `interface` in the same file.
- The inspector will show matching JSON fields for assignment.
- No import of `Reference<T>` needed — just the plain type.
- Field is read-only after `@Init` (don't reassign).
- **Only flat objects work** (`{ key: string, value: number }`). Arrays of objects (`MenuItem[]`) are not supported — the inspector has no UI for editing arbitrary nested arrays. For array-based configs see the pattern below.

---

## @Configurable with a JSON string (array/nested config)

Since the inspector supports **strings**, you can pass any complex structure as a JSON string and parse it in `@Init`. This works for arrays, nested objects — anything you can serialize.

```typescript
interface MenuItem {
    id: string
    title: string
    event_name: string
}

const DEFAULT_ITEMS: MenuItem[] = [
    { id: 'm01', title: 'Home',  event_name: 'open_home'  },
    { id: 'm02', title: 'Game',  event_name: 'open_game'  },
    { id: 'm03', title: 'Rules', event_name: 'open_rules' },
]

export default class Script extends LocalExtension {

    // Inspector: paste the JSON array as a string, e.g.:
    // [{"id":"m01","title":"Home","event_name":"open_home"},...]
    @Configurable
    private itemsJson: string

    private items: MenuItem[] = DEFAULT_ITEMS

    @Init
    private init() {
        if (this.itemsJson) {
            try {
                this.items = JSON.parse(this.itemsJson) as MenuItem[]
            } catch (e) {
                console.warn('[Script] itemsJson parse failed, using defaults:', e)
            }
        }
    }
}
```

Rules:
- The field type is just `string` — the inspector renders a text input.
- Paste valid JSON directly into the inspector field.
- Always wrap in `try/catch` — bad JSON must not crash `@Init`.
- Provide a `DEFAULT_*` constant as a fallback when the field is empty.

---

## When to use what

| Situation | Use |
|---|---|
| Reference a built-in Button | `ChildReference<Button>` |
| Reference your own screen with events | `ExtendEvents<MyDisplay, MyEvents>` |
| Reference a plain container | `Container` (from pixi.js) |
| Child accessible by scene name | `this.view.ChildName` (no field needed) |
| Plain flat config object | plain `type` / `interface` directly |
| Array or nested config from inspector | `@Configurable private xJson: string` + `JSON.parse` |
