# game/ — Godot project rules

Read root `CLAUDE.md` first. Architecture: `docs/ARCHITECTURE.md`. Networking: `docs/NETWORK.md`.

## Engine
- Godot 4 stable, version pinned in root `CLAUDE.md`. Everyone uses the exact same version.
- Renderer: **Compatibility** (best Android support). 2D only.
- Project not scaffolded yet. When scaffolding, add to root `.gitignore`: `.godot/`, `*.apk`, `*.aab`, `export_credentials.cfg`, `*.keystore`.

## GDScript conventions
- Static typing everywhere: `var speed: float = 200.0`, `func move(dir: Vector2) -> void:`.
- snake_case files, functions, variables; PascalCase `class_name` and node names; UPPER_CASE constants.
- One script per scene root. Keep scripts under ~300 lines; split by responsibility.
- Signals for "something happened", direct calls for "do this". Global signals go in `autoload/events.gd`.
- No game rules on the client. Client code shows state and sends `request_*` intents.
- Tunable numbers (cooldowns, speeds, meter rates) live in `autoload/game_config.gd` as constants, not scattered literals.

## Scenes
- Every task is `scenes/tasks/<task_name>/<task_name>.tscn` extending `task_base.gd`.
- Every networked node spawned by the server goes through a `MultiplayerSpawner`.
- UI uses anchors/containers so it fits 16:9 to 20:9 phones.

## Running
- Editor: F5. Multi-instance test: Debug > Customize Run Instances (3 instances, one with `--server`).
- Headless server: `godot --headless --path game -- --server --port 7777`.

## Export (Android)
- Install Android build template and set SDK/JDK paths in Editor Settings (one-time per machine).
- Export preset name: `Android`. Permissions: `INTERNET`, `CAMERA`, `ACCESS_NETWORK_STATE`, `ACCESS_WIFI_STATE`, `CHANGE_WIFI_MULTICAST_STATE` (LAN discovery).
- Debug keystore is fine for the hackathon. Never commit keystores.
- Command line: `godot --headless --path game --export-debug "Android" ../build/clean_city.apk`.

## Testing
- Logic that can break silently (meter math, vote counting, win conditions, cooldowns) gets a unit test with GUT or gdUnit4 (pick one in week 1, note it in `PROGRESS.md`).
- Everything else: run it on a real phone and say what you saw.
