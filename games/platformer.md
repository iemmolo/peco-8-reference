# Platformer

Full annotated platformer with gravity, tile collision, and enemies.

---

## Quick Tips

- Separate X and Y movement/collision (never do both at once)
- `on_ground` flag gates jumping
- Sprite flag 0 = solid tile
- Use `mid()` to clamp velocity
- Enemy AI: simple patrol (reverse on wall/edge)

---

## Map Setup

In the sprite editor:
- Draw solid tiles, set **flag 0** on them
- Sprite 0 = empty (always)
- Sprite 1 = player
- Sprite 2 = enemy

In the map editor:
- Paint tiles to build level
- Enemies can be hard-coded positions or read from map flags

---

## Full Cartridge

```lua
-- === CONSTANTS ===

gravity  = 0.35
jump_f   = -4.5
max_fall = 5
move_spd = 2

-- === INIT ===

function _init()
  make_player()
  make_enemies()
  cam_x = 0
  score = 0
  music(0)
end

-- === PLAYER ===

function make_player()
  p = {}
  p.x  = 16
  p.y  = 16
  p.vx = 0
  p.vy = 0
  p.on_ground = false
  p.hp = 3
  p.inv = 0    -- invincibility frames
  p.dir = 1    -- facing: 1=right, -1=left
end

function update_player()
  -- horizontal input
  p.vx = 0
  if btn(0) then p.vx = -move_spd  p.dir = -1 end
  if btn(1) then p.vx =  move_spd  p.dir =  1 end

  -- jump
  if btnp(4) and p.on_ground then
    p.vy = jump_f
    sfx(0)   -- jump sound
  end

  -- gravity
  p.vy = min(p.vy + gravity, max_fall)

  -- move X then resolve tiles
  p.x += p.vx
  resolve_x()

  -- move Y then resolve tiles
  p.y += p.vy
  resolve_y()

  -- invincibility countdown
  if p.inv > 0 then p.inv -= 1 end
end

-- === TILE COLLISION ===

function solid(tx, ty)
  if tx < 0 or ty < 0 or tx > 127 or ty > 63 then
    return true  -- world edges are solid
  end
  return fget(mget(tx, ty), 0)
end

function solid_at(wx, wy)
  return solid(flr(wx/8), flr(wy/8))
end

function resolve_x()
  if p.vx > 0 then
    -- moving right: check right edge
    if solid_at(p.x+7, p.y+1) or solid_at(p.x+7, p.y+6) then
      p.x  = flr((p.x+7)/8)*8 - 8
      p.vx = 0
    end
  elseif p.vx < 0 then
    -- moving left: check left edge
    if solid_at(p.x, p.y+1) or solid_at(p.x, p.y+6) then
      p.x  = flr(p.x/8)*8 + 8
      p.vx = 0
    end
  end
end

function resolve_y()
  if p.vy > 0 then
    -- falling: check bottom
    if solid_at(p.x+1, p.y+7) or solid_at(p.x+6, p.y+7) then
      p.y         = flr((p.y+7)/8)*8 - 8
      p.vy        = 0
      p.on_ground = true
    end
  elseif p.vy < 0 then
    -- rising: check top
    if solid_at(p.x+1, p.y) or solid_at(p.x+6, p.y) then
      p.y  = flr(p.y/8)*8 + 8
      p.vy = 0
    end
  end

  -- check if still on ground
  if not (solid_at(p.x+1, p.y+8) or solid_at(p.x+6, p.y+8)) then
    p.on_ground = false
  end
end

-- === ENEMIES ===

function make_enemies()
  enemies = {
    {x=80,  y=40, vx=1, hp=2},
    {x=180, y=80, vx=-1, hp=2},
    {x=300, y=32, vx=1, hp=3},
  }
end

function update_enemies()
  for e in all(enemies) do
    -- patrol: move and flip at walls/edges
    e.x += e.vx

    local front_x = (e.vx > 0) and (e.x+8) or e.x
    local floor_x = (e.vx > 0) and (e.x+7) or e.x

    -- reverse at wall
    if solid_at(front_x, e.y+4) then
      e.vx = -e.vx
    end
    -- reverse at edge (no floor ahead)
    if not solid_at(floor_x, e.y+9) then
      e.vx = -e.vx
    end

    -- fall (simple gravity — no resolve, just clamp to floor)
    if not solid_at(e.x+4, e.y+9) then
      e.y = min(e.y + 2, 127)
    end

    -- check if player jumped on top
    if e.hp > 0 then
      if aabb(p.x, p.y, 8, 8, e.x, e.y, 8, 8) then
        if p.y + 7 < e.y + 4 and p.vy > 0 then
          -- stomped!
          e.hp -= 1
          p.vy = -3
          sfx(2)
          if e.hp <= 0 then
            del(enemies, e)
            score += 100
          end
        else
          -- player took damage
          player_hit()
        end
      end
    end
  end
end

-- === DAMAGE ===

function player_hit()
  if p.inv > 0 then return end
  p.hp  -= 1
  p.inv  = 60    -- 2 seconds invincible
  p.vy   = -3    -- bounce up
  sfx(3)
  if p.hp <= 0 then
    gameover_init()
  end
end

-- === AABB ===

function aabb(ax, ay, aw, ah, bx, by, bw, bh)
  return ax < bx+bw and ax+aw > bx and
         ay < by+bh and ay+ah > by
end

-- === CAMERA ===

function update_camera()
  cam_x = p.x - 60
  cam_x = mid(0, cam_x, 128*8 - 128)   -- clamp to world width
end

-- === DRAW ===

function draw_player()
  if p.inv > 0 and p.inv % 4 < 2 then return end  -- flicker when invincible
  local flip_x = (p.dir < 0)
  spr(1, p.x, p.y, 1, 1, flip_x)
end

function draw_enemies()
  for e in all(enemies) do
    spr(2, e.x, e.y, 1, 1, e.vx < 0)
  end
end

function draw_hud()
  camera()    -- reset camera before HUD
  for i = 1, p.hp do
    spr(3, 2 + (i-1)*9, 2)   -- draw heart sprites
  end
  print("score:"..score, 80, 2, 7)
end

-- === GAME STATES ===

function _update()
  update_player()
  update_enemies()
  update_camera()
end

function _draw()
  cls()
  camera(cam_x, 0)
  map(0, 0, 0, 0, 128, 16)   -- draw map
  draw_enemies()
  draw_player()
  draw_hud()
end

-- === GAME OVER (simple version) ===

function gameover_init()
  _update = gameover_update
  _draw   = gameover_draw
  music(-1)
end

function gameover_update()
  if btnp(4) then _init() end
end

function gameover_draw()
  cls()
  print("game over", 44, 56, 8)
  print("score: "..score, 42, 68, 7)
  print("press \x8e to retry", 28, 80, 6)
end
```

---

## Extension Ideas

### Double Jump
```lua
p.jumps = 0     -- in make_player()
max_jumps = 2

-- in update_player() jump section:
if btnp(4) and p.jumps < max_jumps then
  p.vy = jump_f
  p.jumps += 1
  sfx(0)
end
if p.on_ground then p.jumps = 0 end
```

### Coyote Time (grace frames after walking off ledge)
```lua
p.coyote = 0    -- in make_player()

-- in update_player():
if p.on_ground then
  p.coyote = 6   -- 6 frames of grace
else
  p.coyote = max(0, p.coyote - 1)
end

if btnp(4) and (p.on_ground or p.coyote > 0) then
  p.vy = jump_f
  p.coyote = 0
end
```

### Variable Jump Height (let go early = lower jump)
```lua
-- in update_player():
if not btn(4) and p.vy < -1 then
  p.vy *= 0.75   -- cut jump short if button released
end
```

### Wall Jump
```lua
-- check if touching a wall
p.wall = 0
if solid_at(p.x-1, p.y+2) then p.wall = -1 end
if solid_at(p.x+8, p.y+2) then p.wall =  1 end

if btnp(4) and p.wall ~= 0 and not p.on_ground then
  p.vx = -p.wall * 3
  p.vy = jump_f
  sfx(0)
end
```
