# Architecture

## One build, three modes
| Mode | Who hosts | Transport | Players |
|---|---|---|---|
| **Solo / demo** | the phone itself (local server in-process) | none (`OfflineMultiplayerPeer`) | 1 human + bots |
| **LAN** | laptop (headless) or one phone, on our own hotspot | ENet (UDP 7777) | humans + bots fill empty seats |
| **Online** | VPS running the same headless build | WebSocket (`WebSocketMultiplayerPeer`, port 7778) | humans + bots |

All modes run the **same server logic**. The server is just the peer with id 1. In solo mode the phone is peer 1 and runs both server and client code.

```
 phone (client) --ENet/WS-->  Godot server (headless, laptop/VPS)  --HTTP-->  ai_service (Python, same machine)
 phone (client) --ENet/WS-->        |  owns all game state                     /classify  /director  /telemetry
 bots           -- in-process ->    |  runs bot brains
```

## Godot project layout (`game/`)
```
game/
  project.godot
  autoload/
    net.gd          # Net: host/join/leave, transport selection, peer list
    game_state.gd   # GameState: roles, meter, tasks, sabotages, phase machine (server writes, clients read)
    events.gd       # Events: global signals only, no logic
    ai_client.gd    # AiClient: HTTP calls to ai_service with timeout + fallback
    game_config.gd  # GameConfig: all tunable constants (cooldowns, speeds, meter rates)
  scenes/
    main_menu/      # Play solo / Host / Join / Settings
    lobby/
    map/            # map.tscn, rooms, nav mesh, interactables
    player/         # player.tscn (shared by humans and bots), input sources
    tasks/          # one folder per task + task_base.gd
      camera_sort/
    sabotage/
    meeting/
    ui/             # HUD, joystick, meter bar, task list, end screen
  ai/
    bot_brain.gd    # utility AI, produces the same input as the touch UI
    director.gd     # pacing; local fallback when ai_service is down
  assets/           # designer-owned; see ART_GUIDE.md
  i18n/             # translations CSV
```

## Autoloads (singletons)
- `Net` — only place that touches `multiplayer.multiplayer_peer`. API: `host_solo()`, `host_lan(port)`, `host_online(port)`, `join(address, port, transport)`, `leave()`. Emits `Events.peer_joined/peer_left`.
- `GameState` — the single source of truth. Server mutates it; clients receive updates via RPC (see `NETWORK.md`). Phases: `LOBBY -> ROLE_REVEAL -> PLAYING -> MEETING -> PLAYING ... -> END`.
- `Events` — signal bus. Add signals here, never logic.
- `AiClient` — every call: 2 s timeout, returns a fallback result on failure, never awaits on the main gameplay path.

## Player = body + input source
`player.tscn` is a `CharacterBody2D`. It reads movement and actions from an **input source**:
- `TouchInput` (human on this device)
- `BotInput` (driven by `ai/bot_brain.gd`, server only)
- `RemoteInput` (on the server, fed by client intent RPCs)

This is why bots need no special game rules: they press the same virtual buttons.

## Tasks
- `task_base.gd` defines: `task_id`, `room`, `duration_hint`, `start(player)`, signal `completed(player, materials)`.
- Task UI runs on the client. On finish the client sends `request_complete_task(task_id)`; the server validates (player is in range, task assigned, not slimed) and updates `GameState`.
- Bots complete tasks by standing in range for `duration_hint` seconds (no UI).

## Fallback rules (the game never blocks on AI)
| Feature | Normal | Fallback |
|---|---|---|
| Camera sort | camera -> `/classify` | manual sort of a shown image |
| Director | `/director` every 10 s | `ai/director.gd` fixed schedule |
| Telemetry | POST `/telemetry` | drop silently |
| Camera permission denied | — | same as manual sort |

## Ownership (who edits what)
See root `CLAUDE.md` > Ownership. Shared files with more than one owner: `game_state.gd` (AI dev 1 owns, others request changes), `events.gd` (anyone may add a signal, note it in `PROGRESS.md`).

## Performance budget (mid-range Android)
- 60 FPS target, 30 FPS minimum, 10 players + up to 40 pollution objects.
- Use `Compatibility` renderer (best Android support). 2D only.
- Network tick: 20 Hz position sync.
