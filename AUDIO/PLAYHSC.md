# HSC Music Player

Unit: `PlayHSC`

AdLib/OPL2 music player (1994, GLAMOROUS RAY).

## HSC_obj Methods

```pascal
procedure HSC_obj.Init(Port: Word);          { Port: 0=auto-detect }
procedure HSC_obj.LoadFile(const FileName: string);
procedure HSC_obj.LoadMem(Data: Pointer; Size: Word);
procedure HSC_obj.Start;
procedure HSC_obj.Stop;
procedure HSC_obj.Fade;
procedure HSC_obj.Done;
```

## Example

```pascal
uses PlayHSC;

begin
  HSC_obj.Init(0);                  { Auto-detect AdLib port }
  HSC_obj.LoadFile('MENU.HSC');
  HSC_obj.Start;

  { Game loop... }

  HSC_obj.Fade;                     { Fade out }
  HSC_obj.Done;                     { CRITICAL: Unhook IRQ0 }
end;
```

## Critical Notes

1. **IRQ0 Hook**: HSC hooks Timer IRQ0 (18.2 Hz). MUST call `Done` before exit or system hangs.
2. **PIT Conflict**: Don't read PIT Timer 0 while HSC active. Use RTCTIMER (IRQ8) instead.
3. **ExitProc**: Install handler to call `Done` on Ctrl+C:
   ```pascal
   OldExit := ExitProc;
   ExitProc := @MyExitProc;
   ```

## Creating HSC Files

Compose with [HSC Tracker JS](https://hsc-tracker-js.dynart.net), our own browser-based HSC
tracker ([source](https://github.com/DynartInteractive/HscTrackerJs)). It saves `.HSC`
natively (Ctrl+S) and previews through an OPL2 emulator at the same 18.2 Hz tick rate this
player uses, so playback matches the game.

[Adlib Tracker II](https://adlibtracker.net/) and the original DOS
[HSC-Tracker](https://demozoo.org/productions/293837/) also work.

## File Format

See [HSC.md](HSC.md) for HSC file format specification.

## Compatibility

- AdLib (OPL2)
- Sound Blaster FM synthesis
- Requires port $388 (or $220 for SB)
