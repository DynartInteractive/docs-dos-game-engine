# Configuration Management

Unit: `Config`

Extensible configuration system for loading/saving CONFIG.INI settings. Uses object-oriented design with virtual methods, allowing games to extend TConfig with custom configuration fields.

## Architecture

**TConfig** is an extensible object (not a fixed record) that can be inherited to add game-specific configuration fields. The base TConfig handles core engine settings (sound card, mouse), while derived configs can add custom settings (difficulty, volume, etc.).

**Pattern:** Similar to how games extend `TGame` (see DGECORE.md), games can extend `TConfig` to add custom configuration fields and override virtual methods.

## Types

```pascal
const
  SoundCard_None = 0;
  SoundCard_AdLib = 1;
  SoundCard_SoundBlaster = 2;

const
  SoundCardNames: array[0..2] of string = (
    'None',
    'AdLib',
    'Sound Blaster'
  );

const
  GameTitle = 'Game Title';
  GameVersion = '1.0.0';
  TileSize = 16;

type
  PConfig = ^TConfig;

  TConfig = object
    { Configuration file path }
    Path: String;

    { Core engine settings }
    SoundCard: Byte;     { 0=None, 1=AdLib, 2=SoundBlaster }
    SBPort: Word;        { SoundBlaster base (2=$220, 4=$240, 6=$260, 8=$280) }
    SBIRQ: Byte;         { SoundBlaster IRQ (e.g., 5 or 7) }
    SBDMA: Byte;         { SoundBlaster DMA (1, 2, 3, or 4) }
    UseMouse: Byte;      { 0=No, 1=Yes }

    constructor Init(const ConfigIniPath: String);
    destructor Done; virtual;
    procedure Load; virtual;
    procedure Save; virtual;
    procedure SetDefaults; virtual;
    procedure ParseKeyValue(const Key, Value: String); virtual;
    procedure WriteSettings(var F: Text); virtual;
  end;
```

## Methods

### Init

```pascal
constructor Init(const ConfigIniPath: String);
```

Initialize config object and set default values.

**Parameters:**
- `ConfigIniPath` - Path to CONFIG.INI file

**Actions:**
1. Store `Path := ConfigIniPath`
2. Call `SetDefaults` to initialize all fields

**Example:**
```pascal
var
  GameConfig: TConfig;
begin
  GameConfig.Init('CONFIG.INI');
  { Config ready, defaults set }
end;
```

### Done

```pascal
destructor Done; virtual;
```

Cleanup config object. Base implementation is empty. Override in derived configs to cleanup custom resources.

### SetDefaults

```pascal
procedure SetDefaults; virtual;
```

Set default configuration values. Called automatically by `Init`.

**Base implementation defaults:**
- `SoundCard := SoundCard_None` (0)
- `SBPort := 2` ($220, most common)
- `SBIRQ := 5`
- `SBDMA := 1`
- `UseMouse := 0` (disabled)

**Override** in derived configs to set custom defaults (call inherited first).

### Load

```pascal
procedure Load; virtual;
```

Load configuration from INI file specified in `Path`.

**Process:**
1. Call `SetDefaults` first (ensures valid state if file missing)
2. Try to open file at `Path`
3. Parse each line:
   - Skip empty lines
   - Skip comments (lines starting with `;`)
   - Skip section headers (lines starting with `[`)
   - Parse `key=value` pairs
   - Call `ParseKeyValue(Key, Value)` for each pair
4. Close file

**Example:**
```pascal
var
  GameConfig: TConfig;
begin
  GameConfig.Init('CONFIG.INI');
  GameConfig.Load;  { Loads from 'CONFIG.INI' }
end;
```

### Save

```pascal
procedure Save; virtual;
```

Save configuration to INI file specified in `Path`.

**Process:**
1. Create/overwrite file at `Path`
2. Write header comment
3. Call `WriteSettings(F)` to write all settings
4. Close file

**Example:**
```pascal
GameConfig.SoundCard := SoundCard_SoundBlaster;
GameConfig.Save;  { Saves to Path }
```

### ParseKeyValue

```pascal
procedure ParseKeyValue(const Key, Value: String); virtual;
```

Parse a key=value pair from INI file. **Virtual** - override to handle custom keys.

**Base implementation handles:**
- `SoundCard` → `StrToInt(Value)`
- `SBPort` → `StrToInt(Value)`
- `SBIRQ` → `StrToInt(Value)`
- `SBDMA` → `StrToInt(Value)`
- `UseMouse` → `StrToInt(Value)`

**Override pattern:**
```pascal
procedure TMyConfig.ParseKeyValue(const Key, Value: String);
begin
  if Key = 'Difficulty' then
    Difficulty := StrToInt(Value)
  else if Key = 'MusicVolume' then
    MusicVolume := StrToInt(Value)
  else
    inherited ParseKeyValue(Key, Value);  { Let parent handle engine keys }
end;
```

### WriteSettings

```pascal
procedure WriteSettings(var F: Text); virtual;
```

Write configuration settings to file. **Virtual** - override to write custom sections.

**Base implementation writes:**
```ini
[Sound]
SoundCard=0
SBPort=2
SBIRQ=5
SBDMA=1

[Input]
UseMouse=0
```

**Override pattern:**
```pascal
procedure TMyConfig.WriteSettings(var F: Text);
begin
  inherited WriteSettings(F);  { Write parent settings }
  WriteLn(F, '');
  WriteLn(F, '[Game]');
  WriteLn(F, 'Difficulty=', Difficulty);
  WriteLn(F, 'MusicVolume=', MusicVolume);
end;
```

## Backward-Compatible Functions

For compatibility with SETUP.PAS and legacy code:

```pascal
procedure LoadConfig(var Config: TConfig; const ConfigFile: String);
procedure SaveConfig(var Config: TConfig; const ConfigFile: String);
```

**LoadConfig:**
- Calls `Config.Init(ConfigFile)` then `Config.Load`

**SaveConfig:**
- Temporarily changes `Config.Path`, calls `Config.Save`, restores path

**Note:** New code should use object methods instead.

## Basic Usage

```pascal
uses Config, SBDSP, PlayHSC;

var
  GameConfig: TConfig;
begin
  { Initialize and load }
  GameConfig.Init('CONFIG.INI');
  GameConfig.Load;

  { Use configuration }
  if GameConfig.SoundCard = SoundCard_SoundBlaster then
    ResetDSP(GameConfig.SBPort, GameConfig.SBIRQ, GameConfig.SBDMA, 0);

  if GameConfig.SoundCard >= SoundCard_AdLib then
  begin
    HSC_obj.Init(0);
    HSC_obj.LoadFile('MUSIC.HSC');
  end;

  { Cleanup }
  GameConfig.Done;
end;
```

## Extending TConfig

Create a game-specific config by extending TConfig:

```pascal
unit MyConfig;

{$F+}  { Enable far calls for virtual methods }

interface

uses Config;

type
  PMyConfig = ^TMyConfig;

  TMyConfig = object(TConfig)
    { Custom fields }
    Difficulty: Byte;
    MusicVolume: Byte;
    PlayerName: String;

    constructor Init(const ConfigIniPath: String);
    procedure SetDefaults; virtual;
    procedure ParseKeyValue(const Key, Value: String); virtual;
    procedure WriteSettings(var F: Text); virtual;
  end;

implementation

uses StrUtil;

constructor TMyConfig.Init(const ConfigIniPath: String);
begin
  inherited Init(ConfigIniPath);
end;

procedure TMyConfig.SetDefaults;
begin
  inherited SetDefaults;  { Set engine defaults }
  Difficulty := 1;
  MusicVolume := 75;
  PlayerName := 'Player';
end;

procedure TMyConfig.ParseKeyValue(const Key, Value: String);
begin
  if Key = 'Difficulty' then
    Difficulty := StrToInt(Value)
  else if Key = 'MusicVolume' then
    MusicVolume := StrToInt(Value)
  else if Key = 'PlayerName' then
    PlayerName := Value
  else
    inherited ParseKeyValue(Key, Value);  { Handle engine keys }
end;

procedure TMyConfig.WriteSettings(var F: Text);
begin
  inherited WriteSettings(F);  { Write engine settings }
  WriteLn(F, '');
  WriteLn(F, '[Game]');
  WriteLn(F, 'Difficulty=', Difficulty);
  WriteLn(F, 'MusicVolume=', MusicVolume);
  WriteLn(F, 'PlayerName=', PlayerName);
end;

end.
```

**Usage:**
```pascal
var
  GameConfig: TMyConfig;
begin
  GameConfig.Init('CONFIG.INI');
  GameConfig.Load;

  WriteLn('Difficulty: ', GameConfig.Difficulty);
  WriteLn('Volume: ', GameConfig.MusicVolume);

  GameConfig.Done;
end;
```

## Integration with DGECORE

Pass config pointer to TGame.Init:

```pascal
var
  GameConfig: TConfig;
  Game: TMyGame;
begin
  GameConfig.Init('CONFIG.INI');
  Game.Init(@GameConfig, 'DATA\RES.XML');  { Pass pointer }
  Game.Start;  { Calls Config^.Load internally }
  Game.Run;
  Game.Done;
  GameConfig.Done;
end;
```

**Important:** The game does NOT own the config - caller must cleanup with `GameConfig.Done`.

## CONFIG.INI Format

```ini
; DOS Game Engine Configuration

[Sound]
SoundCard=2
SBPort=2
SBIRQ=5
SBDMA=1

[Input]
UseMouse=0
```

Extended configs can add custom sections:

```ini
; DOS Game Engine Configuration

[Sound]
SoundCard=2
SBPort=2
SBIRQ=5
SBDMA=1

[Input]
UseMouse=0

[Game]
Difficulty=2
MusicVolume=75
PlayerName=Alice
```

## Notes

- Created by SETUP utility
- DGECORE auto-loads via `Config^.Load` in `TGame.Start`
- Default values: SoundCard=None, SBPort=2 ($220), SBIRQ=5, SBDMA=1, UseMouse=0
- **Virtual methods** require `{$F+}` directive (far calls)
- **Extensible design** allows games to add custom settings without modifying base unit
- **Object ownership**: Caller owns TConfig, TGame only holds reference (PConfig)
