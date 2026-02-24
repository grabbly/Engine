# Imports and Limitations

## What Works

### Imports from the engine package
```typescript
import { LocalExtension, GlobalExtension } from "@miratech278/editor-kit"
import { Init, FixedUpdate, Configurable } from "@miratech278/editor-kit"
import { ExtendEvents, ChildReference, Button } from "@miratech278/editor-kit"
```

### Imports from PIXI
```typescript
import { Container } from "pixi.js"
```

### Local declarations (interfaces, types, enums) inside the same file
```typescript
interface MyDisplay extends Container { open(): void }
type MyEvents = { done: [] }
```

---

## What Does NOT Work

### Relative imports between component scripts

```typescript
// ❌ This will NOT resolve in the engine
import { LobbyEvents } from "../UI_Lobby.component/Script"
import { LobbyScript } from "./LobbyScript"
```

Why: The engine loads each script as an isolated module. There is no shared file system between component folders at runtime.

### External logic files next to Script.ts

```typescript
// ❌ Cannot import a helper file from the same component folder
import { LobbyScript } from "./LobbyScript"
```

This was confirmed by direct testing — the engine does not resolve sibling files.

---

## Workarounds

| Problem | Solution |
|---|---|
| Need shared types | Re-declare the interface in each file that needs it |
| Need shared logic | Inline it in Script.ts, or use a `GlobalExtension` from the library |
| Need to call another script's method | Use `this.view.ChildName.method()` or `@Configurable` |

---

## The Isolation Rule

Each `Script.ts` file in a component folder is **self-contained**:
- No imports from sibling files
- No imports from other component folders
- All types must be declared inline

This is by design — components are meant to be independent and reusable.

---

## Mirror Workspace Note

In the local mirror (`Project-mirror/`) relative imports work fine for TypeScript type checking, but they reflect a structure that would not work in the actual engine. The mirror exists for offline development and type exploration only.
