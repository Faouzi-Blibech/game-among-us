# AI features

Three features, in priority order. Each must degrade gracefully (see `ARCHITECTURE.md` > Fallback rules).

| # | Feature | Owner | Runs where | Why judges care |
|---|---|---|---|---|
| 1 | Bot players | AI dev 2 | Godot server (GDScript) | the game works with 1 judge; full lobbies always |
| 2 | Camera waste sorting | AI dev 3 | phone camera -> `ai_service` | real-world recycling in a game; strongest demo moment |
| 3 | Pollution director | AI dev 2 (game) + AI dev 3 (service) | `ai_service`, fallback in Godot | rounds stay tense for new and good players |

Later, only if v1 is solid by Nov 16: LLM-driven bot meeting talk.

---

## 1. Bot players (`game/ai/bot_brain.gd`)
A bot is a normal `player.tscn` with `BotInput`. It runs on the server only.

**Approach: utility AI** (score every action, pick the best, re-evaluate every 0.5 s). No RL in v1: judges cannot see the difference and it will not converge in time.

Cleaner bot actions and scoring inputs:
- Go to nearest unfinished task (distance, task count left)
- Fix active sabotage (meter level, distance, critical flag)
- Free a slimed player / report it (seen, distance)
- Clean pollution object (distance, meter)
- Wander near others (safety)

Polluter bot actions:
- Fake a task (other players nearby)
- Sabotage (cooldown ready, no witnesses, meter low)
- Spawn pollution (cooldown, out of sight)
- Slime a cleaner (target alone, no witnesses in vision radius)

**Memory** (feeds voting): each bot logs `{time, player_id, room, action}` for what it saw within its vision radius (line of sight via raycast).

**Voting**: suspicion score per player = seen near sabotage start + seen sliming + seen faking a task + accused by phrases. Vote the max if above threshold, else skip. Polluter bots accuse a random cleaner they "saw" somewhere.

**Phrases**: bots post 1-2 quick phrases from memory ("I saw Red near the Harbor").

**Difficulty knob**: vision radius, reaction delay, mistake rate. Demo mode uses an easy setting.

Pathfinding: `NavigationRegion2D` + `NavigationAgent2D` on the map.

Done when: 1 human + 5 bots play a full round, bots do tasks, a polluter bot sabotages and slimes, meetings produce votes.

---

## 2. Camera waste sorting
Flow: task opens -> camera preview -> player points at a real object -> tap "Scan" -> frame (JPEG, max 640 px) sent to `/classify` -> top label + confidence -> player taps the matching bin -> server marks the task done.

**Classes (v1)**: `plastic`, `glass`, `metal`, `paper_cardboard`, `organic`, `e_waste`, `other`. Map each to a bin color shown in-game.

**Model plan**:
1. Baseline in week 1: fine-tune a small pretrained image classifier (e.g. MobileNetV3 / EfficientNet-B0 / a small ViT) on public data: **TrashNet**, **TACO** (crop the boxes), plus Kaggle "garbage classification" sets. Check each dataset's license before use.
2. Week 2-3: add 200+ **own photos** of the actual demo props (bottle, can, carton, banana peel, old phone) under venue-like light. This matters more than model size.
3. Report accuracy on a held-out set of own photos, not only the public test split.

**Where the model runs**: `ai_service` on the host laptop (CPU is fine for one image). Phone in solo mode reaches the laptop over the hotspot if `ai_service_url` is set in settings; otherwise manual fallback. On-device inference (ONNX/TFLite Android plugin) is a stretch goal, not v1.

**Godot camera risk**: Android camera access from Godot (`CameraServer` / `CameraFeed`) must be proven in the week-1 spike on our phones. If it fails, fallback order: (a) small Android plugin that returns a photo, (b) manual sort mini-game.

---

## 3. Pollution director
Purpose: keep every round tense. Inputs every 10 s from the server:
`{t, meter, task_progress, active_sabotages, pollution_objects, players_alive, slimed, recent_meetings, mode}`

Output: `{spawn_rate_mult: 0.5-2.0, ambient_pollution: [{type, room}], hint: str|null}`
- Ambient pollution = small trash spawns not caused by polluters (keeps cleaners busy when polluters are quiet).
- `spawn_rate_mult` scales the ambient spawns only. It never changes polluter cooldowns (fairness).
- `hint` = optional tip shown to cleaners ("Smoke is rising in the Landfill").

v1: rule-based curve (target meter band over time, push toward it). Stretch: tune the curve from telemetry of playtests. Show the director state in-game as a "city mood" icon so judges can see it.

Fallback: `game/ai/director.gd` runs the same rule curve locally.

---

## `ai_service` API contract (FastAPI, port 8000)
| Method | Path | Request | Response |
|---|---|---|---|
| GET | `/health` | — | `{"ok": true, "model": "name@version"}` |
| POST | `/classify` | multipart `image` (JPEG) | `{"label": "plastic", "confidence": 0.91, "top3": [["plastic",0.91],["glass",0.05],["metal",0.02]], "ms": 85}` |
| POST | `/director` | JSON state (above) | JSON output (above) |
| POST | `/telemetry` | JSON `{event, t, data}` | `{"ok": true}` |

Rules: `/classify` responds in under 1 s on the laptop. Unknown or low confidence (<0.5) returns `label: "other"` and the game asks the player to pick a bin manually. Changing this contract = note in `PROGRESS.md` > Decisions + tell AI dev 2.
