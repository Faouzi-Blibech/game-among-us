# Game Design Document — "Clean City" (working title)

Status: draft v0.2 (Oct 26, 2026). Former OPEN items resolved as Claude's calls (marked **CALL**); the team can overturn any of them before Nov 3.

## Pitch (one line)
A city is drowning in waste. Cleaners recycle and rebuild; hidden polluters sabotage. Find the polluters before the pollution meter hits 100%.

## Players and roles
| Players | Polluters | Notes |
|---|---|---|
| 4-6 | 1 | default for demo |
| 7-10 | 2 | |
| 10 (max) | 3 | only if playtests show it is fun |

Empty seats are filled by bots (see `AI.md`). Roles are assigned by the server at round start and shown privately.

### Cleaner
- Has a task list (4-6 tasks). Completing tasks fills the **team task bar** and produces **materials**.
- Can fix sabotages, clean slimed players, report, call an emergency meeting (1 per round).

### Polluter
- Has fake tasks (can stand at a task to look busy; no progress).
- **Sabotage** (cooldown 30 s): starts a spreading pollution event at a chosen location.
- **Spawn pollution** (cooldown 20 s): drops a trash pile / smog cloud nearby. Each object adds to the meter until cleaned.
- **Slime** (cooldown 35 s, range 1.5 tiles): tags a cleaner. Slimed cleaner is frozen in place, cannot do tasks, and can only be freed by another player holding "clean" for 2 s. A slimed player can be **reported** -> meeting.
- **Slime timeout (CALL)**: a slimed cleaner auto-frees after 20 s, so a bot-only area cannot soft-lock and a judge is never stuck.
- **Sewers (CALL)**: yes, 2 sewer pairs (Harbor <-> Landfill, Market <-> Park), polluters only, 1 s enter/exit animation that nearby players can see.

## Pollution meter (core tension)
- 0-100%, shared, visible to everyone at top of screen.
- Rises from: active sabotages (per second), uncleaned pollution objects (per second), slimed players (small per second).
- Falls from: fixing sabotage (instant drop), cleaning objects, built **upgrades** (see below), passive slow decay when nothing is active.
- **Polluters win** at 100%. **Cleaners win** when the task bar is full OR all polluters are voted out.
- Tuning target: an unchecked polluter reaches 100% in about 4 minutes.

## "Rien ne se jette, tout s'invente" — materials and upgrades
- Tasks turn waste into materials: plastic, glass, metal, paper, compost.
- At the **Workshop**, cleaners spend materials on upgrades (team-wide):
  - Solar panel: meter decays faster.
  - Compost station: organic pollution cleans 2x faster.
  - Sorting robot: one bin task auto-completes.
- **Scope (CALL)**: v1 ships **solar panel only** (cost: 3 metal + 2 glass, meter decay x2). Compost station and sorting robot are week-4 stretch goals.

## Tasks (v1 target: 4 regular + camera task)
Each task is a 5-15 s touch mini-game. On completion show **one real fact** (e.g. "A glass bottle can be recycled endlessly without losing quality.").
| Task | Interaction | Teaches |
|---|---|---|
| Sort the bin | drag 6 items into plastic / glass / paper / metal / organic bins | correct sorting |
| Fix the leaking pipe | rotate pipe tiles to connect | water waste |
| Compost | drag peels in, swipe to mix | organic waste -> soil |
| Repair a phone | drag parts back (battery, screen, board) | e-waste, repair over replace |
| **Camera sort** (AI) | scan a real object with the phone camera; classifier says its type; player confirms the bin | real-world sorting |

Camera sort fallback: if camera or `ai_service` fails, show the same object as an image and the player sorts it manually.

## Sabotages (v1 target: 2 + 1 critical)
| Sabotage | Effect | Fix |
|---|---|---|
| Oil spill (harbor) | spreads over tiles, slows players, +meter | 1 player scrubs 3 spots |
| Overflowing bins (market) | trash piles spawn every 5 s nearby | 1 player empties the bin |
| **Landfill fire** (critical) | +meter fast, smoke reduces vision | 2 players hold two valves within 45 s, else meter +25% |

## Meetings and voting
- Triggered by: report of a slimed player, or emergency button.
- Discussion 30 s, vote 20 s (demo mode: 20 s / 15 s).
- v1 communication: **quick phrases** only ("I saw X near Y", "X was faking a task", "Skip"), no free text. Easier on mobile, safe in a public demo, and bots can use the same phrases.
- Most votes = ejected, role revealed ("X was a polluter"). Ties = nobody.

## Map
Original Tunisian-inspired city. **Map name (CALL): "Medina Bay".** Art direction: `ART_GUIDE.md`.

Layout (CALL), a loop so nobody hits dead ends, Town hall in the center:
```
   [Landfill] ---- [Recycling center] ---- [Workshop]
       |                   |                   |
   [Harbor]  ------- [Town hall] -------- [Market/souk]
       |                   |                   |
   [Beach]  -------- [Park/compost] ----------+
```
- 8 rooms, walk across the whole map in about 25 s (keeps 4-minute demo rounds busy).
- Sabotage rooms spread out: oil spill = Harbor, bins = Market, fire = Landfill, so fixing pulls players apart.
- Designer turns this into a real sketch by Nov 2.

## Controls (mobile)
- Left: virtual joystick. Right: context button (Use / Fix / Clean / Report), plus Sabotage / Spawn / Slime buttons for polluters.
- Map button, task list toggle. Big touch targets (min 88 px at 720p).

## Demo mode (for judges, first-class)
- 1 human + bots, 4-minute round, human is always a cleaner on first play.
- 20-second interactive tutorial: move -> do one task -> see the meter -> see a sabotage -> one meeting.
- End screen shows impact: "Your team recycled 42 items, saved X kg CO2 (estimate)" plus 1 fact.
- "Play as polluter" button for a second round.

## Localization
English first; French and Arabic strings in week 4 (Godot `TranslationServer`, CSV). Arabic needs RTL and a font with Arabic glyphs.

## The judge's first 60 seconds (CALL — design target for demo mode)
| Time | What happens | What the judge should get |
|---|---|---|
| 0-5 s | Title, one tap "Play" (no login, no settings) | zero friction |
| 5-25 s | Tutorial: move with joystick, sort one bin, fact pops up | "I clean the city" |
| 25-40 s | A sabotage fires, meter climbs, arrow points to it, judge fixes it | "someone is polluting" |
| 40-60 s | A bot is found slimed -> meeting -> bots post phrases -> judge votes | "find the polluter" |
| 60 s+ | Free play; camera task offered with a real prop on the table | the AI wow moment |

## Game name (team decides, by Nov 2)
Working title "Clean City". Candidates (Claude's suggestions):
1. **Zibla** — Tunisian word for trash; local, short, memorable.
2. **Medina Bay** — same as the map; calm, Mediterranean.
3. **Who Polluted?** — says the game in two words for judges.

## Resolved design calls (summary)
| Item | Call |
|---|---|
| Sewers for polluters | yes, 2 pairs, visible animation |
| Upgrades in v1 | solar panel only |
| Map | "Medina Bay", 8-room loop above |
| Slime timeout | auto-free after 20 s |
| Name | open, 3 candidates above |
