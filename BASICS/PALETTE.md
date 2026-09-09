# Palette Workflow

Mode 13h gives you one 256-color palette for the whole screen. If your game shows a
full-screen picture *and* a HUD, both have to share it — so you split the palette into
a fixed range and a swappable range.

This page covers the whole pipeline: reserving the ranges, authoring the art in
[Pix8](https://pix8.app), and swapping palettes at runtime. For the API itself see
[VGA](VGA.md).

## Splitting the palette

Decide the split once, then keep it for the life of the project. A typical layout for a
game with full-screen artwork:

| Range     | Count | Contents                         | Changes           |
|-----------|-------|----------------------------------|-------------------|
| `0`–`31`  | 32    | HUD, fonts, sprites, UI widgets  | Never             |
| `32`–`255`| 224   | The current full-screen image    | Per picture/level |

Color 0 stays transparent by engine convention, so keep it in the fixed range.

The proportions are yours to pick — the reverse split (224 for game graphics, 32 for
UI at the top) works equally well. What matters is that **one range never moves**, so
your HUD colors stay stable while backgrounds come and go.

## Authoring in Pix8

### 1. The fixed UI palette

Build your 32 UI colors once and save them on their own:

1. Open the palette editor
2. Enable **6-bit** mode — the VGA DAC is 6 bits per channel, so this shows you the
   colors the hardware will actually produce
3. Set up entries `0`–`31`
4. **Save** the palette as **6-bit raw binary** to `UI.PAL`

The engine's `LoadPalette` reads a raw 768-byte file of 0–63 values. Choose 6-bit raw,
**not** 8-bit JASC-PAL — the text format will not load.

### 2. Each full-screen image

Quantize to the size of your swappable range, then move the block into position:

1. **File → Open** the source image (PNG/JPG/WebP)
2. In the import dialog set **Colors** to `224` and pick a dither mode
   (Floyd–Steinberg for photographic art, None for flat illustration)
3. The quantized colors land at indices `0`–`223`. Open the palette editor, select the
   range `0`–`223`, and **X-Swap** it onto index `32`.
4. Paste or re-enter your 32 UI colors into `0`–`31` — or load `UI.PAL`
5. **File → Export as...** → **PCX**

**Use X-Swap, not Swap.** Swap moves the colors only; the pixels still point at the old
indices and the image scrambles. X-Swap remaps the pixel data along with the colors, so
the picture looks identical while the indices move underneath. The same distinction
applies to every reordering operation in the palette editor.

## Loading at runtime

Apply the fixed range once at startup, then only ever upload the swappable range:

```pascal
uses VGA, PCX;

var
  UIPal: TPalette;
  StagePal: TPalette;
  StageImage: TImage;

begin
  { Once at startup - the HUD colors }
  LoadPalette('DATA\UI.PAL', UIPal);
  SetPartialPalette(UIPal, 0, 31);

  { Per picture - only touches 32-255, the HUD is unaffected }
  LoadPCXWithPalette('DATA\STAGE1.PCX', StageImage, StagePal);
  SetPartialPalette(StagePal, 32, 255);
```

`SetPartialPalette` indexes the source array at `FromColor`, so entries `32`–`255` of
`StagePal` go to DAC registers `32`–`255`. Whatever the stage PCX happens to hold in
`0`–`31` is never uploaded, which means **a stage file cannot corrupt your HUD colors**.

That is the reason to keep `UI.PAL` separate rather than relying on every stage PCX
carrying identical UI entries: with the separate file, nothing needs to stay in sync
across your image set.

Swapping to the next picture is then just:

```pascal
LoadPCXWithPalette('DATA\STAGE2.PCX', StageImage, StagePal);
SetPartialPalette(StagePal, 32, 255);
```

## Avoiding the flash

Uploading a palette while the new image is only half-drawn shows a frame of wrong
colors. Two ways around it:

- **Draw first, then swap** — render the image into a framebuffer, `WaitForVSync`, then
  call `SetPartialPalette`
- **Fade through black** — ramp the swappable range down to black, load the new image
  and palette, then ramp back up. Because you are only touching `32`–`255`, the HUD
  stays lit the whole time

## Palette effects

`RotatePalette(StartColor, Count, Direction)` cycles a sub-range in the DAC directly,
which is how you get running water, fire or marquee lights for free. Confine it to a
handful of indices inside your swappable range, and it costs nothing per frame:

```pascal
RotatePalette(240, 16, 1);  { Cycle colors 240-255 to the right }
```

It writes the hardware palette directly — no `SetPalette` call afterwards.

## See Also

- **[VGA](VGA.md)** — `SetPalette`, `SetPartialPalette`, `RotatePalette`, `LoadPalette`
- **[PCX](PCX.md)** — the image format, and how its palette is stored
- **[Creating Assets](../STARTING/CREATE.md)** — producing images with Pix8
