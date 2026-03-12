````markdown
# HTML Overlays — Popups, Menus, UI

## Concept

The engine runs inside a browser. `document.body` is accessible from any script.
HTML overlays are the correct way to build scrollable menus, popups, and any UI that needs native scroll or rich layout.

In production the `<body>` contains only the game canvas — the overlay sits on top of it cleanly.
In the editor preview it covers the editor UI too, but that is expected behaviour.

---

## Base Pattern (from `autoplay.ts`)

```typescript
import { Init } from '@miratech278/editor-kit';
import { GlobalExtension } from "@miratech278/editor-kit";

const html = `<div style="...your overlay..."></div>`

export default class Script extends GlobalExtension<DisplayInterface> {

    private div: HTMLDivElement

    @Init
    private init() {
        this.div = document.createElement('div')
        this.div.innerHTML = html
    }

    public show() {
        document.body.appendChild(this.div)
        document.getElementById('close_btn').onclick = () => this.hide()
    }

    public hide() {
        if (document.body.contains(this.div)) document.body.removeChild(this.div)
    }
}
```

**Key rules:**
- Create `div` once in `@Init`, reuse in every `show()` call
- Always guard `removeChild` with `contains()` check
- Never use template literals with non-Latin1 characters (emoji etc.) — the engine compiles scripts via `btoa()` which will throw `InvalidCharacterError`
- CSS `<style>` tag injection via `document.head` does **not** work in the editor — use inline styles only

---

## Close Button — CSS Only (no image needed)

Two `<span>` elements rotated ±45°, copied from the working `autoplay.ts` pattern:

```html
<button id="btn_close" style="width:70px;height:70px;position:absolute;right:20px;top:20px;background:transparent;border:none;cursor:pointer;outline:none;">
    <span style="height:8px;width:50px;background:white;border-radius:5px;position:absolute;transform:rotate(45deg);top:50%;left:50%;translate:-50% -50%;"></span>
    <span style="height:8px;width:50px;background:white;border-radius:5px;position:absolute;transform:rotate(-45deg);top:50%;left:50%;translate:-50% -50%;"></span>
</button>
```

---

## Loading an Image from the Library into HTML

HTML `<img>` elements cannot directly reference engine assets.
The solution: render the asset to a PIXI `RenderTexture`, then extract it as base64 and assign to `img.src`.

**Confirmed working** — tested 03.03.2026.

### Imports

```typescript
import { Image, Reference, Configurable, Init } from '@miratech278/editor-kit';
import { Sprite, RenderTexture, Renderer, Container } from 'pixi.js';
```

### Field

```typescript
@Configurable
private myIcon: Reference<Image>
```

Assign any image asset from the library in the inspector.

### Convert once in @Init

```typescript
@Init
private async init() {
    if (this.myIcon) {
        this.iconBase64 = await this.toBase64(this.myIcon)
    }
}

private async toBase64(ref: Reference<Image>): Promise<string> {
    const renderer = this.context.application.renderer as Renderer
    const displayObject = this.context.displayManager.initialize(ref, this.display) as Container

    const bounds = displayObject.getBounds()
    const renderTexture = RenderTexture.create({
        width: Math.ceil(bounds.width),
        height: Math.ceil(bounds.height),
        resolution: renderer.resolution,
    })

    renderer.render(displayObject, { renderTexture })

    const sprite = new Sprite(renderTexture)
    const base64 = renderer.extract.base64(sprite)

    renderTexture.destroy(true)
    sprite.destroy()

    return base64
}
```

### Use in show()

```typescript
public show() {
    document.body.appendChild(this.div)

    const img = document.getElementById('my_icon') as HTMLImageElement
    img.src = this.iconBase64
}
```

### HTML placeholder

```html
<img id="my_icon" src="" style="width:80px;height:80px;object-fit:contain;">
```

---

## Loading a JSON Config

Use a `@Configurable` field typed as `Reference<JSON>` (or the engine's config type if available).
Alternatively, keep the config as a plain TypeScript object in the same file — no import needed.

See [05-imports-and-limits.md](05-imports-and-limits.md) for import rules.

---

## Summary

| Task | Solution |
|---|---|
| Show HTML overlay | `document.body.appendChild(div)` |
| Hide overlay | `document.body.removeChild(div)` with `contains()` guard |
| Close button | Two `<span>` rotated ±45° — no image needed |
| Engine image in `<img>` | `Reference<Image>` → `toBase64()` → `img.src` |
| Scrollable list | `overflow-y: auto` + explicit `max-height` on the scroll container |
| Inline CSS only | `<style>` tag injection does not work — all styles must be inline |
````
