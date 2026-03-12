# Engine Documentation

Knowledge base built from hands-on exploration of the SpinCraft engine (PIXI v8 + `@miratech278/editor-kit`).

## Contents

| File | Topic |
|---|---|
| [01-extensions.md](01-extensions.md) | LocalExtension vs GlobalExtension — when to use which |
| [02-view-and-display.md](02-view-and-display.md) | `this.view`, `this.display`, `this.context` |
| [03-communication.md](03-communication.md) | Events, `emit`, `on` — how scripts talk to each other |
| [04-configurable.md](04-configurable.md) | `@Configurable` and `ExtendEvents` — typed inspector references |
| [05-imports-and-limits.md](05-imports-and-limits.md) | What works and what doesn't — import rules |
| [06-decorators.md](06-decorators.md) | `@Init`, `@Update`, `@FixedUpdate`, `@Configurable` |
| [07-library-structure.md](07-library-structure.md) | Library panel — folders, asset types, src/ modules |
| [08-sound.md](08-sound.md) | `Sound` — playing audio via `@Configurable` |
| [10-html-overlays.md](10-html-overlays.md) | HTML overlays — popups, menus, image-to-base64, scroll |

## Key Principles (TL;DR)

1. **Script on a scene object** → `LocalExtension`
2. **Script in the library** → `GlobalExtension`
3. **Access children by name** → `this.view.ChildName`
4. **Communicate between scripts** → `emit` / `on` via display container
5. **No relative imports** between component scripts
6. **Inspector wiring** → `@Configurable` for external references
