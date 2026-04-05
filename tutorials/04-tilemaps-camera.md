---
tags: [pico8, tutorial]
---
# Tutorial 04 — Tilemaps & Camera

**What you'll learn:** Drawing the map, moving a player through a tile world, camera scrolling, tile collision, and multi-screen rooms.

**Prerequisites:** [[03-input-movement|03 - Input & Movement]]

---

## How the Map Works

PICO-8's map is a **128×64 grid of tiles**. Each tile is a reference to a sprite number. When you paint a tile in the map editor, you're just storing "sprite #5 goes here."

One screen = **16×16 tiles** (128px ÷ 8px per tile = 16).

```
Map size:  128 tiles wide × 64 tiles tall
Pixel size: 1024px × 512px
One screen: 16×16 tiles = 128×128 px
```

---

## Step 1 — Draw the Map

First, build a small map in the map editor. Make sure your solid tiles have **flag 0** set (click the flag panel in the sprite editor for each tile sprite).

```lua
function _draw()
  cls()
  map(0, 0, 0, 0, 16, 16)
end
```

### `map()` arguments

```lua
map(tile_x, tile_y, sx, sy, tile_w, tile_h)
--  tile_x,y  = top-left map tile to start drawing from
--  sx,sy     = screen pixel position to draw at
--  tile_w,h  = how many tiles wide/tall to draw
```

`map(0, 0, 0, 0, 16, 16)` = start at map tile (0,0), draw to screen (0,0), draw 16×16 tiles.

---

## Step 2 — Add a Player

```lua
function _init()
  px = 60   -- player world position (pixels)
  py = 60
end

function _update()
  local spd = 2
  if btn(0) then px -= spd end
  if btn(1) then px += spd end
  if btn(2) then py -= spd end
  if btn(3) then py += spd end
end

function _draw()
  cls()
  map(0, 0, 0, 0, 16, 16)   -- draw map first (behind player)
  spr(1, px, py)              -- draw player on top
end
```

Run it — your player walks over the map. But they walk through walls! We'll fix that next.

---

## Step 3 — Tile Collision

We need to check if the tile at a given **world position** is solid.

```lua
-- is the tile at world pixel (wx, wy) solid?
function solid_at(wx, wy)
  local tx = flr(wx / 8)   -- pixel → tile coordinate
  local ty = flr(wy / 8)
  return fget(mget(tx, ty), 0)   -- check flag 0
end
```

`mget(tx, ty)` gets the sprite number at tile position (tx, ty).
`fget(sprite_num, 0)` returns true if flag 0 is set on that sprite.

### Setting up flags

In the **sprite editor**:
1. Select a solid tile sprite (e.g. your stone wall)
2. At the bottom of the editor you'll see 8 small flag buttons (0–7)
3. Click flag **0** to set it — it turns red/on

Do this for every tile type that should be solid.

---

## Step 4 — Simple Top-Down Collision

For top-down movement, check the four corners of the player before moving:

```lua
function _update()
  local spd = 2
  local dx, dy = 0, 0

  if btn(0) then dx = -spd end
  if btn(1) then dx =  spd end
  if btn(2) then dy = -spd end
  if btn(3) then dy =  spd end

  -- try moving X
  local nx = px + dx
  if not (solid_at(nx,   py+1) or solid_at(nx+7, py+1) or
          solid_at(nx,   py+6) or solid_at(nx+7, py+6)) then
    px = nx
  end

  -- try moving Y
  local ny = py + dy
  if not (solid_at(px+1, ny)   or solid_at(px+6, ny) or
          solid_at(px+1, ny+7) or solid_at(px+6, ny+7)) then
    py = ny
  end
end
```

We check **four points** — the corners of the player's bounding box (inset by 1 pixel to avoid false positives on shared edges).

Moving X and Y **separately** means you can slide along walls instead of getting stuck.

---

## Step 5 — Scrolling Camera (The Big One)

When your world is bigger than one screen, you need a camera. `camera(x, y)` tells PICO-8 to offset all drawing by that amount — so the map and sprites automatically draw at the right place.

```lua
function _init()
  px = 60
  py = 60
  cam_x = 0
  cam_y = 0
end

function _update()
  -- (movement code here)

  -- update camera: center on player
  cam_x = px - 60    -- 60 = half screen (64) minus half sprite (4)
  cam_y = py - 60
end

function _draw()
  cls()

  -- SET CAMERA before drawing world
  camera(cam_x, cam_y)

  -- now draw everything in WORLD coordinates
  map(0, 0, 0, 0, 128, 64)   -- draw the whole map
  spr(1, px, py)               -- player in world coords

  -- RESET CAMERA before drawing HUD
  camera()

  -- HUD is in screen coordinates (doesn't scroll)
  print("pos:"..flr(px)..","..flr(py), 2, 2, 7)
end
```

### Why this works

`camera(cam_x, cam_y)` shifts the coordinate system. If `cam_x = 100`, then drawing at world x=100 appears at screen x=0. Your map and player positions are always in world coordinates — the camera handles the conversion.

---

## Step 6 — Clamp Camera to World Bounds

Without clamping, the camera shows black outside the map:

```lua
-- world size in pixels (128 tiles × 8 pixels)
local world_w = 128 * 8   -- 1024
local world_h = 64  * 8   -- 512

-- in _update(), after setting cam_x/cam_y:
cam_x = mid(0, cam_x, world_w - 128)   -- can't go past right edge
cam_y = mid(0, cam_y, world_h - 128)   -- can't go past bottom edge
```

`world_w - 128` = the rightmost camera position where the screen still fills with map.

---

## Step 7 — Full Working Example

Here's a complete scrolling top-down game you can run right now:

```lua
-- === SETUP ===
-- In sprite editor: draw a floor tile (sprite 2), wall tile (sprite 3)
-- Set flag 0 on sprite 3 (wall)
-- In map editor: paint a room with walls around the edge

world_w = 128 * 8
world_h = 64  * 8

function _init()
  px = 60
  py = 60
  cam_x = 0
  cam_y = 0
end

-- === TILE COLLISION ===

function solid_at(wx, wy)
  if wx < 0 or wy < 0 then return true end   -- world edge is solid
  return fget(mget(flr(wx/8), flr(wy/8)), 0)
end

-- === UPDATE ===

function _update()
  local spd = 2
  local dx, dy = 0, 0
  if btn(0) then dx = -spd end
  if btn(1) then dx =  spd end
  if btn(2) then dy = -spd end
  if btn(3) then dy =  spd end

  -- move X
  local nx = px + dx
  if not (solid_at(nx,   py+1) or solid_at(nx+7, py+1) or
          solid_at(nx,   py+6) or solid_at(nx+7, py+6)) then
    px = nx
  end

  -- move Y
  local ny = py + dy
  if not (solid_at(px+1, ny)   or solid_at(px+6, ny) or
          solid_at(px+1, ny+7) or solid_at(px+6, ny+7)) then
    py = ny
  end

  -- camera
  cam_x = mid(0, px - 60, world_w - 128)
  cam_y = mid(0, py - 60, world_h - 128)
end

-- === DRAW ===

function _draw()
  cls()
  camera(cam_x, cam_y)
  map(0, 0, 0, 0, 128, 64)
  spr(1, px, py)
  camera()
  print(flr(px).."/"..flr(py), 2, 2, 7)
end
```

---

## Step 8 — Room-by-Room (No Scrolling)

Alternatively, snap between discrete rooms — each room is exactly one screen (16×16 tiles).

```lua
local room_x = 0   -- which room column we're in
local room_y = 0   -- which room row we're in

function _update()
  -- movement (same as above)

  -- check if player left the screen
  if px < 0 then
    room_x -= 1
    px = 120       -- appear on right side of new room
  elseif px > 127 then
    room_x += 1
    px = 0
  end
  if py < 0 then
    room_y -= 1
    py = 120
  elseif py > 127 then
    room_y += 1
    py = 0
  end
end

function _draw()
  cls()
  -- draw room: offset into map by room position (each room = 16 tiles)
  map(room_x * 16, room_y * 16, 0, 0, 16, 16)
  spr(1, px, py)
end
```

The map is divided into screen-sized chunks. Room (0,0) uses tiles (0..15, 0..15). Room (1,0) uses tiles (16..31, 0..15). Room (0,1) uses tiles (0..15, 16..31). Plan your map accordingly.

---

## Step 9 — Layer Filtering

Draw different tile layers separately — e.g. floor layer then decoration layer on top:

```lua
-- in sprite editor:
-- floor tiles: flag 1 set (value = 2)
-- decoration tiles: flag 2 set (value = 4)

function _draw()
  cls()
  camera(cam_x, cam_y)
  map(0, 0, 0, 0, 128, 64, 2)    -- draw floor layer (flag 1)
  spr(1, px, py)                   -- player between layers
  map(0, 0, 0, 0, 128, 64, 4)    -- draw decoration on top (flag 2)
  camera()
end
```

The last argument to `map()` is a bitmask — only tiles with matching flags are drawn.

---

## Challenge

1. Add a second room to your map and connect them with an exit/entrance
2. Make the camera do a smooth lerp (see `snippets/camera.md`)
3. Add a door tile (interactable, flag 1) that teleports the player to another position
4. Create a larger world (e.g. 4×4 rooms) and add a mini-map in the corner that shows the player's position

---

## What You Learned

- `map(tile_x, tile_y, sx, sy, tw, th)` draws a section of the map
- `mget(tx, ty)` gets the sprite at a tile position
- `fget(sprite, flag)` checks if a flag is set — use for collision
- Convert pixel → tile with `flr(px / 8)`
- `camera(x, y)` offsets all drawing — the foundation of scrolling
- Always reset with `camera()` before drawing HUD
- Move X and Y separately so you can slide along walls
- Room system: offset map draw by `room_x * 16, room_y * 16`

**Next:** [[05-pong|05 - Pong]] — build a full game from scratch.
