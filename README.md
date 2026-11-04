# x64dbg-extracmds

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE) [![For x64dbg](https://img.shields.io/badge/for-x64dbg-444.svg)](https://github.com/x64dbg/x64dbg) [![Platform: Windows](https://img.shields.io/badge/Platform-Windows%20x86%2Fx64-0078D6.svg)](#)

**ExtraCmds** is a small [x64dbg](https://github.com/x64dbg/x64dbg) plugin that adds a handful of convenience commands, usable from the command bar, scripts, `headless.exe` and other plugins:

- **`.init`** — launch & debug an executable, or open a memory dump in the viewer;
- **`.save`** — one command for the database, a minidump, or a memory region;
- **`scylla_hide.enable` / `.disable` / `.status`** — switch [ScyllaHide](https://github.com/x64dbg/ScyllaHide) profiles without restarting x64dbg.

> **Scope.** This repository is **only** the classic ExtraCmds commands. It is deliberately self-contained — no external tools, no Qt, builds for both x86 and x64 — so it's reusable by any x64dbg user. (It's carved out of a larger personal toolkit; nothing else from that toolkit is here.)

---

## Commands

| Command | Effect |
|---|---|
| `.init file.exe[, arguments[, dir]]` | launches and debugs the program (like `init`) |
| `.init file.dmp` | opens the dump in the **minidump viewer** (see note below) |
| `.save` | saves the program database — like `dbsave` |
| `.save file.dmp` | writes a full process minidump — like `minidump` |
| `.save file.dd64` | saves the database to that file — like `dbsave file` |
| `.save file, address` | saves the memory region containing the address (`mem.base` / `mem.size`) |
| `.save file, address, size` | saves a memory range — like `savedata` (size in hex: `100` = 256 bytes, `.256` decimal) |
| `scylla_hide.status` | shows whether ScyllaHide is active, its profile, and the available profiles |
| `scylla_hide.enable [profile]` | enables ScyllaHide: the given profile (exact name or prefix, e.g. `vmprotect`, `basic`), else the last active one, else "Basic" |
| `scylla_hide.disable` | disables ScyllaHide (switches to the "Disabled" profile) |

```
scylla_hide.enable basic
.init "C:\target.exe"
.save "C:\out\target.dmp"
.save "C:\out\stack.bin", rsp
.init "C:\out\target.dmp"
```

### How it works

- **`.save`** picks `dbsave`, `minidump` or `savedata` from the extension and the number of arguments, so you don't have to remember which built-in command to call.
- **`scylla_hide.*`** edits `plugins\scylla_hide.ini` (`CurrentProfile`) then reloads the ScyllaHide plugin (`plugunload` + `plugload`), so the profile applies immediately without restarting x64dbg. Protections are injected at program start/attach — if a program is already being debugged, restart it to pick up the new profile. Requires the ScyllaHide plugin to be installed.
- **`.init file.dmp`** opens `x64\minidump.exe` — the standalone memory-dump viewer (memory map, disassembly, hex view, threads, registers). A debugger can't "load" a dump (no execution/stepping/breakpoints), so this is a viewer, not a debug session. **Note:** `minidump.exe` is not part of stock x64dbg; if it isn't next to the debugger, this sub-command just logs that the viewer wasn't found — the rest of the plugin works regardless.

---

## Install

Drop the built plugin into your x64dbg `plugins` folder:

- 64-bit: `release\x64\plugins\ExtraCmds.dp64`
- 32-bit: `release\x32\plugins\ExtraCmds.dp32`

Restart x64dbg (or load it from *Plugins*). The commands are then available everywhere.

## Build

Requires Visual Studio 2022 (C++), CMake ≥ 3.15, Ninja, and the x64dbg **plugin SDK** (the `pluginsdk` folder of an x64dbg release).

```bat
:: 64-bit, in an "x64 Native Tools" terminal
cmake -B build-x64 -G Ninja -DCMAKE_BUILD_TYPE=Release -DX64DBG_SDK=<path to pluginsdk>
cmake --build build-x64

:: 32-bit, in an "x86 Native Tools" terminal
cmake -B build-x32 -G Ninja -DCMAKE_BUILD_TYPE=Release -DX64DBG_SDK=<path to pluginsdk>
cmake --build build-x32
```

The output is `ExtraCmds.dp64` / `ExtraCmds.dp32`. If x64dbg was installed with winget, `X64DBG_SDK` is auto-detected and can be omitted.

---

## License & credits

GPL-3.0, same as [x64dbg](https://github.com/x64dbg/x64dbg) (by mrexodia and contributors). ScyllaHide is at [x64dbg/ScyllaHide](https://github.com/x64dbg/ScyllaHide). Not affiliated with or endorsed by the x64dbg team.
