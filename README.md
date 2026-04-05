# PICO-8 Offline Bible

Your complete offline reference for PICO-8 development on Raspberry Pi 4B.

> [!TIP]
> Viewing in Obsidian: open this folder as a vault. For Pi/terminal use: `sudo apt install glow` then run `glow` in this directory.

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
| [[01-hello-pico8\|01 - Hello PICO-8]] | Game loop, shapes, colors |
| [[02-sprites-animation\|02 - Sprites & Animation]] | Sprites, animation frames, flip |
| [[03-input-movement\|03 - Input & Movement]] | btn(), velocity, friction |
| [[04-tilemaps-camera\|04 - Tilemaps & Camera]] | map(), camera, tile collision |
| [[05-pong\|05 - Pong]] | Build Pong step by step |
| [[06-platformer\|06 - Platformer Physics]] | Gravity, jump, physics |
| [[07-entities\|07 - Entities]] | Enemies, bullets, pickups |
| [[08-procedural\|08 - Procedural Generation]] | Random terrain, caves, dungeons |
| [[09-polish\|09 - Polish]] | Sound, game states, particles, saving |

---

## Reference

### Editors
| File | Topic |
|------|-------|
| [[shortcuts\|Shortcuts]] | All keyboard shortcuts |
| [[sprite-editor\|Sprite Editor]] | Drawing sprites & flags |
| [[map-editor\|Map Editor]] | Building tilemaps |
| [[sound-music\|Sound & Music]] | SFX & music tracker |

### Lua
| File | Topic |
|------|-------|
| [[basics\|Lua Basics]] | Lua syntax crash course |

### API Reference
| File | Topic |
|------|-------|
| [[graphics\|Graphics API]] | Drawing, sprites, palette, camera |
| [[input\|Input API]] | Buttons, controllers, mapping |
| [[math\|Math API]] | Math functions & operators |
| [[map\|Map API]] | Tilemap functions |
| [[audio\|Audio API]] | SFX & music playback |
| [[system\|System API]] | stat(), time(), cartdata, memory |

### Snippets (copy-paste ready)
| File | Topic |
|------|-------|
| [[movement\|Movement]] | 4-dir, 8-dir, friction, acceleration |
| [[collision\|Collision]] | AABB box, tile collision |
| [[camera\|Camera]] | Scrolling, deadzone, screen shake |
| [[particles\|Particles]] | Simple particle system |
| [[math-helpers\|Math Helpers]] | lerp, clamp, distance, angle |

### Game Examples (full annotated cartridges)
| File | Topic |
|------|-------|
| [[platformer\|Platformer]] | Jump, gravity, tile collision |
| [[side-scroller\|Side Scroller]] | Scrolling world, parallax |
| [[top-down-rpg\|Top-Down RPG]] | Top-down movement, rooms |
| [[shooter\|Shooter]] | Bullets, enemies, score |

### One-Page Reference
| File | Topic |
|------|-------|
| [[cheatsheet\|Cheatsheet]] | Everything on one page |

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
