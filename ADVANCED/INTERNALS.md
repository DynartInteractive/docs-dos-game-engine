# Engine Internals

Units: `BaseGame` `Screen`

Exact initialization order, main loop body and teardown sequence for
[TBaseGame](../ENGINE/BASEGAME.md). You do not need any of this to write a game — read
it when you are debugging startup order, overriding `Update`, or adding a subsystem.

## Architecture

`BASEGAME.PAS` and `SCREEN.PAS` are a reusable framework with no game-specific
dependencies:

- **BASEGAME.PAS** — `TBaseGame`, the game object
- **SCREEN.PAS** — `TScreen`, the screen/state object

Games extend `TBaseGame` with their own resources, state and initialization. The shipped
XiClone game does this as `TXiCloneGame` in `XIGAME.PAS`.

## What Start does

```pascal
procedure Start; virtual;
```

1. Install `CleanupOnExit` via `ExitProc` (handles Ctrl+C and Break gracefully)
2. Load config: `Config^.Load`
3. Initialize the resource manager: `ResMan.Init(True)` (lazy loading), then
   `ResMan.LoadFromXML(ResFilePath)` — **a failure here prints the error and `Halt(1)`s**
4. Initialize the RTC timer: `InitRTC(1024)` — 1024 Hz, millisecond precision
5. Initialize the keyboard: `InitKeyboard`
6. Initialize Sound Blaster: `ResetDSP(Config^.SBPort, Config^.SBIRQ, Config^.SBDMA, 0)`
   when `Config^.SoundCard = SoundCard_SoundBlaster` (2)
7. Initialize the mouse: `InitMouse` when `Config^.UseMouse = 1`
8. Create framebuffers:
   - `BackgroundBuffer := CreateFrameBuffer`
   - `BackBuffer := CreateFrameBuffer`
   - `ScreenBuffer := GetScreenBuffer`

**`InitVGA` is not called here.** It is deferred to `Run` so screens can be created and
registered before the display mode changes.

## The main loop

```pascal
procedure TBaseGame.Run;
begin
  InitVGA;  { VGA initialized here, not in Start }
  VGAInitialized := True;
  Running := True;

  { PostInit all screens - called once for ALL registered screens }
  for I := 0 to StrMap.MAX_ENTRIES - 1 do
  begin
    Entry := ScreenMap.Entries[I];
    if (Entry <> nil) and Entry^.Used then
    begin
      ScreenPtr := PScreen(Entry^.Value);
      if ScreenPtr <> nil then
        ScreenPtr^.PostInit;
    end;
  end;

  LastTime := GetTimeSeconds;

  while Running do
  begin
    { Calculate delta time }
    CurrentTime := GetTimeSeconds;
    DeltaTime := CurrentTime - LastTime;
    LastTime := CurrentTime;

    Update(DeltaTime);
  end;
end;
```

## The per-frame Update

```pascal
procedure Update(DeltaTime: Real); virtual;
```

Virtual — override it to hook the frame, calling `inherited Update` to keep the default
behaviour:

1. `SetScreen` — apply a queued screen switch when `NextScreen <> nil`
2. `UpdateMouse` when the mouse is initialized
3. Handle the exit shortcut: `Alt+Q` clears `Running`
4. `Screen^.Update(DeltaTime)` when a screen is active
5. `WaitForVSync`
6. `ClearKeyPressed`

Screens render themselves — step 4 is where all drawing happens.

## Screen switching

```pascal
procedure SetScreen; virtual;
```

Called automatically by `Update`; call it directly only if you need an immediate switch:

1. Exit when `NextScreen = nil`
2. `Screen^.Hide` when a screen is active
3. `Screen := NextScreen`
4. `Screen^.Show` on the new screen
5. `NextScreen := nil`

## What Done does

Reverse order of `Start`:

1. Free framebuffers: `BackBuffer`, then `BackgroundBuffer`
2. Free screens: iterate the screen map, `Dispose(ScreenPtr, Done)`
3. Free the screen map: `MapFree(ScreenMap)`
4. `DoneMouse` when initialized
5. `UninstallHandler` when sound was initialized
6. `DoneKeyboard`
7. `DoneRTC`
8. `ResMan.Done`
9. `DoneVGA` when VGA was initialized
10. Clear the `CurrentGame` pointer

Note that `ResMan.Done` runs *before* `DoneVGA` — resources are released while the
display mode is still set.

`CleanupOnExit` calls `Done` through the module-level `CurrentGame` pointer, so an
abnormal exit still restores text mode and unhooks every interrupt.

## TBaseGame fields

```pascal
type
  PBaseGame = ^TBaseGame;
  TBaseGame = object
    { Configuration & Resources }
    Config: PConfig;                 { Game configuration pointer (caller owns) }
    ResFilePath: String;             { Path to resources XML }
    ResMan: TResourceManager;        { Resource manager (see RESMAN.PAS) }

    { Timing }
    CurrentTime: Real;               { Current time in seconds (from GetTimeSeconds) }
    LastTime: Real;                  { Previous frame time in seconds }
    DeltaTime: Real;                 { Time elapsed since last frame (seconds) }

    { State }
    Running: Boolean;                { Main loop control flag }

    { Screen Management }
    Screen: PScreen;                 { Current active screen }
    NextScreen: PScreen;             { Next screen to switch to (queued) }
    ScreenMap: TStringMap;           { Name -> PScreen mapping }

    { Framebuffers }
    BackgroundBuffer: PFrameBuffer;  { Static background (cleared once) }
    BackBuffer: PFrameBuffer;        { Working render buffer }
    ScreenBuffer: PFrameBuffer;      { VGA display buffer (from GetScreenBuffer) }

    { Internal state }
    VGAInitialized: Boolean;         { VGA mode 13h initialized }
    MouseInitialized: Boolean;       { Mouse driver initialized }
    SoundInitialized: Boolean;       { Sound Blaster initialized }
  end;
```

## Dependencies

- **CONFIG** — TConfig, LoadConfig, SoundCard constants
- **RESMAN** — TResourceManager
- **RTCTIMER** — InitRTC, DoneRTC, GetTimeSeconds
- **KEYBOARD** — InitKeyboard, DoneKeyboard, IsKeyPressed, ClearKeyPressed
- **SBDSP** — ResetDSP, UninstallHandler
- **MOUSE** — InitMouse, DoneMouse
- **VGA** — CreateFrameBuffer, FreeFrameBuffer, GetScreenBuffer, ClearFrameBuffer,
  CopyFrameBuffer, RenderFrameBuffer
- **STRMAP** — TStringMap (screen name to PScreen mapping)

## Notes

- **DeltaTime convention** — Real, in seconds, from `GetTimeSeconds`.
- **Sound card checks** — the music methods exit early when
  `Config.SoundCard = SoundCard_None`.
- **Framebuffer roles** — `BackgroundBuffer` holds static content, `BackBuffer` is the
  working buffer, `ScreenBuffer` is the VGA hardware buffer and must never be freed.
- **Alt+Q** stops the game. TODO: make this optional.

## Future Enhancements

- **Screen transitions** — fade in/out, wipes
- **Screen stack** — push/pop for pause menus and dialogs
- **Fixed timestep** — decouple update rate from render rate for deterministic physics

## See Also

- **[TBaseGame](../ENGINE/BASEGAME.md)** — how to actually use the framework
- **[TScreen](../ENGINE/SCREEN.md)** — the screen lifecycle
