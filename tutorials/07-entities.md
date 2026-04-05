---
tags: [pico8, tutorial]
---
# Tutorial 07 — Entities (Tables of Tables)

**What you'll learn:** Managing enemies, bullets, pickups, and any group of objects using tables of tables — the core pattern for almost every PICO-8 game.

**Prerequisites:** [[06-platformer|06 - Platformer Physics]], and Lua tables from [[basics|Lua Basics]]

---

## The Problem

What if you need 20 enemies on screen at once? You can't make 20 separate variables (`enemy1_x`, `enemy1_y`, `enemy2_x`...). That's unmanageable.

The solution: **a table of tables**. One master table holds all enemies. Each enemy is its own table with its own properties.

```lua
enemies = {}   -- master table

-- each enemy is a table inside it:
enemies[1] = {x=40, y=60, hp=2}
enemies[2] = {x=80, y=60, hp=1}
enemies[3] = {x=120, y=60, hp=3}
```

---

## Step 1 — The Pattern

Every group of objects follows the same structure:

```lua
-- 1. MASTER TABLE (created in _init)
things = {}

-- 2. MAKE function (creates one thing and adds it)
function make_thing(x, y)
  local t = {}        -- new empty table
  t.x = x
  t.y = y
  t.active = true
  add(things, t)      -- add to master table
end

-- 3. UPDATE function (runs logic for one thing)
function update_thing(t)
  t.x += 1
  if t.x > 128 then
    del(things, t)    -- remove from master table
  end
end

-- 4. DRAW function (draws one thing)
function draw_thing(t)
  spr(5, t.x, t.y)
end

-- 5. LOOP in _update and _draw
function _update()
  foreach(things, update_thing)
end

function _draw()
  cls()
  foreach(things, draw_thing)
end
```

`foreach(table, function)` calls the function once for each item in the table, passing the item as the argument.

---

## Step 2 — Deleting While Looping

The most common gotcha: **never use a regular `for` loop when deleting items.** Use `for x in all(t)` instead — it handles deletions safely.

```lua
-- SAFE: use all()
for e in all(enemies) do
  if e.hp <= 0 then
    del(enemies, e)    -- safe to delete while looping
  end
end

-- ALSO SAFE: foreach with del inside
function update_enemy(e)
  if e.hp <= 0 then del(enemies, e) end
end
foreach(enemies, update_enemy)

-- UNSAFE: never do this
for i = 1, #enemies do   -- #enemies changes as you delete!
  if enemies[i].hp <= 0 then
    del(enemies, enemies[i])   -- can skip items
  end
end
```

---

## Step 3 — Enemies

A simple patrolling enemy that reverses direction at walls or ledge edges:

```lua
function _init()
  enemies = {}
  make_enemy(80, 48)
  make_enemy(180, 80)
end

-- solid_at is defined in tutorial 06 (platformer).
-- Include this if you're not building on that cart:
function solid_at(wx, wy)
  local tx = flr(wx/8)
  local ty = flr(wy/8)
  if tx<0 or ty<0 or tx>127 or ty>63 then return true end
  return fget(mget(tx, ty), 0)
end

function make_enemy(x, y)
  local e = {}
  e.x   = x
  e.y   = y
  e.vx  = 1       -- patrol speed (positive = moving right)
  e.hp  = 2
  e.spr = 2       -- enemy sprite number
  add(enemies, e)
end

function update_enemy(e)
  -- move
  e.x += e.vx

  -- check for wall ahead (in direction of movement)
  local front = e.x + (e.vx > 0 and 8 or -1)
  if solid_at(front, e.y + 4) then
    e.vx = -e.vx   -- reverse at wall
  end

  -- check for ledge (no floor ahead)
  local floor_x = e.x + (e.vx > 0 and 7 or 0)
  if not solid_at(floor_x, e.y + 9) then
    e.vx = -e.vx   -- reverse at ledge
  end

  -- remove if dead
  if e.hp <= 0 then del(enemies, e) end
end

function draw_enemy(e)
  spr(e.spr, e.x, e.y, 1, 1, e.vx < 0)   -- flip when moving left
end

-- in _update():
foreach(enemies, update_enemy)

-- in _draw() (inside camera block):
foreach(enemies, draw_enemy)
```

---

## Step 4 — Bullets

Bullets come from the player and travel in a direction until they leave the screen or hit something.

```lua
function _init()
  bullets = {}
  fire_cooldown = 0
end

function shoot(x, y, dx, dy)
  add(bullets, {x=x, y=y, dx=dx, dy=dy})
  sfx(1)
end

function update_bullet(b)
  b.x += b.dx
  b.y += b.dy

  -- delete if off screen
  if b.x < -8 or b.x > 136 or b.y < -8 or b.y > 136 then
    del(bullets, b)
    return
  end

  -- delete if hit solid tile
  if solid_at(b.x, b.y) then
    del(bullets, b)
    -- could spawn a particle here
  end
end

function draw_bullet(b)
  rectfill(b.x, b.y, b.x+2, b.y+4, 10)   -- yellow
end

-- shooting in _update():
fire_cooldown -= 1
if btn(4) and fire_cooldown <= 0 then
  shoot(px+3, py, 0, -5)    -- shoot upward from player center
  fire_cooldown = 10         -- frames between shots
end

foreach(bullets, update_bullet)
```

---

## Step 5 — Collision Between Entities

AABB (box vs box) collision:

```lua
function aabb(ax, ay, aw, ah, bx, by, bw, bh)
  return ax < bx+bw and ax+aw > bx and
         ay < by+bh and ay+ah > by
end
```

### Bullets hitting enemies

```lua
function check_hits()
  for b in all(bullets) do
    for e in all(enemies) do
      if aabb(b.x, b.y, 4, 4, e.x, e.y, 8, 8) then
        e.hp -= 1
        del(bullets, b)
        sfx(2)        -- hit sound
        if e.hp <= 0 then
          del(enemies, e)
          score += 100
        end
        break   -- bullet is gone, stop checking this bullet
      end
    end
  end
end
```

### Player touching enemies

```lua
function check_enemy_collision()
  for e in all(enemies) do
    if aabb(px+1, py+1, 6, 6, e.x, e.y, 8, 8) then
      player_hurt()
      break
    end
  end
end
```

---

## Step 6 — Pickups (Coins, Health, Power-ups)

```lua
function _init()
  coins = {}
  make_coin(64, 48)
  make_coin(96, 64)
end

function make_coin(x, y)
  add(coins, {x=x, y=y, frame=0, timer=0})
end

function update_coin(c)
  -- bob up and down
  c.timer += 1
  -- animate (2 frames)
  if c.timer % 16 < 8 then c.frame = 0
  else c.frame = 1 end
  -- check player overlap
  if aabb(px, py, 8, 8, c.x, c.y-1, 8, 8) then
    score += 10
    sfx(3)
    del(coins, c)
  end
end

function draw_coin(c)
  spr(10 + c.frame, c.x, c.y)   -- sprites 10 and 11 = coin frames
end

foreach(coins, update_coin)
foreach(coins, draw_coin)
```

---

## Step 7 — Spawning Over Time

Spawn enemies on a timer rather than all at once:

```lua
function _init()
  enemies    = {}
  spawn_t    = 0
  spawn_rate = 120   -- frames between spawns (4 seconds)
end

function _update()
  spawn_t -= 1
  if spawn_t <= 0 then
    -- spawn from random position at top
    make_enemy(flr(rnd(15)) * 8, 0)
    spawn_t = spawn_rate
    spawn_rate = max(30, spawn_rate - 5)   -- speed up over time
  end

  foreach(enemies, update_enemy)
end
```

---

## Step 8 — Object Pooling (Token Saving)

If you're hitting the 8192 token limit, reuse objects instead of creating and deleting them:

```lua
-- pre-create a pool of bullets
function _init()
  bullets = {}
  for i = 1, 20 do
    add(bullets, {x=0, y=0, dx=0, dy=0, active=false})
  end
end

function shoot(x, y, dx, dy)
  -- find an inactive bullet and reuse it
  for b in all(bullets) do
    if not b.active then
      b.x, b.y   = x, y
      b.dx, b.dy = dx, dy
      b.active   = true
      return
    end
  end
  -- if no free bullet found, do nothing (pool full)
end

function update_bullet(b)
  if not b.active then return end
  b.x += b.dx
  b.y += b.dy
  if b.y < -8 then b.active = false end
end
```

---

## Full Entity System Example

```lua
function _init()
  enemies = {}
  bullets = {}
  coins   = {}
  score   = 0

  -- place some enemies and coins
  for i = 1, 5 do
    make_enemy(i*20, 48)
    make_coin(i*20 + 10, 40)
  end
end

-- (use make_enemy, update_enemy, draw_enemy from above)
-- (use make_coin, update_coin, draw_coin from above)
-- (use shoot, update_bullet, draw_bullet from above)
-- (use check_hits from above)

function _update()
  -- player
  update_player()   -- from tutorial 06

  -- shoot
  fire_cooldown -= 1
  if btnp(4) and fire_cooldown <= 0 then
    shoot(px+3, py-1, 0, -5)
    fire_cooldown = 10
  end

  -- entities
  foreach(enemies, update_enemy)
  foreach(bullets, update_bullet)
  foreach(coins,   update_coin)

  -- collision
  check_hits()
  check_enemy_collision()
end

function _draw()
  cls()
  camera(cam_x, 0)
  map(0, 0, 0, 0, 128, 16)
  foreach(enemies, draw_enemy)
  foreach(coins,   draw_coin)
  foreach(bullets, draw_bullet)
  draw_player()
  camera()

  -- hud
  print("score:"..score, 2, 2, 7)
end
```

---

## Challenge

1. Add an enemy that **shoots back** at the player on a timer
2. Add a **health pickup** that only appears when hp is below half
3. Make enemies **drop a coin** when killed
4. Add a **score multiplier** — consecutive kills in quick succession multiply the score

---

## What You Learned

- Table of tables is the pattern for all groups of objects
- `make_X()` creates one entity and adds it to the master table
- `update_X()` handles logic for one entity
- `draw_X()` draws one entity
- `foreach()` calls a function for every item in a table
- `for x in all(t) do ... del(t, x) end` is the safe way to delete while looping
- AABB collision: check if two rectangles overlap
- Separate collision functions keep the code clean

**Next:** [[08-procedural|08 - Procedural Generation]] — random terrain, caves, dungeons, and noise.
