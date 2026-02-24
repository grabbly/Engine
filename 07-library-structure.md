# Library Structure

The library panel is the **file system of the engine project**. It starts completely empty — all structure is created manually by the developer.

## Example Layout (user-defined)

The structure below is one way to organize a project — not enforced by the engine:

```
Library  (starts empty)
├── Assets/          ← images, fonts, audio, spine files (user-created folder)
├── Components/      ← scene component folders (user-created folder)
├── src/             ← plain TypeScript modules (user-created folder)
├── HUD              ← GlobalExtension script
└── ScaleButton      ← GlobalExtension script
```

There is no required folder naming. You decide the hierarchy.

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

## Import Paths — Open Question

Since folder structure is user-defined, import paths between files depend entirely on how you organize your library.

Whether `import { SnakeModel } from "src/core/SnakeModel"` works in the engine is **unverified** — the engine's module resolution rules are not yet fully understood.

**What is confirmed:** relative imports between component script files do **not** work.
**What is unknown:** whether imports from a sibling `src/` folder (in the same library) resolve correctly.

This needs a direct test in a real scene script.

---

## GlobalExtension in Library

Scripts created in the library (not attached to a scene object at creation time) are `GlobalExtension` instances.

**How to place them on a scene:** drag from the library onto any scene object, or directly onto the scene itself.

Once placed, they behave as a component on that object — with access to `this.display`, `this.view`, decorators, etc.

Examples: `HUD`, `ScaleButton` — reusable widgets dragged from the library onto scene objects.

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
