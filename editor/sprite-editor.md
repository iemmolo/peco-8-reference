# Sprite Editor

## Overview

- 256 sprites total, each **8x8 pixels**
- Split into 4 tabs (0–3), 64 sprites each
- **Tabs 2 & 3 are shared with the Map** — using them for sprites means less map space and vice versa
- Sprite #0 is always the "eraser" in the map editor — don't put important graphics there

---

## Navigating Sprites

```
Q / -     previous sprite
W / =     next sprite
SHIFT+Q   previous sprite row (move back 16)
SHIFT+W   next sprite row (move forward 16)
+ / -     navigate tabs
```

Sprite number shown in bottom-left of editor.

---

## Drawing Tools

Select with icons at top of editor:

| Tool | Use |
|------|-----|
| Pencil | Draw pixel by pixel |
| Stamp | Paste a copied region |
| Select | Select a rectangle of pixels |
| Shape | Draw lines, rects, circles |
| Fill | Flood fill a region |

**Shortcut tips:**
- `RMB` anywhere = eyedropper (pick color under cursor)
- Hold `CTRL` while drawing = replace color mode
- Hold `SPACE` = pan around

---

## Colors

16 colors, numbered 0–15. Click to select. `1/2` keys cycle through them.

```
 0  black       8  red
 1  dark blue   9  orange
 2  dark purple 10 yellow
 3  dark green  11 green
 4  brown       12 blue
 5  dark gray   13 indigo
 6  light gray  14 pink
 7  white       15 peach
```

---

## Flip & Rotate

```
F     flip vertical
V     flip horizontal
R     rotate 90° clockwise (square selections only)
```

Useful for making directional characters — draw one direction, copy and flip for others.

---

## Multi-Sprite (Wide/Tall Sprites)

The **size selector** (bottom-right slider in editor, or the 2x2 grid icon) lets you edit multiple sprites as one unit. You can draw a 16x16 character using a 2x2 sprite block.

In code, draw with width/height args:
```lua
spr(n, x, y, 2, 2)   -- draw 2x2 sprite block (16x16 pixels)
spr(n, x, y, 1, 2)   -- draw 1x2 block (8x16 pixels)
```

---

## Sprite Flags

Each sprite has 8 flags (bits 0–7). Use them to tag tiles for game logic.

```lua
fset(n, f, v)     -- set flag f on sprite n to true/false
fget(n, f)        -- get flag f value for sprite n
fget(n)           -- get all 8 flags as a bitfield number
```

**Common use:** mark solid tiles with flag 0, hazards with flag 1, etc.

```lua
-- in _init, tag tiles in editor first, then:
if fget(mget(tx, ty), 0) then
  -- tile is solid
end
```

---

## Tips

- Plan sprites before drawing — it's easy to run out of sprite space
- Use consistent pixel art conventions (light source top-left, 1px outlines)
- Palette swaps via `pal()` let you reuse the same sprite with different colors
- Set `palt(0, false)` to make black non-transparent if you need black outlines
