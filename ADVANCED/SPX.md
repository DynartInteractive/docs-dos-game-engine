# SPX (Sprite XML) Format

External sprite definition file format for the Resource Manager.

## Overview

SPX files (`.SPX`) are XML files that define sprite sheet images and their animations in a single file. They are referenced from `resources.xml` via the `<sprite-xml>` tag.

## Reference in resources.xml

```xml
<sprite-xml path="PLAYER.SPX" />
<sprite-xml path="SPRITES\ENEMIES.SPX" />
```

No `name` attribute is needed. The SPX file itself contains named image and sprite resources that become available through the Resource Manager.

## File Format

Root element: `<sprite-xml>`

### Image Tag

```xml
<image name="player" path="PLAYER.PCX" palette="player" />
```

| Attribute | Required | Description |
|-----------|----------|-------------|
| `name`    | Yes      | Resource name for referencing |
| `path`    | Yes      | PCX file path (relative to SPX file) |
| `palette` | No       | Name for extracted palette resource |

### Sprite Tag

```xml
<sprite name="player_idle" image="player" width="32" height="32" duration="0.8">
  <frame x="0" y="0" />
  <frame x="32" y="0" />
  <frame x="64" y="0" />
</sprite>
```

| Attribute  | Required | Description |
|------------|----------|-------------|
| `name`     | Yes      | Sprite resource name |
| `image`    | Yes      | Reference to image resource |
| `duration` | Yes      | Total animation duration in seconds |
| `width`    | No       | Default frame width (pixels) |
| `height`   | No       | Default frame height (pixels) |

### Frame Tag

```xml
<frame x="0" y="0" />
<frame x="32" y="0" width="48" height="32" offset-x="-8" offset-y="0" duration="0.2" />
```

| Attribute  | Required | Description |
|------------|----------|-------------|
| `x`        | Yes      | X position in sprite sheet |
| `y`        | Yes      | Y position in sprite sheet |
| `width`    | No       | Frame width (inherits from sprite) |
| `height`   | No       | Frame height (inherits from sprite) |
| `offset-x` | No      | Drawing offset X (default 0) |
| `offset-y` | No      | Drawing offset Y (default 0) |
| `duration` | No       | Per-frame duration in seconds |

## Default Width/Height

When `width` and `height` are set on the `<sprite>` tag, all frames inherit these values. Individual frames can override with their own `width`/`height`.

If the sprite has no default size, every frame must specify `width` and `height`.

## Frame Offsets

`offset-x` and `offset-y` adjust the drawing position of a frame. This is useful when a frame has a different size than the sprite default:

```xml
<sprite name="attack" image="player" width="32" height="32" duration="0.6">
  <frame x="0" y="64" />
  <frame x="32" y="64" width="48" height="32" offset-x="-8" />
  <frame x="80" y="64" width="48" height="32" offset-x="-8" />
  <frame x="128" y="64" />
</sprite>
```

The attack frames are wider (48px vs 32px default), so `offset-x="-8"` centers them relative to the normal position. When the sprite is flipped horizontally, offsets are automatically negated.

## Per-Frame Duration

By default, all frames share the total duration equally (`duration / frame_count`). Individual frames can override this:

```xml
<sprite name="walk" image="player" width="32" height="32" duration="0.8">
  <frame x="0" y="0" />
  <frame x="32" y="0" duration="0.3" />
  <frame x="64" y="0" />
  <frame x="96" y="0" />
</sprite>
```

When any frame specifies `duration`, the engine switches to variable frame timing:
- Frames without explicit duration get `sprite.duration / frame_count` as default
- The total animation duration becomes the sum of all frame durations

## Complete Example

```xml
<sprite-xml>
  <image name="player" path="PLAYER.PCX" palette="player" />

  <sprite name="player_idle" image="player" width="32" height="32" duration="0.8">
    <frame x="0" y="0" />
    <frame x="32" y="0" />
    <frame x="64" y="0" />
    <frame x="96" y="0" />
  </sprite>

  <sprite name="player_run" image="player" width="32" height="32" duration="0.5">
    <frame x="0" y="32" />
    <frame x="32" y="32" />
    <frame x="64" y="32" />
    <frame x="96" y="32" />
    <frame x="128" y="32" />
    <frame x="160" y="32" />
  </sprite>

  <sprite name="player_attack" image="player" width="32" height="32" duration="0.6">
    <frame x="0" y="64" />
    <frame x="32" y="64" width="48" height="32" offset-x="-8" duration="0.2" />
    <frame x="80" y="64" width="48" height="32" offset-x="-8" duration="0.2" />
    <frame x="128" y="64" />
  </sprite>
</sprite-xml>
```

## Path Resolution

Image paths in SPX files are relative to the SPX file location, not to `resources.xml`. For example:

```
DATA\
  RES.XML              <- contains: <sprite-xml path="SPRITES\PLAYER.SPX" />
  SPRITES\
    PLAYER.SPX         <- contains: <image path="PLAYER.PCX" />
    PLAYER.PCX         <- loaded from DATA\SPRITES\PLAYER.PCX
```

## Usage in Code

SPX-loaded sprites are accessed identically to inline sprites:

```pascal
ResMan.Init(True);
ResMan.LoadFromXML('DATA\RES.XML');

{ Sprites from SPX files are available by name }
PlayerIdle := ResMan.GetSprite('player_idle');
PlayerRun := ResMan.GetSprite('player_run');
```
