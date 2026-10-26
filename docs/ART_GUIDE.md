# Art guide — designer <-> dev contract

Owner: Designer. Devs follow these specs so art drops in without code changes.

## Identity (original, not Among Us)
- Setting: a Tunisian coastal city (medina walls, blue doors, souk awnings, harbor, olive trees, a landfill at the edge).
- Characters must **not** use the bean/visor silhouette. Suggestion: chunky round characters with a recycling-worker vest and a distinctive helmet or hat; color = player identity.
- Polluters look identical to cleaners. Their abilities show only on their own screen.
- Palette: clean areas in Mediterranean blue/white/olive green; pollution in sludge brown, oily purple, smog grey. Pollution must read instantly on a small screen.

## Technical specs
- Base resolution **1280x720 landscape**, stretch mode `canvas_items`, aspect `expand`.
- Tile size **64x64 px**. Characters about **64x96 px** frame size.
- Character animations: idle (4 frames), walk (6-8 frames), use/clean (4), slimed (4 loop). Side view only; flip for left/right.
- Format: PNG, transparent background. Sprite sheets in a single row per animation, or separate PNG frames in a folder.
- UI: min touch target 88x88 px. Joystick bottom-left, action buttons bottom-right. Readable text min 24 px.
- Fonts: one Latin font + one with Arabic glyphs (e.g. Noto Sans Arabic / Cairo). Check licenses (OFL is fine).
- Audio: OGG for music, WAV for short SFX. Note the source and license of every sound in `game/assets/CREDITS.md`.

## Folder and naming
```
game/assets/
  characters/   char_base_walk.png, char_base_idle.png ...
  tiles/        tiles_city.png (+ .tres TileSet made by game dev)
  props/        prop_bin_plastic.png, prop_oil_spill.png ...
  ui/           ui_btn_use.png, ui_meter_frame.png, icon_plastic.png ...
  fx/           fx_smoke.png, fx_slime.png
  audio/        sfx_task_done.wav, music_lobby.ogg
  fonts/
  CREDITS.md
```
snake_case, lowercase, prefix by type. Do not rename a file once used; add `_v2` instead.

## Import settings (game dev sets once)
- Pixel-art style: Texture filter `Nearest`. Smooth/vector style: `Linear` + mipmaps off. **Pick one style in week 1.**
- Mobile: keep textures power-of-two where possible, total assets under 50 MB.

## AI-generated assets
Allowed by the hackathon rules? **Unknown** — see `PROGRESS.md` > Decisions. Until confirmed, keep a list of which assets were AI-assisted.

## Placeholder phase
Devs use colored rectangles with the final filenames until art arrives. Designer replaces the file; nothing else changes.
