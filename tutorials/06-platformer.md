---
tags: [pico8, tutorial]
---
# Tutorial 06 — Platformer Physics

**What you'll learn:** Gravity, jumping, proper tile collision (X and Y separately), the on_ground flag, and common platformer feel improvements.

**Prerequisites:** [[04-tilemaps-camera|04 - Tilemaps & Camera]]

---

## The Core Idea

A platformer has three physics rules:
1. **Gravity** pulls the player down every frame
2. **Jumping** adds a sudden upward velocity
3. **Tiles** stop the player when they collide

The trick is handling X and Y movement **separately** so the player can slide along walls instead of getting stuck in corners.

---

## Step 1 — Gravity and Falling

Gravity is just adding a small downward velocity every frame:

```lua
function _init()
  px  = 60
  py  = 60
  pvx = 0      -- velocity x
  pvy = 0      -- velocity y
end

-- constants
gravity  = 0.35
max_fall = 5     -- terminal velocity

function _update()
  -- gravity: pull downward each frame
  pvy = min(pvy + gravity, max_fall)

  -- apply velocity
  px += pvx
  py += pvy
end

function _draw()
  cls()
  spr(1, px, py)
end
```

Run it — the player falls through the bottom of the screen. Next we need tiles to stop them.

---

## Step 2 — Tile Collision

Set up your map first:
- Draw a ground tile (e.g. sprite #2) and set **flag 0** on it
- Paint some platforms in the map editor

```lua
-- check if a world pixel is inside a solid tile
function solid_at(wx, wy)
  local tx = flr(wx / 8)
  local ty = flr(wy / 8)
  -- out-of-bounds = solid (world edges act as walls)
  if tx < 0 or ty < 0 or tx > 127 or ty > 63 then return true end
  return fget(mget(tx, ty), 0)
end
```

Now resolve collisions after moving. **Crucially: move X first, resolve X, then move Y, resolve Y.** Never both at once.

```lua
function _update()
  -- gravity
  pvy = min(pvy + gravity, max_fall)

  -- horizontal: move then resolve
  px += pvx
  resolve_x()

  -- vertical: move then resolve
  py += pvy
  resolve_y()
end

function resolve_x()
  if pvx > 0 then
    -- moving right: check right edge of player (px+7)
    if solid_at(px+7, py+1) or solid_at(px+7, py+6) then
      px  = flr((px+7) / 8) * 8 - 8   -- snap left edge to tile boundary
      pvx = 0
    end
  elseif pvx < 0 then
    -- moving left: check left edge (px)
    if solid_at(px, py+1) or solid_at(px, py+6) then
      px  = flr(px / 8) * 8 + 8        -- snap to right side of tile
      pvx = 0
    end
  end
end

function resolve_y()
  if pvy > 0 then
    -- falling: check bottom edge (py+7)
    if solid_at(px+1, py+7) or solid_at(px+6, py+7) then
      py  = flr((py+7) / 8) * 8 - 8   -- snap to top of tile
      pvy = 0
    end
  elseif pvy < 0 then
    -- jumping: check top edge (py)
    if solid_at(px+1, py) or solid_at(px+6, py) then
      py  = flr(py / 8) * 8 + 8        -- snap to bottom of tile
      pvy = 0
    end
  end
end
```

### Why 4 check points?

We check two points per direction (e.g. `py+1` and `py+6` for left/right checks) to catch the middle of the player, not just corners. Inset by 1 pixel to avoid false hits on shared tile edges.

---

## Step 3 — The on_ground Flag

You need to know if the player is standing on solid ground before allowing a jump.

```lua
function resolve_y()
  on_ground = false    -- assume not on ground

  if pvy > 0 then
    if solid_at(px+1, py+7) or solid_at(px+6, py+7) then
      py        = flr((py+7) / 8) * 8 - 8
      pvy       = 0
      on_ground = true   -- landed!
    end
  elseif pvy < 0 then
    if solid_at(px+1, py) or solid_at(px+6, py) then
      py  = flr(py / 8) * 8 + 8
      pvy = 0
    end
  end
end
```

---

## Step 4 — Jumping

```lua
jump_force = -4.5

function _update()
  -- horizontal movement
  pvx = 0
  if btn(0) then pvx = -2 end
  if btn(1) then pvx =  2 end

  -- jump (only when on ground)
  if btnp(4) and on_ground then
    pvy = jump_force
    sfx(0)     -- jump sound
  end

  -- gravity
  pvy = min(pvy + gravity, max_fall)

  -- move and resolve
  px += pvx
  resolve_x()
  py += pvy
  resolve_y()
end
```

**Save and run.** You should be able to walk and jump on platforms.

---

## Step 5 — Variable Jump Height

Letting go of the jump button early should make you jump lower. This feels much better:

```lua
-- add this after the jump force is applied, in _update:
if pvy < -1 and not btn(4) then
  pvy *= 0.75    -- cut the jump short if button released
end
```

---

## Step 6 — Coyote Time

"Coyote time" = the player can still jump for a few frames after walking off a ledge. Feels more fair and responsive.

```lua
coyote_max = 6   -- frames of grace

function _init()
  coyote = 0
end

function _update()
  -- update coyote timer
  if on_ground then
    coyote = coyote_max
  else
    coyote = max(0, coyote - 1)
  end

  -- jump: allow if on ground OR coyote frames remain
  if btnp(4) and (on_ground or coyote > 0) then
    pvy    = jump_force
    coyote = 0
    sfx(0)
  end

  -- rest of update...
end
```

---

## Step 7 — Complete Platformer Core

Here's everything together — copy this into a new cart and build on it:

```lua
-- === CONSTANTS ===
gravity    = 0.35
max_fall   = 5
jump_force = -4.5
move_spd   = 2
coyote_max = 6

-- === INIT ===
function _init()
  px        = 24
  py        = 24
  pvx       = 0
  pvy       = 0
  on_ground = false
  coyote    = 0
  facing    = 1      -- 1=right, -1=left
  frame     = 0
  ftimer    = 0
  cam_x     = 0
end

-- === TILE COLLISION ===
function solid_at(wx, wy)
  local tx = flr(wx/8)
  local ty = flr(wy/8)
  if tx<0 or ty<0 or tx>127 or ty>63 then return true end
  return fget(mget(tx, ty), 0)
end

function resolve_x()
  if pvx > 0 then
    if solid_at(px+7, py+1) or solid_at(px+7, py+6) then
      px=flr((px+7)/8)*8-8  pvx=0
    end
  elseif pvx < 0 then
    if solid_at(px, py+1) or solid_at(px, py+6) then
      px=flr(px/8)*8+8  pvx=0
    end
  end
end

function resolve_y()
  on_ground = false
  if pvy > 0 then
    if solid_at(px+1, py+7) or solid_at(px+6, py+7) then
      py=flr((py+7)/8)*8-8  pvy=0  on_ground=true
    end
  elseif pvy < 0 then
    if solid_at(px+1, py) or solid_at(px+6, py) then
      py=flr(py/8)*8+8  pvy=0
    end
  end
end

-- === UPDATE ===
function _update()
  -- horizontal
  pvx = 0
  if btn(0) then pvx = -move_spd  facing = -1 end
  if btn(1) then pvx =  move_spd  facing =  1 end

  -- coyote
  if on_ground then coyote = coyote_max
  else coyote = max(0, coyote - 1) end

  -- jump
  if btnp(4) and (on_ground or coyote > 0) then
    pvy    = jump_force
    coyote = 0
    sfx(0)
  end

  -- short hop
  if pvy < -1 and not btn(4) then pvy *= 0.75 end

  -- gravity
  pvy = min(pvy + gravity, max_fall)

  -- resolve
  px += pvx  resolve_x()
  py += pvy  resolve_y()

  -- animation
  if on_ground and pvx ~= 0 then
    ftimer += 1
    if ftimer >= 8 then ftimer=0  frame=(frame+1)%2 end
  else
    frame=0  ftimer=0
  end

  -- camera
  cam_x = mid(0, px - 60, 128*8 - 128)
end

-- === DRAW ===
function _draw()
  cls()
  camera(cam_x, 0)
  map(0, 0, 0, 0, 128, 16)
  spr(1 + frame, px, py, 1, 1, facing < 0)
  camera()
end
```

---

## Common Platformer Problems & Fixes

| Problem | Fix |
|---------|-----|
| Player gets stuck in corners | Move X and Y separately (already done above) |
| Jumping through thin platforms | Use `pvy > 0` check — only land when falling |
| "Sticky" walls when sliding | Inset collision check points by 1px from edges |
| Jump feels floaty | Increase gravity or decrease jump_force magnitude |
| Jump feels stiff | Add short-hop (multiply pvy when button released early) |
| Can't reach ledge | Add coyote time |
| Spam jumping | Already handled — `on_ground` check |

---

## Challenge

1. Add a **double jump** — allow one extra jump while airborne
2. Add **wall jumping** — if touching a wall while airborne, press jump to jump away
3. Make **hazard tiles** (flag 1 = spikes) that reset the player on touch
4. Add a **collectible** (coin/star) — a table of positions, delete when player overlaps

---

## What You Learned

- Gravity = `pvy += constant` every frame, capped with `min()`
- Jump = set a negative `pvy` when on ground and button pressed
- Tile collision: move X first, resolve, then Y, resolve
- `on_ground` flag gates jumping
- Short hop: cut velocity when jump button released early
- Coyote time: grace frames after walking off a ledge

**Next:** [[07-entities|07 - Entities]] — enemies, bullets, and managing many objects with tables.
