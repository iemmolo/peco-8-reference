# Collision Snippets

## AABB Box Collision

Axis-Aligned Bounding Box — the standard for tile-based games.

```lua
-- returns true if two rectangles overlap
function aabb(ax, ay, aw, ah, bx, by, bw, bh)
  return ax < bx+bw and
         ax+aw > bx and
         ay < by+bh and
         ay+ah > by
end

-- usage
if aabb(px, py, 8, 8, ex, ey, 8, 8) then
  -- player and enemy are touching
end
```

---

## Circle Collision (by distance)

```lua
function circle_collide(ax, ay, ar, bx, by, br)
  local dx = bx - ax
  local dy = by - ay
  return dx^2 + dy^2 < (ar + br)^2
end

-- usage (using dist^2 avoids sqrt for perf)
if circle_collide(px+4, py+4, 4, ex+4, ey+4, 4) then
  -- hit!
end
```

---

## Tile Collision (Full Platform System)

The core of any platformer. Resolves collisions on X and Y separately.

```lua
-- is this map tile solid?
function solid(tx, ty)
  if tx < 0 or tx > 127 or ty < 0 or ty > 127 then
    return true  -- treat out-of-bounds as solid
  end
  return fget(mget(tx, ty), 0)
end

-- is a world pixel position inside a solid tile?
function solid_at(wx, wy)
  return solid(flr(wx/8), flr(wy/8))
end

-- check and resolve player collision
-- player is 8x8, pvx/pvy are velocities
function resolve_tiles()
  -- === HORIZONTAL ===
  px += pvx

  if pvx > 0 then
    -- moving right — check right edge
    if solid_at(px+7, py+1) or solid_at(px+7, py+6) then
      px = flr((px+7)/8)*8 - 8  -- snap to tile left edge
      pvx = 0
    end
  elseif pvx < 0 then
    -- moving left — check left edge
    if solid_at(px, py+1) or solid_at(px, py+6) then
      px = flr(px/8)*8 + 8       -- snap to tile right edge
      pvx = 0
    end
  end

  -- === VERTICAL ===
  py += pvy

  if pvy > 0 then
    -- moving down — check bottom edge
    if solid_at(px+1, py+7) or solid_at(px+6, py+7) then
      py = flr((py+7)/8)*8 - 8  -- snap to tile top edge
      pvy = 0
      on_ground = true
    end
  elseif pvy < 0 then
    -- moving up — check top edge
    if solid_at(px+1, py) or solid_at(px+6, py) then
      py = flr(py/8)*8 + 8       -- snap to tile bottom edge
      pvy = 0
    end
  end

  -- update on_ground: not on ground if not against a floor tile
  if not (solid_at(px+1, py+8) or solid_at(px+6, py+8)) then
    on_ground = false
  end
end
```

---

## One-Way Platforms

Pass through from below, land on from above.

```lua
-- flag 1 = one-way platform
function check_platform()
  -- only when falling and feet are near top of platform
  if pvy > 0 then
    local bx = flr((px+1)/8)
    local by = flr((py+7)/8)

    if fget(mget(bx, by), 1) or fget(mget(bx+1, by), 1) then
      -- check feet were above tile last frame
      if (py + 7 - pvy) < by * 8 then
        py = by * 8 - 8
        pvy = 0
        on_ground = true
      end
    end
  end
end
```

---

## Hazard Tiles

```lua
-- flag 1 = hazard (spikes, lava, etc.)
function check_hazards()
  local feet_x = flr((px+4)/8)
  local feet_y = flr((py+6)/8)
  if fget(mget(feet_x, feet_y), 1) then
    player_die()
  end
end
```

---

## Bullet vs Enemy

```lua
-- bullets = table of {x,y,dx,dy}
-- enemies = table of {x,y,hp}

function check_bullet_hits()
  for b in all(bullets) do
    for e in all(enemies) do
      if aabb(b.x-2, b.y-2, 4, 4, e.x, e.y, 8, 8) then
        e.hp -= 1
        del(bullets, b)
        if e.hp <= 0 then
          del(enemies, e)
          score += 100
        end
        break
      end
    end
  end
end
```

---

## Pickup Collection

```lua
-- coins = table of {x,y,collected}
function check_pickups()
  for coin in all(coins) do
    if aabb(px, py, 8, 8, coin.x, coin.y, 8, 8) then
      score += 10
      sfx(2)
      del(coins, coin)
    end
  end
end
```

---

## Push Entities Apart (Circle Separation)

```lua
function separate_entities(a, b, radius)
  local dx = b.x - a.x
  local dy = b.y - a.y
  local d = sqrt(dx^2 + dy^2)
  if d < radius and d > 0 then
    local push = (radius - d) / 2
    a.x -= (dx/d) * push
    a.y -= (dy/d) * push
    b.x += (dx/d) * push
    b.y += (dy/d) * push
  end
end
```
