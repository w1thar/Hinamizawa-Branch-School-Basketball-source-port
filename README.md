# Hinamizawa Branch School Basketball — source port

A source-level reconstruction of the fan-made *Higurashi no Naku Koro ni* 3-on-3 basketball
minigame (`basket_v101.exe`, 2004), rebuilt from the original binary with Ghidra and rewritten
as portable C++.

- Plays the same as the original: title/attract sequence, Normal / Club (部活) / Practice (練習)
  modes, team selection, tip-off, matches with halves, results and ending.
- **Self-contained:** all game data is packed into the executable at build time, so the result is
  a single `hbasket.exe` with no other files needed.
- **Windowed or fullscreen** (the original was fullscreen-only).
- Direct3D 7 / DirectSound / DirectDraw replaced by a small platform layer (OpenGL 1.1, WinMM,
  GDI), designed so other platforms only need a new backend.

## Building

You need **CMake 3.16+** and a C++17 compiler. No other dependencies, no scripting languages.

### Visual Studio 2019/2022

```
cmake -S . -B build
cmake --build build --config Release
```
Output: `build/Release/hbasket.exe`. You can also open the folder directly in Visual Studio
(*File → Open → Folder*) and build from there.

### MinGW-w64 (GCC)

```
mingw32-make
```
Output: `build/hbasket.exe` (statically linked, no MinGW DLLs needed).

Without the Makefile:
```
cmake -S . -B build -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

### Prebuilt

Every push builds on GitHub Actions; the exe can be downloaded from the run's *Artifacts*.

Only the Windows backend exists so far, so building on Linux/macOS stops with a message at
configure time.

## Playing

| Key | Action |
|---|---|
| Arrow keys / numpad 8 4 6 2 | Move / turn, menu selection |
| X | Hold to keep the ball, release to pass |
| Z | Hold to charge a loop pass / loop shot, release to throw; near the hoop while running: jump shot |
| X / Z / C / Space / Enter | Confirm in menus |
| Alt+Enter | Toggle fullscreen |
| Esc | Quit (as in the original) |

The coloured cone under your player shows what Z will do: blue = loop pass, green = loop shot
(the gauge marks the ideal power), red = jump shot.

Command line options:

| Option | |
|---|---|
| `-fullscreen` | Start in fullscreen |
| `-window` | Start windowed (default) |
| `-scale N` | Window size N × 640×480 (default: largest that fits) |
| `-novsync` | Don't wait for vertical sync |

## Project layout

```
src/core/        math (D3DX-era helpers), in-memory stdio replacement, asset archive reader
src/render/      fixed-function renderer with Direct3D 7 semantics (OpenGL backend)
src/engine/      textures, sound mixer + MP3 music
src/game/        the game itself — one file per subsystem of the original
src/platform/    platform layer; win32/ is the Windows backend
assets/          game data, packed into the exe at build time
third_party/     dr_mp3 (public domain MP3 decoder)
tools/           build tool (pakgen) and the reverse-engineering scripts
reference/       Ghidra decompile/asm dumps of the original binary (for reference only)
```

Comments in `src/game/` give the original function address for every function
(e.g. `State6 0x40e400`), so behaviour can be checked against `reference/`.

### Regenerating the reference dumps

Needs Ghidra 12+ and the original `basket_v101.exe`:
```
powershell -File tools/ghidra/run.ps1 -Ghidra <ghidra install dir> -Exe <path to basket_v101.exe>
```
`tools/ghidra/types.h` and `tools/ghidra/symbols.txt` hold the recovered structures and names.

## Differences from the original

- Runs windowed by default; the 640×480 image is scaled with black bars to keep 4:3.
- Text is drawn with anti-aliased MS Gothic (the original used GDI's default rendering of a
  non-existent font name, which fell back to the system Japanese font).
- A single frame advances the game by at most 0.1 s, so dragging the window doesn't make
  everything jump ahead.
- Textures keep full 32-bit colour instead of the original's 16-bit surfaces.

Japanese text needs a Japanese font on the system (MS Gothic, or any font Windows substitutes
for it). Without one, menus show placeholder glyphs.

## Credits

- Original game: **ひぐ中 (YnSoft)** — 「雛見沢分校バスケット大会」 for *ひぐらしのなく頃に*.
- *Higurashi no Naku Koro ni*, its characters, music and sound effects: **07th Expansion / 竜騎士07**.
  The trial version these files come from states that redistributing its data is not prohibited
  (「データの転載、頒布はこれを禁止しません」).
- MP3 decoding: [dr_mp3](https://github.com/mackron/dr_libs) by David Reid (public domain / MIT-0).

This is a non-commercial fan preservation project. The game data in `assets/` belongs to its
respective authors.
