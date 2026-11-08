# Compatibility notes

## Scripting API

Unchanged from 2.1.x: `mysql.inc`, native names/signatures, callbacks
`OnMysqlQuery`, `OnMysqlQueryArray`, `OnMysqlError`.

## Character sets

- Storage may be `utf8mb4`.
- Plugin connection defaults to **`latin1`** on `mysql_connect` (SA-MP/Pawn wire encoding).
- Override: `mysql_set_character_set("...")`.

## Interop tested

- Streamer **2.9.6**
- CrashDetect **4.22**
- sscanf: short verification still recommended

## Scope

Legacy StrickenKid API for getting old gamemodes running on modern Windows
plugin stacks. Prefer BlueG/maddinat0r MySQL **R41** (SA-MP) / **R42** (open.mp)
for new or high-player setups.
