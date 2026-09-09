
# Creating Assets

## PCX Images

Use [Pix8](https://pix8.app) our own 256-color pixel art editor (browser + Windows desktop app):
1. **File → New...** — images are 8-bit indexed (256 colors) by default, no mode switching needed
2. Draw with 256 colors (any resolution supported)
3. **File → Export as...** (Ctrl+Shift+E) → PCX (RLE-compressed)
   - Common sizes: 320×200 (full screen), 32×32 (sprites), 16×16 (tiles)

Pix8 also exports [SPX](../ENGINE/SPX.md) sprite sheets directly, which is the fastest way
to get animated sprites into the engine. See [Sprite Sheets](#sprite-sheets) below.

Alternatives:
- [GrafX2](http://grafx2.chez.com/) — the classic DOS-style pixel art editor (Windows/Linux/Mac).
  Comparable feature set to Pix8, and it still has a few things Pix8 lacks (dithering, gradient fill).
- [Aseprite](https://www.aseprite.org/) — modern pixel art editor, PCX export (paid).

## Sprite Sheets

Use [Pix8](https://pix8.app) to build the sheet and its animations in one go:
1. Draw each animation frame on the frame timeline, and group frames with tags
2. **File → Export as...** (Ctrl+Shift+E) → SPX
3. You get a ZIP with the `.SPX` sprite definition plus the packed `.PCX` sheet(s) —
   unpack it into your `DATA\` directory and reference the `.SPX` from `resources.xml`

See the [SPX format documentation](../ENGINE/SPX.md) for the file layout.

## VOC Sound Effects

Use [Audacity](https://www.audacityteam.org/) (Windows/Linux/Mac):
1. Import audio (WAV, MP3, etc.)
2. **Tracks → Mix → Mix Stereo Down to Mono**
3. **Tracks → Resample → 11025 Hz** (or 22050 Hz)
4. **File → Export → Export Audio**
   - Format: "Other uncompressed files"
   - Header: "VOC (Creative Labs)"
   - Encoding: "Unsigned 8-bit PCM"

## HSC Music

Use [HSC Tracker JS](https://hsc-tracker-js.dynart.net) our own browser-based HSC tracker
([source](https://github.com/DynartInteractive/HscTrackerJs)):
1. Compose in the pattern editor (9 OPL2 channels, 64 rows, all HSC effects)
2. Edit instruments in the built-in instrument editor, with live ADSR visualization
3. **Ctrl+S** to save as `.HSC` — the engine's native music format, no conversion step

It plays back through a JavaScript YM3812 (OPL2) emulator at the same 18.2 Hz tick rate the
engine uses, so what you hear in the browser is what the game plays.

Alternatives:
- [Adlib Tracker II](https://adlibtracker.net/) — modern desktop tracker (Windows/Linux),
  but its native format is A2M, so getting to `.HSC` needs a conversion step.
- [HSC-Tracker](https://demozoo.org/productions/293837/) — the original HSC tracker (DOS only).

## TMX tilemaps

Use [Tiled](https://www.mapeditor.org/) a full-featured level editor (Windows/Linux/Mac).
See the restrictions at the [tilemap documentation](../BASICS/TILEMAP.md).

