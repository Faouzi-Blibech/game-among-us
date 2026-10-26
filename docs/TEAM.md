# Team distribution

5 people, Oct 27 - Nov 22, 2026. Names go in the first column; roles and paths are fixed.
Rules for everyone: root `CLAUDE.md`. Status updates: `PROGRESS.md`.

## Who owns what
| Name | Role | Owns (paths) | Main docs |
|---|---|---|---|
| ___ | Designer | `game/assets/`, `docs/ART_GUIDE.md`, pitch deck | ART_GUIDE, GDD |
| ___ | Game dev | `game/scenes/player`, `map`, `tasks`, `ui`, Android export | ARCHITECTURE, GDD |
| ___ | AI dev 1: network + game state | `game/autoload/`, `game/scenes/lobby`, `meeting`, `sabotage` | NETWORK, ARCHITECTURE |
| ___ | AI dev 2: bots + director | `game/ai/` | AI (sections 1 and 3) |
| ___ | AI dev 3: camera + ai_service | `ai_service/`, `game/scenes/tasks/camera_sort` | AI (sections 2 and API) |

Integration lead (merges to `main`, runs weekly playtest): ___
Pitch owners: Designer + ___

## Week by week
### Week 1 (Oct 27 - Nov 2): spikes and decisions
| Who | Task | Done when |
|---|---|---|
| Designer | style sheet, 1 character, 1 room mockup, map sketch of Medina Bay | team approves the look |
| Game dev | pin Godot version, scaffold `game/`, joystick movement, APK build | APK runs on 2 team phones |
| AI dev 1 | ENet LAN lobby, 3 peers, synced positions | laptop + 2 phones move together |
| AI dev 2 | nav mesh, bot walks to a task and "completes" it | bot finishes 3 tasks alone |
| AI dev 3 | Android camera in Godot; baseline classifier behind `/classify` | photo from phone -> label back |
| All | vote on game name; review GDD CALLs; email organizers | answers in `PROGRESS.md` |

### Week 2 (Nov 3 - 9): core loop
| Who | Task |
|---|---|
| Designer | tileset, character animations, task UI art, meter and HUD |
| Game dev | task framework + 4 tasks (sort bin, pipe, compost, phone repair), HUD |
| AI dev 1 | roles, meter, 2 sabotages + landfill fire, slime, report, meeting, vote, win screen |
| AI dev 2 | cleaner bot utility AI (tasks, fix, free), bot memory |
| AI dev 3 | dataset (TrashNet/TACO + own photos), train v1, camera task scene |

Milestone: 3 humans finish a round on LAN.

### Week 3 (Nov 10 - 16): AI and art in
| Who | Task |
|---|---|
| Designer | full map art, props, pollution FX, sounds, fonts (Arabic) |
| Game dev | integrate art, solar panel upgrade, materials, sewers |
| AI dev 1 | WebSocket online mode, LAN discovery, reconnect handling |
| AI dev 2 | polluter bot, bot voting + phrases, director (game side + local fallback) |
| AI dev 3 | camera task in game end to end, `/director` and `/telemetry`, accuracy report |

Milestone: 1 human + bots finish a round in solo mode.

### Week 4 (Nov 17 - 20): demo and polish. Freeze Nov 19.
| Who | Task |
|---|---|
| Designer | pitch deck, end screen, trailer/screenshots |
| Game dev | demo mode + "first 60 seconds" tutorial, i18n (FR/AR) |
| AI dev 1 | stability: 10 rounds in a row without a crash, bad Wi-Fi test |
| AI dev 2 | bot difficulty tuning for demo, director tuning from playtests |
| AI dev 3 | own-photo fine-tune with the demo props, telemetry numbers for the pitch |
| All | 3 playtests with outsiders |

### Event (Nov 21 - 22)
Fixes only, on-site requirements, pitch rehearsal. Bring: 3 charged phones, laptop, own hotspot, demo props (bottle, can, carton, peel, old phone).

## Dependencies (who waits on whom)
- Everyone waits on **Game dev's scaffold** (week 1, first 2 days). Do it first.
- Game dev's task framework <- AI dev 1's `request_complete_task` RPC: agree on the signature on day 1 (it is in `NETWORK.md`).
- AI dev 2's bots <- player scene with input sources (Game dev) and `GameState` (AI dev 1).
- AI dev 3's camera task <- task framework (Game dev). Until then, build it as a standalone scene.
- Designer works with placeholders: devs use the final filenames from `ART_GUIDE.md`.

## Daily rhythm
- 10-minute sync each evening: what I finished, what blocks me.
- Each person's coding agent reads `CLAUDE.md` + their main docs above, then `PROGRESS.md`.
