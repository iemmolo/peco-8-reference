# Map API

## Draw Map

```lua
map(tile_x, tile_y, sx, sy, tile_w, tile_h, [layer])
```

| Param | Meaning |
|-------|---------|
| `tile_x, tile_y` | Top-left map tile to start from |
| `sx, sy` | Screen position to draw at |
| `tile_w, tile_h` | How many tiles wide/tall to draw |
| `layer` | Optional: bit flags to filter which tiles draw |

```lua
-- draw the top-left 16x16 tile area to screen position 0,0
map(0, 0, 0, 0, 16, 16)

-- draw from map position (8,4), 16 tiles wide x 8 tiles tall
map(8, 4, 0, 0, 16, 8)
```

### Layer filtering

Only draws tiles whose sprite has specific flags set:

```lua
map(0, 0, 0, 0, 16, 16, 1)   -- only tiles with flag 0 (value=1)
map(0, 0, 0, 0, 16, 16, 2)   -- only tiles with flag 1 (value=2)
map(0, 0, 0, 0, 16, 16, 4)   -- only tiles with flag 2 (value=4)
```

Combine layers: `1|2` = draw tiles with flag 0 OR flag 1.

---

## Get / Set Tiles

```lua
mget(x, y)        -- get sprite number at map tile (x, y)
mset(x, y, v)     -- set tile at (x, y) to sprite v
```

```lua
-- read tile at pixel position (world coords)
local tile = mget(flr(world_x / 8), flr(world_y / 8))

-- erase a tile (set to 0)
mset(tx, ty, 0)

-- destroy a breakable block when hit
mset(flr(bx/8), flr(by/8), 0)
```

---

## Sprite Flags (for tile logic)

```lua
fget(n)           -- get all flags as a number
fget(n, f)        -- get flag f (0–7) as true/false
fset(n, f, v)     -- set flag f on sprite n
```

Set flags in the **Sprite Editor** — click the flags panel below each sprite.

**Convention:**
```
Flag 0 = solid (collision)
Flag 1 = hazard / kills player
Flag 2 = climbable / ladder
Flag 3 = platform (one-way)
```

---

## Tile Collision Helpers

```lua
-- is this world pixel inside a solid tile?
function solid(px, py)
  return fget(mget(flr(px/8), flr(py/8)), 0)
end

-- does the player bounding box overlap any solid tile?
-- assumes player is 8x8, position is top-left
function player_collide(px, py)
  return solid(px,   py)   or  -- top-left
         solid(px+7, py)   or  -- top-right
         solid(px,   py+7) or  -- bottom-left
         solid(px+7, py+7)     -- bottom-right
end
```

---

## Scrolling with Camera

```lua
function _update()
  -- scroll camera to follow player
  cam_x = px - 60
  cam_y = py - 60
  cam_x = mid(0, cam_x, (map_w * 8) - 128)
  cam_y = mid(0, cam_y, (map_h * 8) - 128)
end

function _draw()
  cls()
  camera(cam_x, cam_y)       -- offset all drawing
  map(0, 0, 0, 0, 128, 64)  -- draw full map (camera handles clipping)
  spr(1, px, py)             -- player in world coords
  camera()                   -- reset camera for HUD
  print("hp:"..hp, 2, 2, 7) -- HUD stays on screen
end
```

---

## Textured Line

```lua
tline(x0, y0, x1, y1, mx, my, [mdx, mdy])
-- draws a line sampling from the map (for 3D/raycasting effects)
-- mx,my    = starting map coordinate (in tiles, fractional OK)
-- mdx,mdy  = map coordinate step per pixel (default 1/8, 0)
```

---

## Map Specs

| | |
|-|-|
| Size | 128 × 64 tiles |
| Shared area | Rows 32–63 shared with sprite tabs 2 & 3 |
| Tile size | 8 × 8 pixels |
| Full map pixels | 1024 × 512 |
| One screen | 16 × 16 tiles |
| Sprite 0 | Always blank/eraser — don't use for art |
