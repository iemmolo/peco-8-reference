# Particle System Snippets

## Basic Particle Fountain

From the Gamedev with PICO-8 zine — clean, reusable system.

```lua
-- === SETUP in _init() ===
function _init()
  ps = {}           -- particle table
  g = 0.1           -- gravity
  max_vel = 2       -- max initial speed
  min_life = 90     -- particle lifetime (frames)
  max_life = 120
  cols = {1,1,1,13,13,12,12,7}  -- color gradient (dark -> bright)
end

-- === ADD PARTICLE ===
function add_p(x, y)
  local p = {}
  p.x = x
  p.y = y
  p.dx = rnd(max_vel) - max_vel/2    -- random horizontal
  p.dy = rnd(max_vel) * -1           -- always upward
  p.life_start = rndb(min_life, max_life)
  p.life = p.life_start
  add(ps, p)
end

-- === UPDATE PARTICLE ===
function update_p(p)
  if p.life <= 0 then
    del(ps, p)
  else
    p.dy += g
    if p.y + p.dy > 127 then p.dy *= -0.8 end  -- bounce off floor
    p.x += p.dx
    p.y += p.dy
    p.life -= 1
  end
end

-- === DRAW PARTICLE ===
function draw_p(p)
  local ci = flr(p.life / p.life_start * #cols) + 1
  pset(p.x, p.y, cols[ci])
end

-- helper
function rndb(lo, hi)
  return flr(rnd(hi - lo + 1) + lo)
end

-- === IN GAME LOOP ===
function _update()
  add_p(64, 80)             -- emit one particle per frame
  foreach(ps, update_p)
end

function _draw()
  cls()
  foreach(ps, draw_p)
end
```

---

## Burst on Event

```lua
function explode(x, y, count)
  count = count or 20
  for i = 1, count do
    local p = {}
    p.x = x
    p.y = y
    p.dx = rnd(4) - 2
    p.dy = rnd(4) - 2
    p.life = rndb(20, 40)
    p.col = rnd({8, 9, 10, 7})  -- red, orange, yellow, white
    add(ps, p)
  end
end

function update_p(p)
  if p.life <= 0 then
    del(ps, p)
  else
    p.dy += 0.05   -- light gravity
    p.x  += p.dx
    p.y  += p.dy
    p.life -= 1
  end
end

function draw_p(p)
  pset(p.x, p.y, p.col)
end

-- trigger:
explode(px, py, 30)
sfx(3)
```

---

## Dust / Landing Particles

```lua
function land_dust(x, y)
  for i = 1, 8 do
    local p = {}
    p.x = x + rnd(8)
    p.y = y + 7
    p.dx = rnd(3) - 1.5
    p.dy = -(rnd(1.5))
    p.life = rndb(6, 12)
    p.col = 5   -- dark gray
    add(ps, p)
  end
end
```

---

## Trail Effect

```lua
-- call every frame when player is moving
function emit_trail(x, y)
  if rnd(1) < 0.5 then return end  -- 50% chance per frame
  local p = {}
  p.x = x + rnd(8)
  p.y = y + rnd(8)
  p.dx = 0
  p.dy = 0
  p.life = rndb(4, 10)
  p.col = 7  -- white
  add(ps, p)
end

function update_trail_p(p)
  p.life -= 1
  if p.life <= 0 then del(ps, p) end
  -- fade color as life decreases
  if p.life > 6 then p.col = 7
  elseif p.life > 3 then p.col = 6
  else p.col = 5 end
end
```

---

## Rain

```lua
function _init()
  rain = {}
  for i = 1, 40 do
    add(rain, {
      x = rnd(128),
      y = rnd(128),
      spd = rndb(2, 5)
    })
  end
end

function update_rain()
  for r in all(rain) do
    r.y += r.spd
    r.x -= r.spd * 0.2  -- slight angle
    if r.y > 128 then
      r.y = -2
      r.x = rnd(128)
    end
  end
end

function draw_rain()
  for r in all(rain) do
    line(r.x, r.y, r.x - 1, r.y + 2, 1)  -- dark blue lines
  end
end
```

---

## Stars (Background)

```lua
-- static stars that don't move (seeded with srand so same each frame)
function draw_stars()
  srand(42)
  for i = 1, 60 do
    local x = rnd(128)
    local y = rnd(128)
    local c = rnd({5, 6, 7})
    pset(x, y, c)
  end
  srand(time())   -- restore random to time-based
end
```

---

## Scrolling Stars (Parallax)

```lua
function _init()
  stars = {}
  for i = 1, 50 do
    add(stars, {
      x = rnd(128),
      y = rnd(128),
      spd = rnd(1) + 0.2,    -- different speeds = depth
      col = rnd({5, 6, 7})
    })
  end
end

function update_stars(scroll_spd)
  for s in all(stars) do
    s.x -= s.spd * scroll_spd
    if s.x < 0 then s.x = 128 end
  end
end

function draw_stars()
  for s in all(stars) do
    pset(s.x, s.y, s.col)
  end
end
```
