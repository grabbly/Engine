# Sound

## How it works

Sound assets are assigned in the inspector using `@Configurable`.  
Call `.play()` on the field to trigger playback.

```typescript
import { Configurable } from "@miratech278/editor-kit"
import { Sound } from "@miratech278/editor-kit"
import { LocalExtension } from "@miratech278/editor-kit"

export default class Script extends LocalExtension {
    @Configurable
    private sound: Sound

    @Init
    public play() {
        this.sound.play()
    }
}
```

## Pattern: multiple sounds in one script

Declare one `@Configurable Sound` field per sound slot.  
Each slot is assigned a different audio asset in the inspector.

```typescript
@Configurable
private eatSound!: Sound

@Configurable
private turnSound!: Sound
```

Use a truthiness guard before playing — the sound may not be assigned in the inspector:

```typescript
if (this.eatSound) this.eatSound.play()
```

## Notes

- `Sound` is imported from `@miratech278/editor-kit`, same as decorators
- The field name becomes the slot label in the inspector
- No `null` check is required if a sound is guaranteed to be assigned,
  but the guard is safer during development
- `play()` is the only confirmed method on the `Sound` type
