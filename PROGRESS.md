# PROGRESS

Read before starting work. Update your section when you finish something. Keep it short.

## Integration lead
TBD (name): merges to `main`, runs the weekly playtest on real phones.

## Timeline
| Week | Dates | Goal | Done when |
|---|---|---|---|
| 1 | Oct 27 - Nov 2 | GDD locked, Godot pinned, 5 spikes | each spike demoed on a real phone |
| 2 | Nov 3 - 9 | core loop multiplayer (move, 4 tasks, meter, 2 sabotages, slime, meeting, vote, win) | 3 humans finish a round on LAN |
| 3 | Nov 10 - 16 | bots, director, camera task in game, real art | 1 human + bots finish a round in solo |
| 4 | Nov 17 - 20 | demo mode, tutorial, i18n, sound, 3 outside playtests, pitch. **Freeze Nov 19** | a stranger understands it in < 30 s |
| Event | Nov 21 - 22 | polish, fixes, on-site requirements, pitch rehearsal | demo runs 10 times in a row without a crash |

## Week-1 spikes
- [ ] Designer: style sheet, 1 character, 1 room mockup
- [ ] Game dev: Godot project scaffolded, APK runs on 2 team phones
- [ ] AI dev 1: 2 phones + laptop in one LAN lobby (ENet), positions synced
- [ ] AI dev 2: bot walks to a task with NavigationAgent2D and "completes" it
- [ ] AI dev 3: Godot camera frame on Android + baseline classifier served by `/classify`

## Status by owner
### Designer
- Done: —
- Next: style sheet
### Game dev
- Done: —
- Next: scaffold `game/`
### AI dev 1 (network + game state)
- Done: —
- Next: LAN lobby spike
### AI dev 2 (bots + director)
- Done: —
- Next: bot navigation spike
### AI dev 3 (camera + ai_service)
- Done: —
- Next: Android camera spike, dataset download

## Known issues
- (none yet)

## Decisions
| Date | Decision | Why |
|---|---|---|
| 2026-10-26 | Engine: Godot 4 (Compatibility renderer, 2D) | team prefers Python-like scripting; light Android export |
| 2026-10-26 | Server-authoritative, one build for solo/LAN/online | one code path; demo works without internet |
| 2026-10-26 | AI v1 = bots (utility AI), camera sorting, director. No RL for bots | visible value for judges within 4 weeks |
| 2026-10-26 | No reuse of Among Us code/assets/names | IP risk; judges from the game industry |
| 2026-10-26 | Polluters spawn pollution objects, not convert players | keeps deduction and balance clear |
| 2026-10-26 | Sewers: yes, 2 pairs, polluters only (Claude's call) | escape tool replaces "kill and vent" tension |
| 2026-10-26 | v1 upgrades: solar panel only (Claude's call) | keeps the slogan mechanic, minimal scope |
| 2026-10-26 | Map "Medina Bay", 8-room loop (Claude's call) | no dead ends, sabotages spread out |
| 2026-10-26 | Slime auto-frees after 20 s (Claude's call) | no soft-lock for judges or bot-only areas |
| 2026-10-26 | Demo mode follows the "first 60 seconds" script in GDD (Claude's call) | judges play only 2-3 minutes |

## Open questions (ask organizers / decide as team)
- [ ] Is code written before Nov 21 allowed? (email organizers this week)
- [ ] Are AI-generated assets allowed?
- [ ] Judging criteria and submission format?
- [ ] Godot exact version: ___
- [ ] Test framework: GUT or gdUnit4?
- [ ] Game name: Zibla / Medina Bay / Who Polluted? (team vote by Nov 2)
- [ ] Team review of Claude's design calls in GDD (marked CALL) by Nov 2
