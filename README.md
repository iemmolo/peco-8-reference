# PICO-8 Offline Bible

Your complete offline reference for PICO-8 development on Raspberry Pi 4B.

> **Tip:** View these files with `glow` for best experience.
> Install: `sudo apt install glow` then run `glow` in this directory.

---

## Quick Start

```
CTRL+R   run / reload
CTRL+S   save
ESC      toggle editor / console
ALT+←→   switch between editors
```

---

## Tutorials (Start Here)

| File | Topic |
|------|-------|
| [tutorials/01-hello-pico8.md](tutorials/01-hello-pico8.md) | Game loop, shapes, colors |
| [tutorials/02-sprites-animation.md](tutorials/02-sprites-animation.md) | Sprites, animation frames, flip |
| [tutorials/03-input-movement.md](tutorials/03-input-movement.md) | btn(), velocity, friction |
| [tutorials/04-tilemaps-camera.md](tutorials/04-tilemaps-camera.md) | map(), camera, tile collision |
| [tutorials/05-pong.md](tutorials/05-pong.md) | Build Pong step by step |
| [tutorials/06-platformer.md](tutorials/06-platformer.md) | Gravity, jump, physics |
| [tutorials/07-entities.md](tutorials/07-entities.md) | Enemies, bullets, pickups |
| [tutorials/08-procedural.md](tutorials/08-procedural.md) | Random terrain, caves, dungeons |
| [tutorials/09-polish.md](tutorials/09-polish.md) | Sound, game states, particles, saving |

---

## Reference

### Editors
| File | Topic |
|------|-------|
| [editor/shortcuts.md](editor/shortcuts.md) | All keyboard shortcuts |
| [editor/sprite-editor.md](editor/sprite-editor.md) | Drawing sprites & flags |
| [editor/map-editor.md](editor/map-editor.md) | Building tilemaps |
| [editor/sound-music.md](editor/sound-music.md) | SFX & music tracker |

### Lua
| File | Topic |
|------|-------|
| [lua/basics.md](lua/basics.md) | Lua syntax crash course |

### API Reference
| File | Topic |
|------|-------|
| [api/graphics.md](api/graphics.md) | Drawing, sprites, palette, camera |
| [api/input.md](api/input.md) | Buttons, controllers, mapping |
| [api/math.md](api/math.md) | Math functions & operators |
| [api/map.md](api/map.md) | Tilemap functions |
| [api/audio.md](api/audio.md) | SFX & music playback |
| [api/system.md](api/system.md) | stat(), time(), cartdata, memory |

### Snippets (copy-paste ready)
| File | Topic |
|------|-------|
| [snippets/movement.md](snippets/movement.md) | 4-dir, 8-dir, friction, acceleration |
| [snippets/collision.md](snippets/collision.md) | AABB box, tile collision |
| [snippets/camera.md](snippets/camera.md) | Scrolling, deadzone, screen shake |
| [snippets/particles.md](snippets/particles.md) | Simple particle system |
| [snippets/math-helpers.md](snippets/math-helpers.md) | lerp, clamp, distance, angle |

### Game Examples (full annotated cartridges)
| File | Topic |
|------|-------|
| [games/platformer.md](games/platformer.md) | Jump, gravity, tile collision |
| [games/side-scroller.md](games/side-scroller.md) | Scrolling world, parallax |
| [games/top-down-rpg.md](games/top-down-rpg.md) | Top-down movement, rooms |
| [games/shooter.md](games/shooter.md) | Bullets, enemies, score |

### One-Page Reference
| File | Topic |
|------|-------|
| [cheatsheet.md](cheatsheet.md) | Everything on one page |

---

## PICO-8 Specs at a Glance

| | |
|-|-|
| Screen | 128x128 pixels |
| Colors | 16 (fixed palette) |
| Sprites | 256 total (8x8 px each) |
| Map | 128x64 tiles |
| Sound | 4 channels, 64 SFX |
| Code | Lua, max 8192 tokens |
| Number range | -32768 to 32767.99 |

## Color Palette

```
 0 ⬛ Black       8 🟥 Red
 1 🟦 Dark Blue   9 🟧 Orange
 2 🟪 Dark Purple 10 🟨 Yellow
 3 🟩 Dark Green  11 🟩 Green
 4 🟫 Brown       12 🟦 Blue
 5 ◼  Dark Gray   13 🟣 Indigo
 6 ◻  Light Gray  14 🩷 Pink
 7 ⬜ White       15 🍑 Peach
```

## Controllers (N64/SNES USB)

```
btn(0) = LEFT     btn(4) = O  (A on N64, B on SNES)
btn(1) = RIGHT    btn(5) = X  (B on N64, Y on SNES)
btn(2) = UP
btn(3) = DOWN
```

Player 2: same calls with second argument `btn(0,1)`
# peco-8-refence
