# Input API

## Button Functions

```lua
btn(b, [p])    -- true if button b is held down
btnp(b, [p])   -- true only on the frame button b was first pressed
               -- also repeats after 15 frames held, every 4 frames
```

- `b` = button number (0–5)
- `p` = player number (0 or 1, default 0)

---

## Button Numbers

```
btn(0)  LEFT    btn(4)  O  (confirm / jump)
btn(1)  RIGHT   btn(5)  X  (cancel / action)
btn(2)  UP
btn(3)  DOWN
```

---

## Controller Mapping

### SNES USB Controller

```
D-pad        → btn(0/1/2/3)
B            → btn(4)  [O button]
Y            → btn(5)  [X button]
A/X/L/R      → not mapped by default
```

### N64 USB Controller

```
D-pad        → btn(0/1/2/3)
A            → btn(4)  [O button]
B            → btn(5)  [X button]
Other buttons → not mapped by default
```

Use `keyconfig` in the PICO-8 console to remap if needed.

---

## Two Players

```lua
-- player 1
if btn(0, 0) then p1_x -= 1 end

-- player 2
if btn(0, 1) then p2_x -= 1 end

-- check all buttons as bitfield (no args)
local bits = btn()
-- bits 0-5 = player 1, bits 8-13 = player 2
```

---

## btn vs btnp

| | `btn` | `btnp` |
|--|-------|--------|
| Returns true | Every frame held | Only first frame pressed |
| Good for | Movement, held actions | Jump, shoot, menu select |
| Auto-repeat | No | Yes (15f delay, 4f interval) |

```lua
-- movement: use btn (held)
if btn(0) then px -= 2 end

-- jump: use btnp (once per press)
if btnp(4) then
  vy = -4
end
```

---

## Custom Auto-Repeat Timing

```lua
poke(0x5f5c, delay)    -- frames before repeat starts (255 = never)
poke(0x5f5d, interval) -- frames between repeats
-- defaults: 15 delay, 4 interval
```

---

## Common Patterns

### 4-Direction Movement

```lua
local dx, dy = 0, 0
if btn(0) then dx = -1 end
if btn(1) then dx =  1 end
if btn(2) then dy = -1 end
if btn(3) then dy =  1 end
px += dx * spd
py += dy * spd
```

### 8-Direction (Normalize Diagonal)

```lua
local dx, dy = 0, 0
if btn(0) then dx -= 1 end
if btn(1) then dx += 1 end
if btn(2) then dy -= 1 end
if btn(3) then dy += 1 end

-- normalize diagonal speed
if dx ~= 0 and dy ~= 0 then
  dx *= 0.707
  dy *= 0.707
end

px += dx * spd
py += dy * spd
```

### Any Button Pressed

```lua
if btn() > 0 then
  -- any button is held
end

if btnp() > 0 then
  -- any button just pressed
end
```

### Menu / Pause Detection

```lua
if btnp(4) then
  -- confirm / select
  sfx(0)
end

if btnp(5) then
  -- back / cancel
end
```

---

## Keyboard & Mouse (Dev Mode)

Enable extended input (useful in dev/desktop, not typical for Pi):

```lua
poke(0x5f2d, 0x1)   -- enable keyboard/mouse

stat(30)    -- key pressed this frame (bool)
stat(31)    -- key character string
stat(32)    -- mouse X
stat(33)    -- mouse Y
stat(34)    -- mouse buttons (bitfield: 1=L, 2=R, 4=M)
stat(36)    -- mouse wheel (1=up, -1=down)
```
