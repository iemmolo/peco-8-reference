# Tutorial 01 — Hello PICO-8

**What you'll learn:** The game loop, drawing shapes and text, colors, variables.

**Prerequisites:** None. Start here.

---

## The Three Magic Functions

Every PICO-8 game is built around three functions. You don't call them yourself — PICO-8 calls them for you automatically, in a loop, forever.

```lua
function _init()
  -- runs ONCE when the cart starts
end

function _update()
  -- runs 30 times per second (30fps)
  -- put your game logic here
end

function _draw()
  -- runs once per visible frame
  -- put all your drawing here
end
```

Think of it like this:
- `_init()` = set everything up
- `_update()` = think (move things, check buttons, do maths)
- `_draw()` = draw (paint the current state to screen)

**Important rule:** Never draw inside `_update()`. Never do logic inside `_draw()`. Keep them separate.

---

## Step 1 — Your First Cart

Open PICO-8, press ESC to get to the code editor, and type this:

```lua
function _init()
  msg = "hello, world!"
end

function _update()
end

function _draw()
  cls()
  print(msg, 40, 60, 7)
end
```

Press `CTRL+R` to run it.

You should see "hello, world!" printed in white near the center of the screen.

### What's happening?

- `cls()` clears the screen to black every frame. Without it, everything you draw piles up on top of itself.
- `print(msg, 40, 60, 7)` prints the variable `msg` at position x=40, y=60 in color 7 (white).
- Screen coordinates: **0,0 is top-left**, **127,127 is bottom-right**. Positive Y goes **down**.

---

## Step 2 — Drawing Shapes

Replace your `_draw()` with this and run it:

```lua
function _draw()
  cls()

  -- rectangle outline
  rect(10, 10, 50, 40, 7)

  -- filled rectangle
  rectfill(60, 10, 100, 40, 8)

  -- circle outline
  circ(30, 80, 15, 11)

  -- filled circle
  circfill(90, 80, 15, 12)

  -- a line
  line(0, 64, 127, 64, 5)
end
```

### Shape functions

```lua
rect(x0, y0, x1, y1, col)       -- outline rectangle
rectfill(x0, y0, x1, y1, col)   -- filled rectangle
circ(x, y, radius, col)         -- circle outline
circfill(x, y, radius, col)     -- filled circle
line(x0, y0, x1, y1, col)       -- line between two points
pset(x, y, col)                  -- single pixel
```

For `rect` and `circ`: `x0,y0` is the top-left corner, `x1,y1` is the bottom-right.
For `circ`: `x,y` is the **centre**.

---

## Step 3 — Colors

PICO-8 has exactly 16 colors, numbered 0–15:

```
 0  black        8  red
 1  dark blue    9  orange
 2  dark purple  10 yellow
 3  dark green   11 green
 4  brown        12 blue
 5  dark gray    13 indigo
 6  light gray   14 pink
 7  white        15 peach
```

Try this to see them all:

```lua
function _draw()
  cls()
  for i = 0, 15 do
    rectfill(i*8, 0, i*8+7, 127, i)
    print(i, i*8+1, 60, (i < 6 or i == 8) and 7 or 0)
  end
end
```

---

## Step 4 — Variables & Simple Animation

Variables store values. You can change them every frame to make things move.

```lua
function _init()
  x = 0     -- starting x position
  spd = 1   -- speed
end

function _update()
  x += spd          -- move right each frame
  if x > 127 then   -- if we hit the right edge
    x = 0           -- wrap back to left
  end
end

function _draw()
  cls()
  circfill(x, 64, 4, 10)    -- draw circle at x, y=64
end
```

Run it. A yellow dot moves across the screen and wraps around.

### What's `+=`?

`x += spd` is shorthand for `x = x + spd`. PICO-8 supports `+=`, `-=`, `*=`, `/=`.

---

## Step 5 — Multiple Moving Things

```lua
function _init()
  x1 = 0
  x2 = 64
  y1 = 40
  y2 = 90
end

function _update()
  x1 += 1.5
  x2 -= 0.8
  if x1 > 127 then x1 = 0 end
  if x2 < 0   then x2 = 127 end
end

function _draw()
  cls()
  circfill(x1, y1, 6, 8)    -- red ball moving right
  circfill(x2, y2, 6, 12)   -- blue ball moving left
  line(0, 64, 127, 64, 5)   -- center line
end
```

---

## Step 6 — Printing Values (Useful for Debugging)

```lua
function _draw()
  cls()
  circfill(x1, y1, 6, 8)

  -- print variable values on screen
  print("x1=" .. x1, 2, 2, 7)
  print("x2=" .. x2, 2, 10, 7)
end
```

`..` joins strings together. `tostr(x1)` converts a number to a string (print does this automatically).

**Quick debug trick:** While a game is running, press ESC to pause it, then type `?x1` in the console to see the current value of any variable.

---

## Step 7 — Using `time()`

`time()` returns seconds since the cart started. Useful for animations:

```lua
function _draw()
  cls()

  -- circle that pulses in size using sine wave
  local r = 10 + sin(time() * 0.5) * 8
  circfill(64, 64, r, 11)

  -- box that moves in a circle
  local bx = 64 + cos(time() * 0.3) * 40
  local by = 64 + sin(time() * 0.3) * 40
  rectfill(bx-4, by-4, bx+4, by+4, 8)
end
```

Note: `sin()` in PICO-8 takes values in 0..1 range (not radians), and is **y-inverted**. See `api/math.md` for details.

---

## Challenge

Try these on your own:

1. Make the circle move in a figure-8 pattern
2. Draw a "face" using circles and lines
3. Make a shape change color every second using `flr(time()) % 16`
4. Print your name on screen and make it bounce between left and right edges

---

## What You Learned

- The game loop: `_init` → `_update` → `_draw` (forever)
- `cls()` clears the screen each frame
- Basic drawing: `rect`, `rectfill`, `circ`, `circfill`, `line`, `pset`, `print`
- 16 colors numbered 0–15
- Variables change between frames to create movement
- `time()` gives you seconds for animation

**Next:** [02-sprites-animation.md](02-sprites-animation.md) — drawing sprites and making them animate.
