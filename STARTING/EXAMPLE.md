
# Example Codes

## VGA Graphics
```pascal
uses VGA, PCX;

var
  FrameBuffer: PFrameBuffer;
  TestImage: TImage;
  Palette: TPalette;

begin
  { Load image with palette }
  LoadPCXWithPalette('DATA\TEST.PCX', TestImage, Palette);

  InitVGA;
  SetPalette(Palette);

  { Render to framebuffer }
  FrameBuffer := CreateFrameBuffer;
  PutImage(0, 0, @TestImage, FrameBuffer);
  RenderFrameBuffer(FrameBuffer);
  ReadLn;

  FreeFrameBuffer(FrameBuffer);
  FreeImage(TestImage);
  DoneVGA;
end.
```

## Playing Music
```pascal
uses PlayHSC, Keyboard;

var
  Music: HSC_Obj;

begin
  InitKeyboard;
  Music.Init(0);  { Auto-detect AdLib at port 388h }
  if Music.LoadFile('DATA\FANTASY.HSC') then
  begin
    Music.Start; { Music is playing at IRQ 0 }
    while not IsKeyDown(Key_Escape) do
    begin
      { ... your game loop ... }
      ClearKeyPressed;
    end;
    Music.Done;  { CRITICAL: Unhook interrupt! }
  end;
  DoneKeyboard;
end.
```

## Sound Effects with XMS
```pascal
uses SBDSP, SndBank, XMS;

var
  Bank: TSoundBank;
  ExplosionID: Integer;

begin
  { Initialize Sound Blaster }
  ResetDSP(2, 5, 1, 0);  { Port $220, IRQ 5, DMA 1 }

  { Initialize sound bank }
  Bank.Init;

  { Load sounds into XMS at startup }
  ExplosionID := Bank.LoadSound('DATA\EXPLODE.VOC');

  { Play on demand - no disk I/O! }
  Bank.PlaySound(ExplosionID);

  { Cleanup }
  Bank.Done;
  UninstallHandler;
end.
```

## Game Loop with Delta Timing
```pascal
uses VGA, Keyboard, RTCTimer;

var
  Running: Boolean;
  LastTime, CurrentTime, DeltaTime: Real;
  PlayerX, PlayerY: Real;

begin
  InitVGA;
  InitKeyboard;
  InitRTC(1024);  { 1024 Hz timer }

  LastTime := GetTimeSeconds;
  Running := True;

  while Running do
  begin
    { Calculate delta time }
    CurrentTime := GetTimeSeconds;
    DeltaTime := CurrentTime - LastTime;
    LastTime := CurrentTime;

    { Frame-rate independent movement }
    if IsKeyDown(Key_Right) then
      PlayerX := PlayerX + 100.0 * DeltaTime;  { 100 pixels/sec }

    if IsKeyPressed(Key_Escape) then
      Running := False;

    { Render frame... }

    ClearKeyPressed;  { MUST call at end of loop }
  end;

  { CRITICAL: Clean up all interrupts }
  DoneRTC;
  DoneKeyboard;
  DoneVGA;
end.
```
