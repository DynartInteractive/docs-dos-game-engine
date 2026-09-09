# Screens (TScreen)

Unit: `Screen`

A screen is one state of your game — the title screen, the gameplay screen, the
settings menu. `TScreen` is the base object you extend; this is where most of your
game code actually lives.

The [game object](BASEGAME.md) holds a map of named screens and calls the lifecycle
methods of whichever one is active.

## Lifecycle

| Method     | When it runs                                        | Typical use                          |
|------------|-----------------------------------------------------|--------------------------------------|
| `Init`     | When you construct the screen, before VGA is up     | Plain field initialization           |
| `PostInit` | Once for every screen, after VGA is up, before loop | Set the palette, prepare buffers     |
| `Show`     | Each time the screen becomes active                 | Reset state, start music, draw       |
| `Update`   | Every frame while active                            | Input, logic, rendering              |
| `Hide`     | Each time the screen becomes inactive               | Pause timers, save state             |
| `Done`     | When the game shuts down                            | Free what the screen allocated       |

The split between `Init` and `PostInit` is the one to remember: **screens are
constructed before the display mode is set**, so anything that touches VGA — the
palette above all — belongs in `PostInit`, not `Init`.

`Show` and `Hide` can run many times as the player moves between screens; `PostInit`
and `Done` run exactly once.

## A minimal screen

```pascal
unit TitleScr;

interface

uses
  VGA, Screen, Keyboard, MyGameUnit;

type
  PTitleScreen = ^TTitleScreen;
  TTitleScreen = object(TScreen)
    Elapsed: Real;
    constructor Init;
    procedure PostInit; virtual;
    procedure Show; virtual;
    procedure Update(DeltaTime: Real); virtual;
  end;

implementation

constructor TTitleScreen.Init;
begin
  inherited Init;
  Elapsed := 0.0;
end;

procedure TTitleScreen.PostInit;
begin
  inherited PostInit;

  { VGA is up - safe to touch the palette }
  SetPalette(Game.Palette^);
end;

procedure TTitleScreen.Show;
begin
  { Draw the static background once, then copy it in each frame }
  PutImage(Game.TitleImage^, 0, 0, False, Game.BackgroundBuffer);
  CopyFrameBuffer(Game.BackgroundBuffer, Game.BackBuffer);
  RenderFrameBuffer(Game.BackBuffer);
  Elapsed := 0.0;
end;

procedure TTitleScreen.Update(DeltaTime: Real);
begin
  Elapsed := Elapsed + DeltaTime;

  if IsKeyPressed(Key_Enter) then
    Game.SetNextScreen('game');

  if IsKeyPressed(Key_Escape) then
    Game.Stop;
end;

end.
```

Register it from your main program:

```pascal
New(TitleScreen, Init);
Game.AddScreen('title', TitleScreen);
Game.SetNextScreen('title');
```

## Switching screens

Call `Game.SetNextScreen('name')`. The switch is **deferred** — it happens at the start
of the next frame, never in the middle of one:

1. The old screen's `Hide` runs
2. The active screen changes
3. The new screen's `Show` runs

So it is safe to switch from inside `Update`, including from a click handler, without
worrying about the rest of the frame running against a half-swapped state.

## Rendering

Screens render themselves — the framework does not draw anything for you. In `Update`,
draw into `Game.BackBuffer` and then push it to the display with `RenderFrameBuffer`,
or with `FlushDirtyRects` if you are only updating parts of the screen.

The common pattern is to draw the static parts once into `Game.BackgroundBuffer` in
`Show`, then each frame restore the areas that changed from it before drawing the moving
objects on top. See [dirty rectangles](../ADVANCED/DRECT.md).

## Notes

- Every method is virtual — override only the ones you need.
- Screens reach shared state through the global `Game` instance, which is why they take
  no constructor parameters.
- The game frees all registered screens in `Done`; do not dispose them yourself.
- `DeltaTime` is in seconds, so multiply speeds by it to stay frame-rate independent.

## See Also

- **[TBaseGame](BASEGAME.md)** — the game object that owns and switches screens
- **[Dirty Rectangles](../ADVANCED/DRECT.md)** — partial screen updates
- **[Engine Internals](../ADVANCED/INTERNALS.md)** — where the lifecycle calls come from
