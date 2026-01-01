# DOS Game Engine

A professional game development framework for MS-DOS, written in Turbo Pascal 7.0. Build retro games with VGA graphics (320×200, 256 colors), Sound Blaster audio, Adlib music, and smooth 60 FPS gameplay.

## Features

- **VGA Mode 13h Graphics** - Double-buffered rendering, palette effects, sprite animation
- **Sound Blaster DSP** - DMA-based PCM playback with XMS sound banking
- **Adlib/OPL2 Music** - HSC tracker music player with IRQ0 synchronization
- **Input Handling** - Low-level keyboard (INT 9h) and mouse (INT 33h) drivers
- **Tilemap System** - TMX/Tiled Map Editor integration with layer rendering
- **Asset Loaders** - PCX images (Aseprite-compatible), VOC sounds, XML configuration
- **Delta-time Animation** - Frame-rate independent sprite animation system
- **RTC Timer** - High-precision timing (up to 8192 Hz) via IRQ8
- **XMS Memory** - Extended memory support for storing large assets

## System Requirements

**Development:**
- Turbo Pascal 7.0 (or compatible compiler)
- DOS or DOSBox

**Runtime:**
- MS-DOS 5.0 or higher
- VGA graphics card
- 640KB RAM minimum
- Sound Blaster compatible sound card (optional)
- HIMEM.SYS for XMS support (recommended)
- MOUSE.COM or MOUSE.SYS for mouse input (optional)

## Platform Support

Tested and working on:
- Real MS-DOS hardware (286-Pentium)
- DOSBox (0.74+)
- DOSBox-X
- 86Box emulator
- FreeDOS

```{toctree}
:maxdepth: 1
:caption: Contents

STARTING/BUILD
STARTING/CREATE
STARTING/EXAMPLE
CORE/UNITS_REFERENCE
CORE/VGA
CORE/SPRITE
CORE/TILEMAP
AUDIO/SBDSP
AUDIO/SNDBANK
AUDIO/PLAYHSC
AUDIO/HSC
INPUT/KEYBOARD
INPUT/MOUSE
FORMATS/PCX
FORMATS/BMP
FORMATS/MINIXML
UTILS/CONFIG
UTILS/TEXTUI
UTILS/RTCTIMER
UTILS/XMS
UTILS/STRUTIL
UTILS/LINKLIST
UTILS/STRMAP
UTILS/GENTYPES
UTILS/LOGGER
ADVANCED/HISCORE
ADVANCED/GAMEUNIT
ADVANCED/RESMAN
ADVANCED/VGAFONT
ADVANCED/VGAPRINT
ADVANCED/VGAUI
ADVANCED/DRECT
ADVANCED/MD5
DEVEL/ISSUES
DEVEL/GITATTRIBUTES
