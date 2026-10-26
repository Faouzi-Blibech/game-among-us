# CLAUDE.md — Green Hackathon game (working title: "Clean City")

Read this first. Then read only the doc for the area you are working on.

## What we are building
A mobile social-deduction party game about recycling, for the **Green Hackathon** (Swedish-Tunisian Games and Innovation Days, El Ghazala, **Nov 21-22, 2026**). Slogan: *"Rien ne se jette, tout s'invente."*

- **Cleaners** walk a Tunisian city map doing recycling tasks.
- **Polluters** (1-3, hidden) sabotage, spawn pollution, and "slime" cleaners. **No killing.**
- A shared **pollution meter**: polluters win at 100%. Cleaners win by finishing all tasks or voting out every polluter.
- AI features: **bot players**, **camera waste sorting** (scan real trash), **AI pollution director**.

Full rules: `docs/GDD.md`.

## Hackathon constraints (do not forget)
- Judged on a **real phone**, probably for 2-3 minutes. Must be understandable in under 30 seconds. Demo mode is a first-class feature.
- **Venue internet is unreliable.** Everything must work on a local hotspot with no internet.
- Judges include Swedish game-industry people, the ministry, Tunisie Telecom, Ericsson, Swedish Embassy.
- Unknown rules (judging criteria, AI-asset policy, pre-built code policy). Check `PROGRESS.md` > Decisions before assuming.

## Hard rules
1. **Original code and art only.** Never copy code, assets, names, sounds or map layouts from Among Us or the reverse-engineered C code. Mechanics may be similar; nothing else.
2. **Server-authoritative.** Clients send intents; only the server changes game state. See `docs/NETWORK.md`.
3. **Never block on the AI service.** Every AI call has a timeout and an offline fallback. See `docs/AI.md`.
4. **Mobile-first.** Touch controls, **landscape**, 1280x720 base resolution. Test on a real Android phone before calling a feature done.
5. **Bots and humans use the same code path.** A bot is a player whose input comes from `game/ai/` instead of the touch UI.
6. Stay in your area (table below). Changing a shared contract (`docs/NETWORK.md` RPC list, `docs/AI.md` API, `Events` signals) needs a note in `PROGRESS.md` > Decisions.
7. No commits or pushes unless the human asks. No AI attribution in commits. No emojis.

## Repo map
```
CLAUDE.md          this file
PROGRESS.md        done / next / known issues / decisions (read when resuming)
docs/
  TEAM.md          who owns what, week-by-week tasks, dependencies
  GDD.md           game design: roles, rules, tasks, sabotages, demo mode
  ARCHITECTURE.md  modes, autoloads, scene tree, folder ownership, fallbacks
  NETWORK.md       authority model, RPCs, synced state, lobby flow
  AI.md            bots, director, ai_service API contract, dataset
  ART_GUIDE.md     designer <-> dev contract: sizes, names, import settings
game/              Godot 4 project (client and server in one build)  -> game/CLAUDE.md
ai_service/        Python FastAPI service on the host laptop          -> ai_service/CLAUDE.md
```

## Ownership
| Area | Owner | Paths |
|---|---|---|
| Art, UI kit, audio picks, pitch visuals | Designer | `game/assets/`, `docs/ART_GUIDE.md` |
| Movement, map, task framework, tasks, APK builds | Game dev | `game/scenes/player`, `game/scenes/map`, `game/scenes/tasks`, `game/scenes/ui` |
| Networking, lobby, roles, meter, sabotage, slime, meetings, votes | AI dev 1 | `game/autoload/`, `game/scenes/lobby`, `game/scenes/meeting` |
| Bot players, director (game side) | AI dev 2 | `game/ai/` |
| Camera task, classifier, ai_service, telemetry | AI dev 3 | `ai_service/`, `game/scenes/tasks/camera_sort` |

Integration lead (merges to `main`, runs the weekly playtest): see `PROGRESS.md`.

## How to run (fill in during week 1)
- Godot version: **pin in week 1** and write it here: `Godot 4.x.x`
- Run client in editor: open `game/project.godot`, F5.
- Headless server: `godot --headless --path game -- --server --port 7777`
- AI service: see `ai_service/CLAUDE.md`.
- Android build: see `game/CLAUDE.md` > Export.

## Working style for agents
- Read `PROGRESS.md` before starting. Update your section when you finish a task.
- Prefer the smallest change that works; this is a 4-week prototype.
- Before saying "done": run it (editor or phone) and state what you saw.
- If a rule here conflicts with the human's request, ask the human.
