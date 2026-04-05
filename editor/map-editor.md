# Map Editor

## Overview

- Map is **128 tiles wide × 64 tiles tall**
- Each tile is an 8x8 sprite → full map = 1024×512 pixels
- One screen = **16×16 tiles** (128/8 = 16)
- **Lower 32 rows (y=32–63) are shared with sprite tabs 2 & 3** — decide early which you need more of
- Sprite #0 = eraser tile — always draws as blank

---

## Navigating the Map

```
SPACE (hold)    pan around
Mouse wheel     zoom in / out
< / >           zoom
Q / W           previous / next sprite in picker
```

---

## Placing Tiles

1. Select a sprite from the picker at the bottom
2. `LMB` to paint on the map
3. `RMB` to pick (eyedropper) the tile under cursor

Sprite #0 erases (it's the empty tile).

---

## Drawing the Map in Code

```lua
-- draw a 16x16 tile section of the map to screen position 0,0
map(0, 0, 0, 0, 16, 16)

-- with camera offset (for scrolling):
camera(cam_x, cam_y)
map(0, 0, 0, 0, 128, 64)  -- draw whole map, camera handles offset
```

### `map()` signature

```lua
map(tile_x, tile_y, sx, sy, tile_w, tile_h, [layer])
--  tile_x/y  = top-left map tile to start from
--  sx/sy     = screen position to draw at
--  tile_w/h  = how many tiles wide/tall to draw
--  layer     = optional: only draw tiles where fget(tile, layer_bit) == true
```

### Layer filtering

```lua
-- only draw tiles with flag 0 set (e.g. solid ground layer)
map(0, 0, 0, 0, 16, 16, 1)

-- only draw tiles with flag 1 set (e.g. decoration layer)
map(0, 0, 0, 0, 16, 16, 2)
```

Flag values for layer: `1`=flag0, `2`=flag1, `4`=flag2, etc. (powers of 2)

---

## Reading & Writing Tiles

```lua
-- get the sprite number at map position (tx, ty)
local tile = mget(tx, ty)

-- set a tile at map position
mset(tx, ty, sprite_num)

-- check if a tile is solid (has flag 0)
if fget(mget(tx, ty), 0) then
  -- solid tile!
end
```

**Convert pixel position to tile position:**
```lua
local tx = flr(px / 8)
local ty = flr(py / 8)
```

---

## Tile Collision Pattern

```lua
-- check if a point (px, py) is inside a solid tile
function solid_at(px, py)
  local tx = flr(px / 8)
  local ty = flr(py / 8)
  return fget(mget(tx, ty), 0)
end

-- check player bounding box corners
function player_on_ground()
  return solid_at(px+1, py+8) or solid_at(px+6, py+8)
end
```

---

## Multi-Room / Multi-Screen Maps

Divide the map into screen-sized chunks (16×16 tiles each):

```lua
-- room 0,0 starts at tile 0,0
-- room 1,0 starts at tile 16,0
-- room 0,1 starts at tile 0,16

function draw_room(rx, ry)
  map(rx*16, ry*16, 0, 0, 16, 16)
end
```

---

## Tips

- Use flag 0 = solid, flag 1 = hazard, flag 2 = climbable — consistent flagging saves time
- Tile 0 is transparent/empty — don't accidentally draw anything in sprite slot 0
- For large scrolling worlds: track `cam_x, cam_y` in pixels, use `camera(cam_x, cam_y)` before `map()`
- The map editor has a "copy region" mode — useful for repeating terrain patterns
