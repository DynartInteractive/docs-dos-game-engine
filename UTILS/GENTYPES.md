# Generic Types

Unit: `GenTypes`

Generic pointer, array, and alignment types used across the engine.

## Pointer Types

```pascal
type
  PByte = ^Byte;
  PWord = ^Word;
  PShortString = ^string;
```

Generic pointer types for type-safe pointer arithmetic and memory access.

## Array Types

```pascal
type
  TByteArray = array[0..65520] of Byte;
  PByteArray = ^TByteArray;

  TWordArray = array[0..32000] of Word;
  PWordArray = ^TWordArray;
```

**Purpose:** Large array types for pointer-based memory access.

**Sizes:**
- `TByteArray`: 65,520 bytes (64KB - 16 bytes)
- `TWordArray`: 32,000 words (64,000 bytes)

**Use cases:**
- Direct memory access
- XMS transfers
- DMA buffers
- Large data structures

## Alignment Types

```pascal
type
  TAlign = Byte;

const
  { Horizontal alignment }
  Align_Left   = 1;
  Align_Center = 2;
  Align_Right  = 4;

  { Vertical alignment }
  Align_Top    = 8;
  Align_Middle = 16;
  Align_Bottom = 32;
```

**Purpose:** Text and UI element alignment flags.

**Bit flags:** Can be combined using `or` operator for 2D positioning:
- `Align_Left or Align_Top` - Top-left corner
- `Align_Center or Align_Middle` - Centered both ways
- `Align_Right or Align_Bottom` - Bottom-right corner

**Usage in engine:**
- **VGAFONT**: `PrintFontText`, `PrintFontTextColored` (horizontal only)
- **VGAUI**: Widget alignment and text rendering (future 2D support)

### Horizontal Alignment

**Align_Left (1):**
- X coordinate specifies left edge of content
- Text starts at X position
- Most common default alignment

**Align_Center (2):**
- X coordinate specifies center of content
- Text centered horizontally around X
- Common for titles, centered UI elements

**Align_Right (4):**
- X coordinate specifies right edge of content
- Text ends at X position
- Common for right-aligned scores, values

### Vertical Alignment

**Align_Top (8):**
- Y coordinate specifies top edge of content
- Content aligned to top
- Most common default alignment

**Align_Middle (16):**
- Y coordinate specifies vertical center of content
- Content centered vertically around Y
- Common for vertically centered dialogs

**Align_Bottom (32):**
- Y coordinate specifies bottom edge of content
- Content aligned to bottom
- Common for footer elements

### Checking Alignment

```pascal
{ Check if alignment includes horizontal centering }
if (Align and Align_Center) <> 0 then
  X := X - (Width div 2);

{ Check if alignment includes right alignment }
if (Align and Align_Right) <> 0 then
  X := X - Width;

{ Check if alignment includes vertical middle }
if (Align and Align_Middle) <> 0 then
  Y := Y - (Height div 2);
```

**Pattern:** Use bitwise `and` to test if a specific alignment flag is set.

## Usage Examples

### Memory Access

```pascal
var
  Buf: PByteArray;
  W: PWord;
begin
  GetMem(Buf, 1024);

  { Direct byte access }
  Buf^[0] := $FF;
  Buf^[1] := $00;

  { Word access via pointer }
  W := @Buf^[100];
  W^ := $1234;

  FreeMem(Buf, 1024);
end;
```

### Text Alignment

```pascal
uses VGAFont, GenTypes;

var
  Font: TFont;
  BackBuffer: PFrameBuffer;
begin
  { Left-aligned text }
  PrintFontText(10, 50, 'Score: 1000', Align_Left, Font, BackBuffer);

  { Centered title }
  PrintFontText(160, 10, 'GAME TITLE', Align_Center, Font, BackBuffer);

  { Right-aligned value }
  PrintFontText(310, 50, '9999', Align_Right, Font, BackBuffer);
end;
```

### Combined Alignment (Future)

```pascal
{ Example of 2D alignment (when supported by widget system) }
const
  TopLeft = Align_Left or Align_Top;
  Centered = Align_Center or Align_Middle;
  BottomRight = Align_Right or Align_Bottom;

{ Position widget at bottom-right corner }
Widget.SetAlignment(BottomRight);
```

## Notes

- **Array limits:** `TByteArray` max 65,520 bytes, `TWordArray` max 32,000 words (both fit in 64KB segment)
- **Alignment flags:** Use bitwise operations (`and`, `or`) to combine and test flags
- **Horizontal only:** Currently, most engine functions only use horizontal alignment (Left/Center/Right)
- **Bit flag pattern:** Powers of 2 allow combining multiple flags without conflicts
- **Memory safety:** Always ensure array indices stay within bounds (0..65520 for bytes, 0..32000 for words)
