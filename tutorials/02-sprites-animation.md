# Tutorial 02 — Sprites & Animation

**What you'll learn:** Drawing sprites, animating them with multiple frames, flipping for direction, transparency, and multi-tile sprites.

**Prerequisites:** [01-hello-pico8.md](01-hello-pico8.md)

---

## Drawing a Sprite

You already know how to make sprites in the sprite editor. Now let's draw one in code.

```lua
spr(n, x, y)
```

- `n` = sprite number (shown in the editor, bottom left)
- `x, y` = screen position of the **top-left corner** of the sprite

```lua
function _init()
  px = 60
  py = 60
end

function _draw()
  cls()
  spr(1, px, py)   -- draw sprite #1 at px,py
end
```

Sprite #1 is whatever you drew in slot 1 of the sprite editor.

> **Make sure you have something in sprite #1 before running!** Draw a simple character — a face, a box with eyes, anything. Sprite #0 is the eraser — leave it empty.

---

## How Sprites Sit on Screen

Each sprite is **8×8 pixels**. The `x,y` you give `spr()` is the **top-left corner**.

```
x,y ──────────────┐
│  8px wide       │
│  8px tall       │
└─────────────────┘ x+8, y+8
```

So to center an 8×8 sprite on the 128×128 screen:
```lua
spr(1, 60, 60)   -- 128/2 - 8/2 = 60
```

---

## Step 1 — Moving a Sprite

```lua
function _init()
  px = 60
  py = 60
end

function _update()
  px += 1          -- move right each frame
  if px > 127 then px = -8 end  -- wrap when off right edge
end

function _draw()
  cls()
  spr(1, px, py)
end
```

---

## Step 2 — Flipping (Direction)

`spr()` has optional flip arguments:

```lua
spr(n, x, y, w, h, flip_x, flip_y)
```

`flip_x = true` mirrors the sprite horizontally. Use this so you only need to draw your character facing one direction — flip it for the other.

```lua
function _init()
  px = 60
  py = 60
  facing_right = true
end

function _update()
  if btn(0) then   -- left
    px -= 2
    facing_right = false
  end
  if btn(1) then   -- right
    px += 2
    facing_right = true
  end
end

function _draw()
  cls()
  spr(1, px, py, 1, 1, not facing_right)
  -- flip_x is true when facing LEFT (not facing_right)
end
```

The `1, 1` before the flip args are width and height in sprite-tiles (default is 1×1 = 8×8px). We'll cover larger sprites below.

---

## Step 3 — Animation

Animation in PICO-8 is simple: **draw different sprites on different frames.**

Make two (or more) sprites for your character — e.g.:
- Sprite #1: walking frame A (left leg forward)
- Sprite #2: walking frame B (right leg forward)

Then in code, switch between them based on a timer:

```lua
function _init()
  px = 60
  py = 60
  anim_frame = 0    -- which animation frame we're on
  anim_timer = 0    -- counts up to switch frames
  anim_spd   = 8    -- frames per sprite (lower = faster animation)
end

function _update()
  local moving = false

  if btn(0) then px -= 2  moving = true end
  if btn(1) then px += 2  moving = true end

  if moving then
    anim_timer += 1
    if anim_timer >= anim_spd then
      anim_timer = 0
      anim_frame = (anim_frame + 1) % 2   -- toggle between 0 and 1
    end
  else
    anim_frame = 0   -- reset to idle frame when stopped
    anim_timer = 0
  end
end

function _draw()
  cls()
  local s = 1 + anim_frame   -- sprite #1 or #2
  spr(s, px, py)
end
```

### How the animation cycle works

`anim_timer` counts up each frame. When it hits `anim_spd`, it resets to 0 and advances `anim_frame` by 1. `% 2` wraps it back to 0 after 1, so it cycles: 0 → 1 → 0 → 1 → ...

---

## Step 4 — A Cleaner Animation System

For more frames and states, use a table to define animations:

```lua
-- sprite layout in editor:
-- row 0: idle (sprites 1)
-- row 1: walk (sprites 2, 3, 4)
-- row 2: jump (sprite 5)

function _init()
  p = {}
  p.x     = 60
  p.y     = 60
  p.state = "idle"
  p.frame = 0
  p.timer = 0

  -- animation definitions: {first sprite, frame count, speed}
  anims = {
    idle = {1, 1, 8},    -- 1 frame starting at sprite 1
    walk = {2, 3, 6},    -- 3 frames starting at sprite 2
    jump = {5, 1, 8},    -- 1 frame starting at sprite 5
  }
end

function animate(entity)
  local a = anims[entity.state]
  entity.timer += 1
  if entity.timer >= a[3] then
    entity.timer = 0
    entity.frame = (entity.frame + 1) % a[2]
  end
  return a[1] + entity.frame   -- returns current sprite number
end

function _update()
  -- movement
  if btn(0) or btn(1) then
    p.state = "walk"
  else
    p.state = "idle"
  end
  if btn(0) then p.x -= 2 end
  if btn(1) then p.x += 2 end
end

function _draw()
  cls()
  local s = animate(p)
  spr(s, p.x, p.y)
end
```

---

## Step 5 — Larger Sprites (Multi-Tile)

The 4th and 5th arguments to `spr()` are width and height **in tiles** (each tile = 8px):

```lua
spr(n, x, y, w, h)

spr(1, px, py, 2, 2)   -- draw a 16x16 sprite (2x2 tile block)
spr(1, px, py, 1, 2)   -- draw an 8x16 sprite (1x2 tile block — tall character)
spr(1, px, py, 2, 1)   -- draw a 16x8 sprite (2x1 tile block — wide)
```

In the sprite editor, your sprites must be **adjacent**. For a 2×2 sprite:
- Sprite #1 = top-left
- Sprite #2 = top-right
- Sprite #17 = bottom-left (next row down)
- Sprite #18 = bottom-right

`spr(1, x, y, 2, 2)` automatically draws all four.

---

## Step 6 — Transparency

By default, **color 0 (black) is transparent** when drawing sprites. Anything black in your sprite won't be drawn — the background shows through.

```lua
palt(0, true)    -- default: black is transparent (already set)
palt(0, false)   -- make black opaque (useful for black outlines)
palt(14, true)   -- make pink transparent instead
palt()           -- reset all transparency to defaults
```

**Example:** if you want your character to have a solid black outline, but the background to be dark blue:

```lua
-- color 0 = black (normally transparent)
-- if your outline color is black, set it opaque:
palt(0, false)
spr(1, px, py)
palt(0, true)   -- restore after drawing
```

---

## Step 7 — Palette Swap (Color Effects)

Swap colors when drawing sprites — useful for damage flashing, team colors, etc.

```lua
-- flash player white when hit
if hit_timer > 0 then
  pal(8, 7)    -- swap red -> white (or whatever your sprite colors are)
  pal(11, 7)   -- swap green -> white
end
spr(1, px, py)
pal()          -- ALWAYS reset palette after drawing
```

---

## Full Example — Animated Character

```lua
function _init()
  px = 60
  py = 60
  facing = 1      -- 1=right, -1=left
  walking = false
  frame = 0
  ftimer = 0
end

function _update()
  walking = false
  if btn(0) then px -= 2  facing = -1  walking = true end
  if btn(1) then px += 2  facing =  1  walking = true end

  -- animate walk
  if walking then
    ftimer += 1
    if ftimer >= 8 then
      ftimer = 0
      frame = (frame + 1) % 2   -- sprite 1 and 2
    end
  else
    frame = 0  -- idle: always sprite 1
    ftimer = 0
  end

  -- clamp to screen
  px = mid(0, px, 120)
end

function _draw()
  cls(1)   -- dark blue background
  local s = 1 + frame
  local flip = (facing < 0)
  spr(s, px, py, 1, 1, flip)
end
```

---

## Challenge

1. Add an "up" and "down" animation state for a top-down character (4 directions = 4 sprite rows)
2. Make an idle "breathing" animation that only plays when standing still (alternate between 2 idle frames slowly)
3. Make an enemy patrol left and right, flipping direction automatically
4. Try a `sspr()` call to stretch a sprite to double size

---

## What You Learned

- `spr(n, x, y)` draws sprite n at position x,y
- `flip_x` flips horizontally — draw once, face both ways
- Animation = cycling through sprite numbers on a timer
- `w, h` args let you draw multi-tile sprites
- Color 0 (black) is transparent by default — use `palt()` to change this
- `pal()` swaps colors for effects, reset with `pal()` after

**Next:** [03-input-movement.md](03-input-movement.md) — proper input handling and movement systems.
