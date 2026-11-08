# AGENTS.md

MTA:SA (Multi Theft Auto) server distribution — "SAxVCxLC Stories" RPG/RP gamemode. Not a buildable codebase: it is a ready-to-run Windows server (`MTA Server.exe`) plus Lua resources. There is no build, lint, or test tooling.

## Layout

- `MTA Server.exe` + DLLs in root: run the server. Windows-only, no build/test/lint tooling.
- `mods/deathmatch/`: server config and data (`mtaserver.conf`, `acl.xml`, `local.conf`, `internal.db`, `registry.db`).
- `mods/deathmatch/resources/`: all Lua resources. `228/` is the main gamemode (entry: `meta.xml` → `shared.lua`, `script.lua` (server, ~15.7k lines), `client.lua` (client, ~6.4k lines)). `shared.lua` holds skin/faction data (`SkinData`, `VacancyDATA`, `BandRangs`); `serverdata/*.xml` + `save.txt` are persistent game state.
- Bracketed dirs (`[admin]`, `[gameplay]`, …) are stock resource groups shipped mostly as `.zip`s (loaded on demand, e.g. by admin/mapmanager) — they are NOT listed in `mtaserver.conf`.
- `mods/deathmatch/resource-cache/` and `resources/editor_dump/` are runtime-generated — do not edit.
- `dumps/private/` is data, not code (encrypted; copied to `dumps/public` on startup). `serverdata_backup_*` dirs are data backups.

## Setup gotcha

- **Required setup step (README.md):** `mods/deathmatch/resources/228/script.lua` line 1 — set `local SERVER_IP = ""` to your server IP before running. Clients read it via the `IP` element data. It is usually already filled in with the live server IP — leave it as is unless told otherwise.

## Conventions

- Lua 5.1 (MTA's Lua VM), OOP disabled (`<oop>false</oop>` in `228/meta.xml`) — no `.` OOP syntax on elements; use MTA functions and `setElementData/getElementData`.
- Resource files must be listed in the resource's `meta.xml` (`<script>`, `<file download="true">`) or they won't load/download to clients. Adding a file means editing `meta.xml`.
- Gamemode-level exports in `228/meta.xml` (e.g. `CreateVehicle`, `AddPlayerMoney`) are called cross-resource via `call()`; cross-resource calls like `exports["vehicle_node"]:GetVehicleNodes()` require the function to be exported in the source resource's `meta.xml`.
- Server startup resource list is at the bottom of `mods/deathmatch/mtaserver.conf` (~lines 340–357); `228` depends on other resources (`vehicle_node`, `chat`, `ai`, …) being listed there and started first. `local.conf` is the stock template; `editor*.conf/xml` are for the map editor.
- Bracketed dirs are organizational only: MTA sees all resources as a flat list — never reference `[...]` paths in scripts/configs, and a resource existing twice anywhere in the hierarchy will fail to load (see `resources/Directory layout readme.txt`).
- Content is largely Russian-language (UI strings, .po files under `228/lang/`). Keep that when editing user-facing strings.
- Some resources ship `.old` variants of scripts — these are backups, not loaded.

## Gotchas

- Changes to client scripts take effect only after players redownload the resource (restart the resource on the server).
- No Git LFS, no CI; binary assets (`.col`/`.dff`/`.txd` models, stock `.zip` archives) are committed directly. Don't try to build or package them. Custom model additions for `228` live under `resources/228/models/` and must be listed in its `meta.xml`.
- Other runtime-produced dirs: `logs/`, `databases/` (SQLite), `backups/` — don't edit.
- `git status` normally shows hundreds of dirty files under `resource-cache/`, `databases/`, `logs/`, `registry.db` etc. — that's server runtime churn, not real changes. Never commit them; stage only the resource files you actually edited.
- Lua syntax can be checked locally with Lua 5.4 (`lua -e "local f,e=loadfile('<file>'); print(f and 'SYNTAX OK' or e)"`) — same core syntax family as MTA's Lua 5.1 for load-time checks (no MTA API available).

## Git

- Remotes: `origin` = https://github.com/zoan96/MTA-Stories (test/AI-modernization mirror — **all commits and pushes go here automatically**), `ax705` = https://github.com/alexaxel705/MTA-Stories (original project by the author — **push there ONLY after explicit user approval**). Branch `main`.
