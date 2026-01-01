# Building from Source

Start up your DOS console, go to the engine's folder. You should have a `tpc.exe` (Turbo Pascal Compile executable) in your `%PATH%`.

Compile the tests:

**Automated (recommended):**
```batch
cd TESTS
CBMPTEST.BAT    # BMP image loader test
CVGATEST.BAT    # VGA graphics test
CSNDTEST.BAT    # Sound bank test
CSPRTEST.BAT    # Sprite animation test
CTMXTEST.BAT    # TMX tilemap scrolling test
CXMLTEST.BAT    # XML parser test
```

If you run `CTMXTEST` for example, you should have a `TMXTEST.EXE` at the end.

**Manual compilation:**
```batch
cd UNITS
tpc VGA.PAS
tpc PCX.PAS
tpc SBDSP.PAS
# ... compile other units

cd ..\TESTS
tpc -U..\UNITS VGATEST.PAS
```
