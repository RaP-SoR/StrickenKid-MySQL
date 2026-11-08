# StrickenKid-MySQL

SA-MP MySQL plugin **v2.2** (legacy API), modernisiert für aktuelle Windows-Toolchains.

Copyright (c) 2011 **StrickenKid**  
Copyright (c) **Repsol**

## Wichtiger Hinweis (kein Production-Default)

Dieses Plugin ist **nicht** für neue Projekte oder produktiven Dauerbetrieb mit vielen Spielern gedacht.

- Ziel: **Legacy-Gamemodes** mit der alten StrickenKid-API wieder zum Laufen bringen (moderne Windows-Builds, Plugin-Konflikte).
- Für wenige Spieler / Migration / Wartung: oft ausreichend.
- Ab ca. **20–30 Spielern** und starker DB-Last: ungeeignet (alte Query-/Threading-Architektur).

**Empfehlung für neue oder langfristige Setups:**

- SA-MP: [MySQL plugin by BlueG / maddinat0r](https://github.com/pBlueG/SA-MP-MySQL) **R41**
- open.mp: **R42** (bzw. die aktuelle open.mp-taugliche Release derselben Linie)

Dieses Repo hält bewusst die **alte Scripting-API** (`mysql.inc` / Callbacks), damit bestehende Mods ohne Refactor starten.

## Was in v2.2 geändert wurde

- SA-MP Plugin-SDK: [maddinat0r/samp-plugin-sdk](https://github.com/maddinat0r/samp-plugin-sdk)
- Client: **MariaDB Connector/C 3.4.5** (statisch)
- Build: **CMake** (VS 2022/2026 Win32, Linux i386 via CI)
- Connect-Charset-Default: **latin1** (SA-MP-Umlaute; Override: `mysql_set_character_set`)
- Pawn-API unverändert zu 2.1.x

## Getestet mit (Kompatibilität)

Offizielle letzte Versionen, mit denen dieser Build geprüft wurde:

| Plugin | Version | Status |
| --- | --- | --- |
| Streamer | **2.9.6** | getestet |
| CrashDetect | **4.22** | getestet |
| sscanf | — | kurzer Test noch ausstehend / empfohlen selbst zu prüfen |

Diese Kombination ist typisch für Legacy-Server und oft Ursache für ABI-/Runtime-Konflikte mit uralten Plugin-Builds.

## Build (Windows, 32-bit)

SA-MP-Plugins sind **Win32 (x86)**.

```powershell
git submodule update --init --recursive
# x86 Native Tools / vcvarsall x86
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

Oder: `cmake -S . -B build -G "Visual Studio 17 2022" -A Win32`

Output: `build/bin/mysql.dll`

## Build (Linux, 32-bit)

Lokal oder über GitHub Actions (`.github/workflows/build.yml`: Ubuntu 20.04/22.04/24.04, Debian 12/13).  
Für breite Laufzeit-Kompatibilität das **ubuntu-20.04**-Artifact bevorzugen.

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_FLAGS=-m32 -DCMAKE_CXX_FLAGS=-m32
cmake --build build
```

Output: `build/bin/mysql.so`

## Installation

1. `mysql.dll` / `mysql.so` nach `plugins/`
2. `mysql.inc` in den Pawn-Include-Pfad
3. In `server.cfg`: `plugins mysql` (Reihenfolge mit Streamer/CrashDetect wie gewohnt testen)

## Lizenz

GPL (siehe `LICENSE`). Originalwerk von StrickenKid; Wartung/Rebuild: Repsol.
