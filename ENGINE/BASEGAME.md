# Game Engine (TBaseGame)

Units: `BaseGame` `Screen`

`TBaseGame` is the framework you build your game on. It owns the game loop, so you
do not have to write one.

## What you get for free

Without the framework you would hand-roll VGA setup, a delta-time loop, and interrupt
teardown — and forgetting a single `Done` call hangs the machine. `TBaseGame` does all
of it:

- **Subsystem setup** — VGA, keyboard, mouse, Sound Blaster, the RTC timer and the
  framebuffers, all brought up in the right order
- **The main loop** — delta timing, screen updates, VSync, keyboard state, every frame
- **Screen management** — named screens with a safe, deferred switch
- **Resources** — a [resource manager](RESMAN.md) fed by an XML file, so images, fonts,
  sprites and music load by name
- **Guaranteed cleanup** — subsystems shut down in reverse order, and an exit handler
  runs even on Ctrl+C or a runtime error

You write two things: a game object that extends `TBaseGame`, and one or more
[screens](SCREEN.md) that extend `TScreen`.

## A complete program

This is the real structure the shipped XiClone game uses:

```pascal
program MyGame;

uses
  MyGameUnit, GameScr;  { MyGameUnit provides Game and GameConfig }

var
  GameScreen: PGameScreen;

begin
  { Configuration }
  GameConfig.Init('CONFIG.INI');

  { Bring up the engine }
  Game.Init(@GameConfig, 'DATA\RES.XML');
  Game.Start;

  { Create and register screens }
  New(GameScreen, Init);
  Game.AddScreen('game', GameScreen);

  { Run }
  Game.SetNextScreen('game');
  Game.Run;

  { Cleanup }
  Game.Done;
  GameConfig.Done;
end.
```

That is the whole entry point. No `InitVGA`, no timer arithmetic, no interrupt
unhooking — `Start`, `Run` and `Done` cover it.

**Order matters:** `Init` then `Start`, then register screens, then `Run`. Registering
after `Start` means your screens can already reach resources your game object loaded.

## Extending TBaseGame

Put your game object in its own unit. This is where game-wide resources and state live,
so screens can reach them through the global `Game`:

```pascal
unit MyGameUnit;

interface

uses
  VGA, VGAFont, BaseGame, Screen, Config;

type
  TMyGame = object(TBaseGame)
    { Game-wide resources }
    TitleImage: PImage;
    PlayerSprite: PImage;
    TitleFont: PFont;
    Palette: PPalette;

    { Game-wide state }
    HighScore: LongInt;

    constructor Init(AConfig: PConfig; const ResXmlPath: String);
    destructor Done; virtual;
    procedure Start; virtual;
  end;

var
  Game: TMyGame;        { Global game instance }
  GameConfig: TConfig;  { Global config instance }

implementation

constructor TMyGame.Init(AConfig: PConfig; const ResXmlPath: String);
begin
  inherited Init(AConfig, ResXmlPath);

  { Plain state only - resources are not available yet }
  HighScore := 0;
  TitleImage := nil;
  PlayerSprite := nil;
  TitleFont := nil;
  Palette := nil;
end;

procedure TMyGame.Start;
begin
  inherited Start;  { Always first - brings up the subsystems }

  { Now the resource manager is live, so grab what the game needs }
  TitleImage := ResMan.GetImage('title');
  PlayerSprite := ResMan.GetImage('player');
  TitleFont := ResMan.GetFont('large');
  Palette := ResMan.GetPalette('default');
end;

destructor TMyGame.Done;
begin
  { ResMan frees everything it loaded - only free what you created yourself }
  inherited Done;
end;

end.
```

**The rule:** `Init` sets plain fields, `Start` loads resources. The resource manager is
not ready until `inherited Start` has run, so a `ResMan.GetImage` in `Init` returns
nothing useful.

`BASEGAME.PAS` deliberately declares no global `Game` variable — your unit provides it.

## API

### Constructor

```pascal
constructor Init(AConfig: PConfig; ResXmlPath: String);
```

- `AConfig` — pointer to an initialized [TConfig](CONFIG.md). **The caller owns it**:
  you initialize it before `Game.Init` and free it after `Game.Done`.
- `ResXmlPath` — path to the resources XML file for the [resource manager](RESMAN.md)

### Lifecycle

```pascal
procedure Start; virtual;
```

Loads config and brings up the resource manager, timer, input, sound and framebuffers.
**Override this** to load your game's resources, calling `inherited Start` first.

VGA is deliberately *not* initialized here — it is initialized in `Run`, so screens can
be created and registered before the display mode changes.

```pascal
procedure Run;
```

Initializes VGA, calls `PostInit` on every registered screen, then runs the main loop
until something calls `Stop`.

```pascal
destructor Done; virtual;
```

Shuts every subsystem down in reverse order and frees all registered screens.

```pascal
procedure Stop;
```

Ends the loop by clearing `Running`. The current frame finishes, then `Run` returns.

```pascal
procedure ResetTiming;
```

Sets `LastTime` to now. Call it after a pause or any long blocking operation, otherwise
the next frame sees a huge delta time.

### Screens

```pascal
procedure AddScreen(Name: String; AScreen: PScreen); virtual;
procedure SetNextScreen(Name: String); virtual;
function GetScreen(Name: String): PScreen;
```

`AddScreen` registers a screen under a name. `SetNextScreen` **queues** a switch — it
takes effect at the start of the next `Update`, never mid-frame. The old screen's `Hide`
and the new screen's `Show` are called for you. See [TScreen](SCREEN.md).

The game owns every registered screen and frees them in `Done`.

### Music

```pascal
procedure PlayMusic(Name: String); virtual;
procedure PauseMusic; virtual;
procedure StopMusic; virtual;
```

Play a music resource by name. All three exit immediately when
`Config.SoundCard = SoundCard_None`, so they are safe to call unconditionally.

### Fields you will use

| Field              | Purpose                                                    |
|--------------------|------------------------------------------------------------|
| `ResMan`           | [Resource manager](RESMAN.md) — images, fonts, sprites, music |
| `Config`           | Pointer to the [config](CONFIG.md) object                   |
| `DeltaTime`        | Seconds since the previous frame                            |
| `BackBuffer`       | Working render buffer — draw here                           |
| `BackgroundBuffer` | Static background, drawn once and copied from               |
| `ScreenBuffer`     | The VGA display buffer (a pointer — never free it)          |
| `Screen`           | The currently active screen                                 |
| `Running`          | Loop flag — clear it with `Stop`                            |

The three-buffer split is what makes partial redraws cheap: keep the untouched
background in `BackgroundBuffer`, restore from it into `BackBuffer`, draw the moving
parts, and push only what changed. See [dirty rectangles](../ADVANCED/DRECT.md).

## Notes

- **Alt+Q** stops the game by default.
- `PostInit` runs for *all* registered screens once, after VGA is up — that is the
  place to set the palette.
- Cleanup is installed automatically, so Ctrl+C and runtime errors still restore text
  mode and unhook interrupts.

## See Also

- **[TScreen](SCREEN.md)** — the screen/state object where your game code lives
- **[Config](CONFIG.md)** — settings file, sound card and input configuration
- **[Resource Manager](RESMAN.md)** — the XML-driven asset loader
- **[Engine Internals](../ADVANCED/INTERNALS.md)** — exact init order, loop body and
  teardown sequence
