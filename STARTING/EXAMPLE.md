# Your First Game

A complete, working game built on the engine framework. Three files: a game unit, a
screen unit, and the program itself.

The framework ([TBaseGame](../ENGINE/BASEGAME.md)) owns the game loop, so none of this
touches VGA setup, timers or interrupt teardown directly.

## 1. Describe your resources

Assets load by name from an XML file. Put this in `DATA\RES.XML`:

```xml
<?xml version="1.0" encoding="US-ASCII"?>
<resources>
  <image name="player" path="PLAYER.PCX" palette="default" />
  <font name="small" path="SMALL.XML" />
  <music name="theme" path="THEME.HSC" />
</resources>
```

See the [Resource Manager](../ENGINE/RESMAN.md) for the full format, and
[Creating Assets](CREATE.md) for how to produce the files.

## 2. The game unit

This holds everything the whole game needs. Screens reach it through the global `Game`.

```pascal
unit MyGame;

interface

uses
  VGA, VGAFont, BaseGame, Screen, Config;

type
  TMyGame = object(TBaseGame)
    PlayerImage: PImage;
    Font: PFont;
    Palette: PPalette;

    constructor Init(AConfig: PConfig; const ResXmlPath: String);
    procedure Start; virtual;
  end;

var
  Game: TMyGame;
  GameConfig: TConfig;

implementation

constructor TMyGame.Init(AConfig: PConfig; const ResXmlPath: String);
begin
  inherited Init(AConfig, ResXmlPath);

  { Resources are not available yet - only plain state here }
  PlayerImage := nil;
  Font := nil;
  Palette := nil;
end;

procedure TMyGame.Start;
begin
  inherited Start;  { Brings up resources, input, sound, timing }

  { Now the resource manager is live }
  PlayerImage := ResMan.GetImage('player');
  Font := ResMan.GetFont('small');
  Palette := ResMan.GetPalette('default');
end;

end.
```

## 3. The game screen

A [screen](../ENGINE/SCREEN.md) is one state of the game. This one moves a sprite with
the arrow keys.

```pascal
unit GameScr;

interface

uses
  VGA, VGAFont, Screen, Keyboard, MyGame;

type
  PGameScreen = ^TGameScreen;
  TGameScreen = object(TScreen)
    X, Y: Real;
    constructor Init;
    procedure PostInit; virtual;
    procedure Show; virtual;
    procedure Update(DeltaTime: Real); virtual;
  end;

implementation

const
  Speed = 60.0;  { Pixels per second }

constructor TGameScreen.Init;
begin
  inherited Init;
  X := 160.0;
  Y := 100.0;
end;

procedure TGameScreen.PostInit;
begin
  inherited PostInit;

  { VGA is up by now - safe to set the palette }
  SetPalette(Game.Palette^);
end;

procedure TGameScreen.Show;
begin
  { Build the static background once }
  ClearFrameBuffer(Game.BackgroundBuffer);
  PrintFontText(4, 4, 'ARROWS TO MOVE, ESC TO QUIT',
                Align_Left, Game.Font^, Game.BackgroundBuffer);

  Game.PlayMusic('theme');
end;

procedure TGameScreen.Update(DeltaTime: Real);
begin
  { Frame-rate independent movement }
  if IsKeyDown(Key_Left)  then X := X - Speed * DeltaTime;
  if IsKeyDown(Key_Right) then X := X + Speed * DeltaTime;
  if IsKeyDown(Key_Up)    then Y := Y - Speed * DeltaTime;
  if IsKeyDown(Key_Down)  then Y := Y + Speed * DeltaTime;

  if IsKeyPressed(Key_Escape) then
    Game.Stop;

  { Restore the background, draw the player, show the frame }
  CopyFrameBuffer(Game.BackgroundBuffer, Game.BackBuffer);
  PutImage(Game.PlayerImage^, Round(X), Round(Y), True, Game.BackBuffer);
  RenderFrameBuffer(Game.BackBuffer);
end;

end.
```

Note what is *not* here: no delta-time arithmetic, no VSync wait, no
`ClearKeyPressed` at the end of the loop. The framework does all of that around your
`Update`.

## 4. The program

```pascal
program First;

uses
  MyGame, GameScr;

var
  GameScreen: PGameScreen;

begin
  GameConfig.Init('CONFIG.INI');

  Game.Init(@GameConfig, 'DATA\RES.XML');
  Game.Start;

  New(GameScreen, Init);
  Game.AddScreen('game', GameScreen);

  Game.SetNextScreen('game');
  Game.Run;

  Game.Done;
  GameConfig.Done;
end.
```

Compile it the way [Building](BUILD.md) describes, and you have a running game with
music, input, double buffering and clean shutdown — including on Ctrl+C.

## Adding a second screen

Screens are how you get a title screen, a menu or a pause state. Register as many as
you like and switch by name:

```pascal
New(TitleScreen, Init);
Game.AddScreen('title', TitleScreen);

New(GameScreen, Init);
Game.AddScreen('game', GameScreen);

Game.SetNextScreen('title');
Game.Run;
```

From inside any screen's `Update`:

```pascal
if IsKeyPressed(Key_Enter) then
  Game.SetNextScreen('game');
```

The switch is deferred to the start of the next frame, so it is safe to call mid-update.
The old screen's `Hide` and the new screen's `Show` run automatically.

## Where to go next

- **[TBaseGame](../ENGINE/BASEGAME.md)** — the full game object API
- **[TScreen](../ENGINE/SCREEN.md)** — the screen lifecycle in detail
- **[Resource Manager](../ENGINE/RESMAN.md)** — loading assets by name
- **[Sprites](../BASICS/SPRITE.md)** — animation instead of a static image
- **[Tilemaps](../BASICS/TILEMAP.md)** — scrolling levels from Tiled
- **[Dirty Rectangles](../ADVANCED/DRECT.md)** — redraw only what changed

If you need to drop below the framework — writing a demo, or driving VGA directly —
every subsystem has its own page under [Graphics & Input](../BASICS/index.md) and
[Audio](../AUDIO/index.md), and they all work standalone.
