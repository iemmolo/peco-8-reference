# Math Helper Snippets

## Clamp

```lua
-- keep x between lo and hi
function clamp(x, lo, hi)
  return mid(lo, x, hi)
end

-- built-in shorthand (no function needed)
x = mid(0, x, 127)
```

---

## Lerp (Linear Interpolation)

```lua
-- move a toward b by fraction t (0=stay, 1=jump to b)
function lerp(a, b, t)
  return a + (b - a) * t
end

-- smooth camera
cam_x += (target_x - cam_x) * 0.1

-- fade color
local c = flr(lerp(8, 11, hp / max_hp))  -- red to green health bar
```

---

## Distance

```lua
function dist(x1, y1, x2, y2)
  return sqrt((x2-x1)^2 + (y2-y1)^2)
end

-- faster for comparisons (no sqrt)
function dist_sq(x1, y1, x2, y2)
  return (x2-x1)^2 + (y2-y1)^2
end

-- example: check within 8 pixels
if dist_sq(px, py, ex, ey) < 64 then  -- 8^2 = 64
  -- close enough
end
```

---

## Angle Between Points

```lua
-- returns angle as 0..1 (PICO-8 range)
function angle_to(sx, sy, tx, ty)
  return atan2(tx - sx, ty - sy)
end

-- move toward target
local a = angle_to(sx, sy, tx, ty)
sx += cos(a) * spd
sy -= sin(a) * spd    -- negate sin for screen coords
```

---

## Normalize Direction

```lua
-- normalize dx,dy to unit length (length=1)
function normalize(dx, dy)
  local d = sqrt(dx^2 + dy^2)
  if d == 0 then return 0, 0 end
  return dx/d, dy/d
end

-- usage
local nx, ny = normalize(tx-sx, ty-sy)
sx += nx * spd
sy += ny * spd
```

---

## Approach (No Overshoot)

```lua
function approach(v, target, step)
  if v < target then
    return min(v + step, target)
  else
    return max(v - step, target)
  end
end

-- examples
spd = approach(spd, max_spd, 0.3)
hp  = approach(hp, 0, 1)
```

---

## Wrap (Loop a Value)

```lua
-- wrap x within 0..max (exclusive)
function wrap(x, max)
  return x % max
end

-- screen wrap
px = (px + 128) % 128
```

---

## Random Between Two Values

```lua
function rndb(lo, hi)
  return flr(rnd(hi - lo + 1) + lo)
end

rndb(1, 6)      -- 1 to 6 (like a d6)
rndb(-5, 5)     -- -5 to 5
rndb(0, 127)    -- random screen x
```

---

## Sign

```lua
-- returns -1, 0, or 1
function sign(x)
  if x > 0 then return 1
  elseif x < 0 then return -1
  else return 0 end
end

-- built-in sgn() returns 1 for 0, not 0
sgn(-5)  -- -1
sgn(5)   -- 1
sgn(0)   -- 1 (built-in quirk)
```

---

## Oscillate (Sine Wave)

```lua
-- smooth back-and-forth value between lo and hi
function oscillate(lo, hi, spd)
  local t = time() * spd
  local s = (sin(t) + 1) / 2  -- 0..1 range (sin is -1..1 in normal math)
                               -- but PICO-8 sin is inverted, so use:
  s = (1 - sin(t)) / 2        -- correct for PICO-8 inverted sin
  return lo + (hi - lo) * s
end

-- usage: bob a sprite up and down
local bob = oscillate(-3, 3, 0.5)
spr(1, px, py + bob)

-- simpler version
local bob = sin(time() * 0.5) * 3  -- swings -3 to 3 (inverted)
```

---

## Timer

```lua
function _init()
  timers = {}
end

function after(frames, callback)
  add(timers, {t=frames, fn=callback})
end

function update_timers()
  for t in all(timers) do
    t.t -= 1
    if t.t <= 0 then
      t.fn()
      del(timers, t)
    end
  end
end

-- usage
after(60, function()     -- 2 seconds at 30fps
  spawn_enemy()
end)
```

---

## Coin / Score Pop (Formatted Number)

```lua
-- display large numbers with commas (e.g. 12,345)
function format_score(n)
  local s = tostr(n)
  local result = ""
  local len = #s
  for i = 1, len do
    if i > 1 and (len - i) % 3 == 2 then
      result = result .. ","
    end
    result = result .. sub(s, i, i)
  end
  return result
end
```

---

## Angle Wrap

```lua
-- keep angle in 0..1 range
function wrap_angle(a)
  return a % 1
end

-- interpolate angles (taking short path)
function angle_lerp(a, b, t)
  local diff = (b - a + 0.5) % 1 - 0.5
  return a + diff * t
end
```

---

## Map Value to Range

```lua
-- remap value from one range to another
function remap(v, in_lo, in_hi, out_lo, out_hi)
  return out_lo + (out_hi - out_lo) * ((v - in_lo) / (in_hi - in_lo))
end

-- example: hp 0..10 → bar width 0..40 pixels
local bar_w = remap(hp, 0, max_hp, 0, 40)
rectfill(10, 4, 10 + bar_w, 8, 11)
```
