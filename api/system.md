# System API

## Time

```lua
time()    -- seconds elapsed since cart started (also: t())
```

```lua
-- simple timer
if time() - last_shot > 0.5 then
  shoot()
  last_shot = time()
end

-- animate using time
local frame = flr(time() * 8) % 4   -- cycle 4 frames at 8fps
spr(base_spr + frame, px, py)
```

---

## stat() — System Status

```lua
stat(0)    -- Lua memory used (KB), max ~2048
stat(1)    -- CPU used last frame (0..1); >1 = dropping frames
stat(2)    -- System CPU (API calls only)
stat(7)    -- Current framerate

-- Audio
stat(16)   -- SFX on channel 0 (-1=none)
stat(17)   -- SFX on channel 1
stat(18)   -- SFX on channel 2
stat(19)   -- SFX on channel 3
stat(54)   -- Current music pattern
stat(57)   -- Music playing (bool)

-- Input (devkit mode only)
stat(30)   -- Keypress available
stat(31)   -- Key character
stat(32)   -- Mouse X
stat(33)   -- Mouse Y
stat(34)   -- Mouse buttons bitmask
stat(36)   -- Mouse wheel (1=up, -1=down)

-- Date/Time
stat(80)   -- UTC year
stat(81)   -- UTC month (1-12)
stat(82)   -- UTC day
stat(83)   -- UTC hour
stat(84)   -- UTC minute
stat(85)   -- UTC second
stat(90)   -- Local year
stat(91)   -- Local month
stat(92)   -- Local day
stat(93)   -- Local hour
stat(94)   -- Local minute
stat(95)   -- Local second

-- Misc
stat(4)    -- Clipboard contents
stat(6)    -- Parameter string (from load())
stat(100)  -- Current breadcrumb label
stat(110)  -- Frame-by-frame mode (bool)
```

---

## Persistent Save Data (cartdata)

64 slots, each holding a fixed-point number. Survives cart restarts.

```lua
-- call once at startup with a unique id string
cartdata("mygame_v1")

-- read a value
local hi = dget(0)

-- write a value
dset(0, new_hi_score)
```

**Important:** call `cartdata()` only once per execution, in `_init()`.

```lua
function _init()
  cartdata("mygame_saves_v1")
  hi_score = dget(0)
  level_unlocked = dget(1)
end

function save_game()
  dset(0, hi_score)
  dset(1, level_unlocked)
end
```

---

## Flow Control

```lua
run()           -- run cart from start
run(param_str)  -- run with parameter string (read with stat(6))
stop()          -- stop execution
stop("msg")     -- stop with message in console
reboot          -- restart PICO-8 (command, not function)
reset()         -- reset draw state only
```

---

## Loading Carts

```lua
load("other_cart")           -- load and run another cart
load("other_cart", "label")  -- with breadcrumb (back button text)
load("other_cart", "label", "params")  -- with params
```

---

## Debug Tools

```lua
assert(condition)           -- stop if condition is false
assert(condition, "msg")    -- stop with message
printh(str)                 -- print to host terminal (not screen)
printh(str, "log.txt")      -- append to file
printh(str, "log.txt", true) -- overwrite file
```

**Live variable inspection while running:**
```
?player.x     -- shorthand print to console (press ctrl+r, then esc)
```

**Frame stepping:** Press `. ` (period) in console to advance one frame.

---

## extcmd()

```lua
extcmd("screen")        -- take screenshot
extcmd("label")         -- set cart label to current screen
extcmd("rec")           -- start GIF recording
extcmd("video")         -- save GIF
extcmd("pause")         -- open pause menu
extcmd("reset")         -- request cart reset
extcmd("set_title", "My Game")  -- set window title
```

---

## Custom Pause Menu

```lua
-- add items to the ESC pause menu (1–5)
menuitem(1, "restart", function()
  _init()
end)

menuitem(2, "quit", function()
  run()  -- or load another cart
end)

-- remove an item
menuitem(1)
```

---

## Coroutines (Advanced)

Useful for scripted sequences, cutscenes, typewriter text.

```lua
function _init()
  co = cocreate(my_sequence)
end

function _update()
  if co and costatus(co) ~= "dead" then
    coresume(co)
  end
end

function my_sequence()
  print("step 1")
  for i = 1, 30 do yield() end  -- wait 1 second (30 frames)
  print("step 2")
  for i = 1, 60 do yield() end  -- wait 2 seconds
  print("done!")
end
```

**costatus values:** `"running"`, `"suspended"`, `"dead"`

---

## Game States Pattern

```lua
function _init()
  menu_init()
end

-- menu state
function menu_init()
  _update = menu_update
  _draw   = menu_draw
end

function menu_update()
  if btnp(4) then game_init() end
end

function menu_draw()
  cls()
  print("press o to start", 28, 60, 7)
end

-- gameplay state
function game_init()
  _update = game_update
  _draw   = game_draw
  score = 0
end

function game_update()
  -- game logic
end

function game_draw()
  cls()
  -- game drawing
end

-- game over state
function gameover_init()
  _update = gameover_update
  _draw   = gameover_draw
end

function gameover_update()
  if btnp(4) then menu_init() end
end

function gameover_draw()
  cls()
  print("game over", 44, 56, 8)
  print("press o", 48, 68, 7)
end
```

---

## Memory

```lua
peek(addr)         -- read byte from RAM
poke(addr, val)    -- write byte to RAM
peek2(addr)        -- read 16-bit value
poke2(addr, val)
peek4(addr)        -- read 32-bit value
poke4(addr, val)

memcpy(dest, src, len)       -- copy bytes in RAM
memset(dest, val, len)       -- fill bytes in RAM

reload(dest, src, len)       -- copy from cart ROM to RAM
cstore(dest, src, len)       -- copy from RAM to cart ROM
```

### Key RAM Addresses

```
0x0000  sprite sheet (gfx)
0x2000  map data
0x3000  sprite flags
0x3200  SFX data
0x5e00  persistent cart data (256 bytes)
0x6000  screen buffer (8k)
```
