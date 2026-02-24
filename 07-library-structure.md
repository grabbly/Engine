# Library Structure

The library panel is the file system of the engine project. Everything lives here.

## Panel Layout

```
Library
├── Assets/          ← images, fonts, audio, spine files
├── Components/      ← scene component folders (each has a Script.ts)
├── src/             ← plain TypeScript modules (logic, utilities)
├── HUD              ← GlobalExtension script (lives in library root)
└── ScaleButton      ← GlobalExtension script (lives in library root)
```

---

## What You Can Create (Right-click → New)

### Extension
| Type | Description |
|---|---|
| **Script** | TypeScript script → `LocalExtension` (on scene object) or `GlobalExtension` (in library) |
| **Visual Script** | Node-based visual programming |
| **Plugin** | Low-level engine plugin (unknown scope) |

### Assets
| Type | Description |
|---|---|
| **Directory** | Create a folder |
| **Files** | Upload files (images, fonts, audio, spine, etc.) |

---

## The `src/` Folder

The `src/` folder exists in the library alongside `Assets/` and `Components/`.

This is where **plain TypeScript modules** live — classes with no engine dependency (pure logic, utilities, services).

**Key insight:** imports from `src/` likely work from component scripts, because the engine resolves paths within the same library project:

```typescript
// This may work in the engine (needs verification):
import { SnakeModel } from "src/core/SnakeModel"
```

This is the primary candidate for housing `SnakeModel`, `GridRules`, `RandomService` without wrapping them in `GlobalExtension`.

**Status:** unverified — needs a test import in a real scene script.

---

## GlobalExtension in Library Root

Scripts created directly in the library (not inside a Component folder) are `GlobalExtension` instances.

Examples visible in the library panel:
- `HUD` — full game HUD as a reusable component
- `ScaleButton` — reusable button with scale animation

These are drag-placed from the library onto scene objects, just like Unity prefabs.

---

## Component Folder Convention

Each folder inside `Components/` represents one scene component:

```
Components/
├── Run.component/
│   └── Run Local Script.ts
├── UI_Lobby.component/
│   └── Lobby Local Script.ts
├── UI_Gameplay.component/
│   └── Local Script.ts
└── UI_Gameover.component/
    └── (script)
```

The folder name is the component name shown in the editor.
The script inside is the `LocalExtension` attached to the scene object.

---

## Supported Asset Types

Based on the engine stack (`PIXI v8 + @pixi/sound + spine-pixi-v8`):

| Category | Formats |
|---|---|
| Images | PNG, JPG, WebP, SVG |
| Texture atlases | JSON + PNG (PIXI atlas format) |
| Fonts | TTF, WOFF, bitmap fonts |
| Audio | MP3, OGG, WAV |
| Spine | `.skel` / `.json` + `.atlas` + PNG |
