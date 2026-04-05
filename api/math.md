# Math API

## Number Range

PICO-8 uses **16:16 fixed-point** numbers.
- Range: **-32768** to **32767.9999**
- Precision: ~0.00002 (1/65536)
- No true floats — division results are truncated to fixed-point

---

## Core Functions

```lua
abs(x)        -- absolute value: abs(-3) = 3
flr(x)        -- floor (round toward -inf): flr(4.9) = 4, flr(-4.1) = -5
ceil(x)       -- ceiling (round toward +inf): ceil(4.1) = 5
max(x, y)     -- maximum of two values
min(x, y)     -- minimum of two values
mid(x, y, z)  -- middle value of three: mid(1,5,3) = 3
sgn(x)        -- sign: returns 1 or -1 (sgn(0) = 1)
sqrt(x)       -- square root
```

---

## Random

```lua
rnd(x)        -- random number: 0 <= n < x
rnd(t)        -- random element from table t
srand(x)      -- seed the RNG (same seed = same sequence)
```

```lua
-- random integer 1..6 (like a die)
local roll = flr(rnd(6)) + 1

-- random color
local col = flr(rnd(16))

-- random element
local item = rnd({"sword", "shield", "bow"})
```

---

## Trig

PICO-8 uses **0..1** range instead of radians/degrees.
- 0 = 0°, 0.25 = 90°, 0.5 = 180°, 0.75 = 270°, 1.0 = 360°

```lua
sin(x)        -- sine of x (0..1); Y-INVERTED for screen-space
cos(x)        -- cosine of x (0..1)
atan2(dx, dy) -- angle from (0,0) toward (dx,dy), returns 0..1
```

**Important:** `sin()` is inverted — `sin(0.25)` returns **-1** (not +1).
This matches screen-space where positive Y is down.

### Moving at an angle

```lua
-- move in direction `a` (0..1) at speed `spd`
dx = cos(a) * spd
dy = -sin(a) * spd   -- negate sin for screen coords

-- OR use sin directly (moves downward when a=0.25)
dx = cos(a) * spd
dy = sin(a) * spd    -- use this if you want normal screen-space movement
```

### Angle between two points

```lua
local a = atan2(tx - sx, ty - sy)  -- angle from (sx,sy) toward (tx,ty)
```

### Circle motion

```lua
-- object orbiting a center point
local t = time() * 0.2  -- speed of orbit
local x = cx + cos(t) * radius
local y = cy + sin(t) * radius
```

---

## Operators

```lua
-- arithmetic
+  -  *  /  %  ^

-- integer division
7 \ 2        -- = 3 (same as flr(7/2))

-- ceil workaround (no ceil function in old versions)
-flr(-x)     -- = ceil(x)

-- shorthand
x += 1
x -= 1
x *= 2
x /= 2
x %= 8
```

---

## Bitwise

```lua
-- operators
x & y        -- AND
x | y        -- OR
x ^^ y       -- XOR
~x           -- NOT
x << n       -- shift left
x >> n       -- arithmetic right shift
x >>> n      -- logical right shift

-- functions (equivalent)
band(x, y)
bor(x, y)
bxor(x, y)
bnot(x)
shl(x, n)
shr(x, n)
lshr(x, n)
rotl(x, n)   -- rotate bits left
rotr(x, n)   -- rotate bits right
```

---

## Useful Math Patterns

### Clamp

```lua
-- clamp x between lo and hi
x = mid(lo, x, hi)
```

### Lerp (linear interpolation)

```lua
-- move v toward target by t (0..1)
v = v + (target - v) * t

-- or explicit:
function lerp(a, b, t)
  return a + (b - a) * t
end
```

### Distance

```lua
function dist(x1, y1, x2, y2)
  return sqrt((x2-x1)^2 + (y2-y1)^2)
end

-- faster (no sqrt) for comparisons only:
function dist_sq(x1, y1, x2, y2)
  return (x2-x1)^2 + (y2-y1)^2
end
if dist_sq(px, py, ex, ey) < 64 then  -- 64 = 8^2
  -- within 8 pixels
end
```

### Screen Wrap

```lua
x = (x + 128) % 128    -- wrap x to 0..127
y = (y + 128) % 128    -- wrap y to 0..127
```

### Approach (move toward without overshoot)

```lua
function approach(v, target, step)
  if v < target then
    return min(v + step, target)
  else
    return max(v - step, target)
  end
end

-- use:
spd = approach(spd, max_spd, accel)
```

### Random Between Two Values

```lua
function rndb(lo, hi)
  return flr(rnd(hi - lo + 1) + lo)
end

rndb(1, 6)    -- 1 to 6 inclusive
rndb(0, 127)  -- random screen x
```
