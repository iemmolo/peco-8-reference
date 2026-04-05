# Movement Snippets

## 4-Direction (Top-Down)

```lua
-- setup
spd = 2

function move_player()
  local dx, dy = 0, 0
  if btn(0) then dx = -1 end
  if btn(1) then dx =  1 end
  if btn(2) then dy = -1 end
  if btn(3) then dy =  1 end
  px += dx * spd
  py += dy * spd
end
```

---

## 8-Direction (Normalized Diagonals)

Without normalization, diagonal movement is ~40% faster. Fix it:

```lua
function move_player()
  local dx, dy = 0, 0
  if btn(0) then dx -= 1 end
  if btn(1) then dx += 1 end
  if btn(2) then dy -= 1 end
  if btn(3) then dy += 1 end

  if dx ~= 0 and dy ~= 0 then
    -- 0.707 ≈ 1/sqrt(2)
    dx *= 0.707
    dy *= 0.707
  end

  px += dx * spd
  py += dy * spd
end
```

---

## Platformer (Gravity + Jump)

```lua
-- setup in _init()
function make_player()
  px = 60
  py = 60
  pvx = 0      -- horizontal velocity
  pvy = 0      -- vertical velocity
  on_ground = false
end

-- constants
gravity = 0.3
jump_force = -4
max_fall = 5
move_spd = 2

function move_player()
  -- horizontal
  pvx = 0
  if btn(0) then pvx = -move_spd end
  if btn(1) then pvx =  move_spd end

  -- jump (only when on ground)
  if btnp(4) and on_ground then
    pvy = jump_force
  end

  -- gravity
  pvy = min(pvy + gravity, max_fall)

  -- apply movement
  px += pvx
  py += pvy
end
```

---

## Friction / Deceleration

Good for ice levels, inertia, drifty controls:

```lua
-- setup
pvx = 0
friction = 0.85   -- lower = more slippery (0.7-0.95 range)
accel = 0.5

function move_player()
  if btn(0) then pvx -= accel end
  if btn(1) then pvx += accel end

  -- clamp speed
  pvx = mid(-max_spd, pvx, max_spd)

  -- apply friction
  pvx *= friction

  -- stop tiny sliding
  if abs(pvx) < 0.1 then pvx = 0 end

  px += pvx
end
```

---

## Acceleration (Momentum-Based)

```lua
function make_player()
  px, py = 64, 64
  pvx, pvy = 0, 0
  max_spd = 3
  accel   = 0.4
  decel   = 0.8   -- friction when not pressing
end

function move_player()
  -- horizontal input
  if btn(0) then
    pvx = max(pvx - accel, -max_spd)
  elseif btn(1) then
    pvx = min(pvx + accel, max_spd)
  else
    pvx *= decel  -- slow down when no input
  end

  -- vertical input
  if btn(2) then
    pvy = max(pvy - accel, -max_spd)
  elseif btn(3) then
    pvy = min(pvy + accel, max_spd)
  else
    pvy *= decel
  end

  px += pvx
  py += pvy
end
```

---

## Follow / Chase (Enemy AI)

```lua
-- enemy chases player
function update_enemy(e)
  local dx = px - e.x
  local dy = py - e.y
  local dist = sqrt(dx^2 + dy^2)

  if dist > 2 then
    e.x += (dx / dist) * e.spd
    e.y += (dy / dist) * e.spd
  end
end
```

---

## Angle-Based Movement

```lua
-- ship/top-down car that turns and thrusts
function make_ship()
  sx, sy = 64, 64
  svx, svy = 0, 0
  sangle = 0      -- direction (0..1)
  turn_spd = 0.02
  thrust = 0.1
  drag = 0.95
end

function move_ship()
  -- turn
  if btn(0) then sangle -= turn_spd end
  if btn(1) then sangle += turn_spd end

  -- thrust
  if btn(2) then
    svx += cos(sangle) * thrust
    svy -= sin(sangle) * thrust
  end

  -- drag
  svx *= drag
  svy *= drag

  sx += svx
  sy += svy
end
```

---

## Screen Wrap

```lua
-- wrap entity to opposite side when it leaves
px = (px + 128) % 128
py = (py + 128) % 128
```

---

## Screen Clamp (Stay on Screen)

```lua
px = mid(0, px, 120)    -- 120 = 128-8 (sprite width)
py = mid(0, py, 120)
```

---

## Approach (Move Toward Target, No Overshoot)

```lua
function approach(v, target, step)
  if v < target then return min(v + step, target)
  else               return max(v - step, target)
  end
end

-- use to smoothly accelerate/decelerate
spd = approach(spd, target_spd, 0.2)
```
