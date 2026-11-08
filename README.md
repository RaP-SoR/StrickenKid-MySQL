# StrickenKid-MySQL

SA-MP MySQL plugin **v2.2** (legacy API).

Rebuilt with an updated **SA-MP plugin SDK**, **MariaDB Connector/C**, and modern
**Windows + Linux** toolchains (CMake / MSVC Win32 / Linux i386).

Copyright (c) 2011 **StrickenKid**  
Copyright (c) 2026 **RaPSoR**

## Important (not a production default)

This plugin is **not** intended for new projects or long-running production servers.
Its purpose is to get **legacy gamemodes** that still use the old StrickenKid API
running again on modern Windows builds (and typical plugin ABI conflicts).

**Recommended for new or long-term setups:**

- SA-MP: [MySQL plugin by BlueG / maddinat0r](https://github.com/pBlueG/SA-MP-MySQL) **R41**
- open.mp: **R42** (or the current open.mp-compatible release of that line)

This repo intentionally keeps the **old scripting API** (`mysql.inc` / callbacks) so existing mods can start without a full refactor.

## What changed in v2.2

- SA-MP plugin SDK: [maddinat0r/samp-plugin-sdk](https://github.com/maddinat0r/samp-plugin-sdk)
- Client library: **MariaDB Connector/C 3.4.5** (statically linked)
- Build system: **CMake** for **Windows Win32** (VS 2022/2026) and **Linux i386** (CI + local `-m32`)
- Connection charset: plugin sets **`latin1` on `mysql_connect`** by default (SA-MP/Pawn wire encoding for umlauts). You can still change it from the script with `mysql_set_character_set(...)`.
- Pawn API unchanged from 2.1.x

## Compatibility tested with

These are the last widely used **stable SA-MP** releases we verified this build with
(load/run OK together with this MySQL plugin). Newer open.mp-only plugin lines are
intentionally out of scope here — legacy gamemodes usually cannot adopt them without
a large rewrite.

| Plugin | Tested version | Release |
| --- | --- | --- |
| Streamer | **v2.9.6** — OK | https://github.com/samp-incognito/samp-streamer-plugin/releases/tag/v2.9.6 |
| CrashDetect | **v4.22** — OK | https://github.com/Y-Less/samp-plugin-crashdetect/releases/tag/v4.22 |
| sscanf | **v2.13.8** — OK | https://github.com/Y-Less/sscanf/releases/tag/v2.13.8 |

That stack is common on legacy servers and a frequent source of ABI/runtime conflicts with ancient plugin builds.

## Build (Windows, 32-bit)

SA-MP plugins must be **Win32 (x86)**.

```powershell
git submodule update --init --recursive
# x86 Native Tools / vcvarsall x86
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

Or: `cmake -S . -B build -G "Visual Studio 17 2022" -A Win32`

Output: `build/bin/mysql.dll`

## Build (Linux, 32-bit)

Locally or via GitHub Actions (`.github/workflows/build.yml`: Ubuntu 20.04/22.04/24.04, Debian 12/13).  
Prefer the **ubuntu-20.04** artifact for wider runtime compatibility.

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_FLAGS=-m32 -DCMAKE_CXX_FLAGS=-m32
cmake --build build
```

Output: `build/bin/mysql.so`

## Install

1. Copy `mysql.dll` / `mysql.so` into `plugins/`
2. Copy `mysql.inc` into your Pawn include path
3. In `server.cfg`: `plugins mysql` (keep testing load order with Streamer/CrashDetect)

## License

GPL (see `LICENSE`). Original work by StrickenKid; maintenance/rebuild by **RaPSoR**.
