# Compatibility notes

## Scripting API

Unchanged from 2.1.x: `mysql.inc`, native names/signatures, and callbacks
`OnMysqlQuery`, `OnMysqlQueryArray`, `OnMysqlError`.

## Character sets

- Table/database storage may be `utf8mb4`.
- The **plugin** sets **`latin1` on `mysql_connect`** by default (SA-MP/Pawn wire encoding).
- Scripts can still override with `mysql_set_character_set("...")`.

## Interop tested (SA-MP stable releases)

Verified OK with this MySQL build (legacy SA-MP stack; newer open.mp-only plugin
lines are out of scope):

- Streamer **v2.9.6** — https://github.com/samp-incognito/samp-streamer-plugin/releases/tag/v2.9.6
- CrashDetect **v4.22** — https://github.com/Y-Less/samp-plugin-crashdetect/releases/tag/v4.22
- sscanf **v2.13.8** — https://github.com/Y-Less/sscanf/releases/tag/v2.13.8

## Scope

Legacy StrickenKid API for getting old gamemodes running on modern Windows
plugin stacks. Prefer BlueG/maddinat0r MySQL **R41** (SA-MP) / **R42** (open.mp)
for new or high-player setups.

Maintenance/rebuild: **RaPSoR** (2026).
