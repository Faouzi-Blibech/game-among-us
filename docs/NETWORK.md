# Networking

Owner: AI dev 1. Changes to the RPC list need a line in `PROGRESS.md` > Decisions.

## Authority model
- Server (peer id 1) owns **all** game state: roles, positions (authoritative), tasks, meter, sabotages, slime, votes.
- Clients send **intents** (`request_*`). Server validates, applies, then broadcasts results.
- Clients never learn other players' roles until reveal (ejection or game end). Do not sync role to everyone.
- Movement: client sends input direction at 20 Hz; server moves the body and syncs positions back with `MultiplayerSynchronizer` (client-side prediction only if lag is visible in playtests).

## Transports (`Net` autoload)
| Mode | Peer | Port |
|---|---|---|
| Solo | `OfflineMultiplayerPeer` (default) | — |
| LAN | `ENetMultiplayerPeer.create_server(7777, 10)` / `create_client(ip, 7777)` | UDP 7777 |
| Online | `WebSocketMultiplayerPeer.create_server(7778)` / `create_client("ws://host:7778")` | TCP 7778 |

LAN discovery (nice-to-have, week 3): server broadcasts on UDP 7779 every 1 s with `{name, players, port}`; join screen lists hosts. Until then: type the IP shown on the host screen.

## Lobby flow
1. Host creates lobby -> phase `LOBBY`. Host screen shows IP and a room code.
2. Clients join -> `request_join(name, color)` -> server assigns slot, broadcasts `lobby_update(players)`.
3. Host taps Start -> server fills empty seats with bots (up to chosen count), assigns roles, sends each peer `role_reveal(role, teammates_if_polluter)` privately.
4. Phase `PLAYING`.

## RPC list (v1)
Client -> server (`@rpc("any_peer", "call_remote", "reliable")` unless noted):
| RPC | Args | Server checks |
|---|---|---|
| `request_join` | name, color | lobby phase, slot free |
| `send_input` (unreliable_ordered) | dir: Vector2 | player alive in PLAYING |
| `request_complete_task` | task_id | assigned, in range, not slimed |
| `request_sabotage` | sabotage_id | is polluter, cooldown ready |
| `request_spawn_pollution` | pos | is polluter, cooldown, valid tile |
| `request_slime` | target_id | is polluter, in range, cooldown |
| `request_clean` | target_id or object_id | in range, is cleaner |
| `request_fix` | sabotage_id, step | in range |
| `request_report` | target_id | target slimed, in range |
| `request_emergency` | — | button unused, in town hall |
| `request_vote` | target_id or -1 (skip) | MEETING phase, not voted |
| `request_phrase` | phrase_id, arg_ids | MEETING phase |

Server -> clients (`@rpc("authority", "call_local", "reliable")`):
| RPC | Args |
|---|---|
| `lobby_update` | players[] |
| `role_reveal` | role, polluter_teammates[] (sent to one peer) |
| `phase_changed` | phase, timer |
| `meter_update` (unreliable, 4 Hz) | value 0-100 |
| `task_bar_update` | value 0-1 |
| `sabotage_started` / `sabotage_fixed` | sabotage_id |
| `pollution_spawned` / `pollution_cleaned` | object_id, pos, type |
| `player_slimed` / `player_freed` | player_id |
| `meeting_started` | caller_id, reason, body_id |
| `phrase_posted` | player_id, phrase_id, arg_ids |
| `vote_result` | ejected_id, was_polluter |
| `game_over` | winner, stats, roles_all |

## Spawning
Players and pollution objects are spawned by the server through `MultiplayerSpawner`. Clients never `add_child` networked nodes themselves.

## Testing networking
- Editor: Debug > Customize Run Instances: run 3 instances (1 with `--server`).
- Real test every week: laptop host + 2 phones on the team hotspot.
- Simulate bad Wi-Fi once before the event (phone far from hotspot); game must recover or show a clear "reconnecting" state.
