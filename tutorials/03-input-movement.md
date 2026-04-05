# Tutorial 03 — Input & Movement

**What you'll learn:** btn() vs btnp(), velocity, friction, acceleration, screen bounds, and wrapping.

**Prerequisites:** [02-sprites-animation.md](02-sprites-animation.md)

---

## Two Input Functions

```lua
btn(b)    -- true EVERY frame the button is held
btnp(b)   -- true ONLY on the first frame the button is pressed
```

### Button numbers

```
btn(0) = LEFT     btn(4) = O  (confirm, jump, shoot)
btn(1) = RIGHT    btn(5) = X  (cancel, action)
btn(2) = UP
btn(3) = DOWN
```

### When to use each

| Use `btn()` for | Use `btnp()` for |
|----------------|-----------------|
| Movement (held) | Jump (once per press) |
| Holding fire | Menu select |
| Scrolling | Toggle options |
| Charging | Opening/closing |

```lua
-- movement: btn (held down)
if btn(0) then px -= 2 end

-- jump: btnp (only triggers once per press)
if btnp(4) then
  vy = -5
end
```

---

## Step 1 — Direct Movement (Simplest)

No velocity — just move the position directly based on input:

```lua
function _init()
  px = 60
  py = 60
  spd = 2
end

function _update()
  if btn(0) then px -= spd end
  if btn(1) then px += spd end
  if btn(2) then py -= spd end
  if btn(3) then py += spd end
end

function _draw()
  cls()
  spr(1, px, py)
end
```

Feels a bit stiff and instant. Good for top-down tiles, menus, Tetris-style games.

---

## Step 2 — Velocity-Based Movement

Instead of moving directly, change a **velocity** variable and add it to position. This gives you the foundation for friction, acceleration, and physics.

```lua
function _init()
  px  = 60
  py  = 60
  pvx = 0     -- velocity x
  pvy = 0     -- velocity y
  spd = 3
end

function _update()
  -- set velocity from input
  pvx = 0
  pvy = 0
  if btn(0) then pvx = -spd end
  if btn(1) then pvx =  spd end
  if btn(2) then pvy = -spd end
  if btn(3) then pvy =  spd end

  -- apply velocity to position
  px += pvx
  py += pvy
end

function _draw()
  cls()
  spr(1, px, py)
end
```

Same result for now, but the pattern is important — everything builds on this.

---

## Step 3 — Screen Clamping

Keep the player on screen:

```lua
-- after moving:
px = mid(0, px, 120)    -- 120 = 128 - 8 (sprite is 8px wide)
py = mid(0, py, 120)    -- same for height
```

`mid(lo, val, hi)` clamps `val` between `lo` and `hi`. Much cleaner than if/else.

---

## Step 4 — Screen Wrapping

Appear on the opposite side when you leave:

```lua
-- wrap horizontally
if px > 127 then px = -8 end   -- went off right → appear on left
if px < -8  then px = 127 end  -- went off left  → appear on right

-- or the one-liner (wrap 0..127):
px = (px + 128) % 128
py = (py + 128) % 128
```

---

## Step 5 — Friction (Slippery / Momentum)

Multiply velocity by a number less than 1 each frame. Velocity decays toward zero.

```lua
function _init()
  px  = 60
  py  = 60
  pvx = 0
  pvy = 0
  accel   = 0.6   -- how fast you speed up
  friction = 0.8  -- how fast you slow down (lower = more ice)
  max_spd  = 3
end

function _update()
  -- accelerate from input
  if btn(0) then pvx -= accel end
  if btn(1) then pvx += accel end
  if btn(2) then pvy -= accel end
  if btn(3) then pvy += accel end

  -- clamp to max speed
  pvx = mid(-max_spd, pvx, max_spd)
  pvy = mid(-max_spd, pvy, max_spd)

  -- apply friction
  pvx *= friction
  pvy *= friction

  -- stop sliding when very slow
  if abs(pvx) < 0.1 then pvx = 0 end
  if abs(pvy) < 0.1 then pvy = 0 end

  -- apply to position
  px += pvx
  py += pvy

  -- clamp to screen
  px = mid(0, px, 120)
  py = mid(0, py, 120)
end
```

Try different `friction` values:
- `0.9` = light sliding (normal)
- `0.75` = ice rink
- `0.5` = very slippery
- `1.0` = no friction (keeps going forever)

---

## Step 6 — Angle-Based Movement (Top-Down Car/Ship)

Instead of moving in cardinal directions, the player rotates and thrusts:

```lua
function _init()
  sx    = 64      -- ship x
  sy    = 64      -- ship y
  svx   = 0       -- ship velocity x
  svy   = 0       -- ship velocity y
  angle = 0       -- direction (0..1 = full circle)
  turn  = 0.02    -- turn speed
  thrust = 0.15   -- acceleration
  drag  = 0.96    -- friction (close to 1 = low drag)
end

function _update()
  -- turn
  if btn(0) then angle -= turn end
  if btn(1) then angle += turn end

  -- thrust forward
  if btn(2) then
    svx += cos(angle) * thrust
    svy -= sin(angle) * thrust   -- sin is inverted in PICO-8
  end

  -- drag
  svx *= drag
  svy *= drag

  -- move
  sx += svx
  sy += svy

  -- wrap screen
  sx = (sx + 128) % 128
  sy = (sy + 128) % 128
end

function _draw()
  cls(1)
  -- draw ship as a rotated triangle (simple version: just a sprite)
  spr(1, sx - 4, sy - 4)
  -- draw direction indicator
  local tip_x = sx + cos(angle) * 8
  local tip_y = sy - sin(angle) * 8
  line(sx, sy, tip_x, tip_y, 7)
end
```

> **Note on sin/cos:** PICO-8 angles are 0..1 (not degrees or radians). `cos(0)` points right. `sin()` is y-inverted, so we negate it when moving up the screen.

---

## Step 7 — 8-Direction Normalized Movement

Moving diagonally with pure cardinal input is ~40% faster than straight movement. Fix it:

```lua
function _update()
  local dx, dy = 0, 0
  if btn(0) then dx -= 1 end
  if btn(1) then dx += 1 end
  if btn(2) then dy -= 1 end
  if btn(3) then dy += 1 end

  -- normalize diagonal
  if dx ~= 0 and dy ~= 0 then
    dx *= 0.707   -- 1/sqrt(2)
    dy *= 0.707
  end

  px += dx * spd
  py += dy * spd
end
```

---

## Step 8 — Shooting

Use `btnp()` for shooting so holding the button doesn't spam bullets:

```lua
function _init()
  px = 60
  py = 100
  bullets = {}
  fire_delay = 0
end

function _update()
  -- move player
  if btn(0) then px -= 2 end
  if btn(1) then px += 2 end

  -- shoot: use a cooldown timer instead of btnp for rapid fire
  fire_delay -= 1
  if btn(4) and fire_delay <= 0 then
    add(bullets, {x=px+3, y=py, dy=-5})
    fire_delay = 8
  end

  -- update bullets
  for b in all(bullets) do
    b.y += b.dy
    if b.y < -4 then del(bullets, b) end
  end
end

function _draw()
  cls(1)
  spr(1, px, py)
  for b in all(bullets) do
    rectfill(b.x, b.y, b.x+1, b.y+4, 10)   -- yellow bullet
  end
end
```

---

## Common Movement Patterns at a Glance

```lua
-- TOP-DOWN: direct, 4-dir
px += (btn(1) and 2 or 0) - (btn(0) and 2 or 0)
py += (btn(3) and 2 or 0) - (btn(2) and 2 or 0)

-- PLATFORMER: gravity (covered in tutorial 06)
pvy += 0.35         -- gravity each frame
if btnp(4) then pvy = -4.5 end

-- SHOOTER: move left/right, stay on y axis
if btn(0) then px = max(0,   px - 2) end
if btn(1) then px = min(120, px + 2) end
```

---

## Challenge

1. Add a fire rate mechanic — holding O fires faster the longer you hold it (up to a limit)
2. Make the friction value change dynamically — lower on ice tiles, higher on mud
3. Make the ship leave a particle trail when thrusting (hint: use a table of positions)
4. Add a speed boost: press X to dash (temporarily multiply velocity by 3 for 10 frames)

---

## What You Learned

- `btn()` for held input, `btnp()` for single-press input
- Velocity variables let you add physics effects
- `mid()` is your best friend for clamping
- Friction = multiply velocity by a value < 1 each frame
- `cos(angle)` / `-sin(angle)` for angle-based movement
- Normalize diagonal movement with `* 0.707`
- Cooldown timers for shooting rate

**Next:** [04-tilemaps-camera.md](04-tilemaps-camera.md) — drawing the map, making your player walk through it, and scrolling the camera.
