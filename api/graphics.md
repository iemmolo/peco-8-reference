# Graphics API

## Screen

```lua
cls([col])                  -- clear screen (default col 0 = black)
color(col)                  -- set default draw color
camera([x, y])              -- offset all drawing by -x,-y (for scrolling)
camera()                    -- reset camera to 0,0
clip(x, y, w, h)            -- set clipping rect (only draw inside)
clip()                      -- reset clipping
reset()                     -- reset all draw state
flip()                      -- manually flip backbuffer (use with _update60)
```

**Screen space:** 0,0 = top-left, 127,127 = bottom-right. Positive Y is **down**.

---

## Primitives

```lua
pset(x, y, [col])              -- set pixel
pget(x, y)                     -- get pixel color at x,y

line(x0, y0, x1, y1, [col])   -- draw line

rect(x0, y0, x1, y1, [col])        -- rectangle outline
rectfill(x0, y0, x1, y1, [col])    -- filled rectangle

circ(x, y, r, [col])               -- circle outline
circfill(x, y, r, [col])           -- filled circle

oval(x0, y0, x1, y1, [col])        -- oval/ellipse outline (bounding box)
ovalfill(x0, y0, x1, y1, [col])    -- filled oval
```

---

## Text

```lua
print(str, [x, y, [col]])    -- print string at x,y
cursor(x, y)                  -- set text cursor position
?"hello"                      -- shorthand for print (single line only)
```

**Text wraps** at screen edge. Each character is 4×6 pixels (+ 1px gap).

**Useful print trick:** hide trailing newline with `\0`
```lua
print("no newline\0")
```

---

## Sprites

```lua
spr(n, x, y, [w, h], [flip_x], [flip_y])
-- n        = sprite number (0-255)
-- x,y      = top-left pixel position
-- w,h      = width/height in sprites (default 1,1)
-- flip_x   = flip horizontally (true/false)
-- flip_y   = flip vertically (true/false)

spr(1, px, py)              -- draw sprite 1 at px,py
spr(1, px, py, 2, 2)        -- draw 2x2 block of sprites (16x16 px)
spr(1, px, py, 1, 1, true)  -- flip horizontal
```

### Sprite sheet direct

```lua
sspr(sx, sy, sw, sh, dx, dy, [dw, dh], [flip_x], [flip_y])
-- sx,sy  = source pixel in sprite sheet
-- sw,sh  = source width/height in pixels
-- dx,dy  = destination on screen
-- dw,dh  = destination size (stretches if different from sw,sh)

sspr(0, 0, 16, 16, 10, 10, 32, 32)  -- draw 16x16 stretched to 32x32
```

---

## Palette & Transparency

```lua
pal()                    -- reset palette to default
pal(c0, c1)              -- swap color c0 → c1 in draw palette
pal(c0, c1, 1)           -- swap on display palette (affects whole screen)

palt()                   -- reset transparency (color 0 = transparent)
palt(col, t)             -- set color col transparent (t=true) or solid (t=false)
palt(0, false)           -- make black solid (useful for outlines)
palt(14, true)           -- make pink transparent
```

**Default:** color 0 (black) is transparent when drawing sprites.

### Palette tricks

```lua
-- enemy flash red when hit
pal(7, 8)    -- swap white -> red
spr(enemy_spr, ex, ey)
pal()        -- reset

-- night mode — shift all colors darker
pal(7, 5)    -- white -> dark gray
pal(11, 3)   -- green -> dark green
-- etc.
```

---

## Fill Pattern

```lua
fillp(mask)    -- set 4x4 dithering pattern for shapes
fillp()        -- clear fill pattern
```

Pattern mask values (each bit = one pixel in 4×4 grid):
```
32768 16384  8192  4096
 2048  1024   512   256
  128    64    32    16
    8     4     2     1
```

```lua
fillp(0b0101010101010101)  -- 50% checkerboard
fillp(0b1111000011110000)  -- horizontal stripes
```

Works with: `circ`, `circfill`, `rect`, `rectfill`, `pset`, `line`

---

## Camera (Scrolling)

```lua
camera(cam_x, cam_y)    -- offset: world position that maps to screen 0,0
camera()                -- reset

-- Example: follow player
cam_x = px - 64
cam_y = py - 64
cam_x = mid(0, cam_x, world_w - 128)  -- clamp to world bounds
cam_y = mid(0, cam_y, world_h - 128)
camera(cam_x, cam_y)
map(0, 0, 0, 0, 16, 64)  -- draw map (camera handles offset)
spr(1, px, py)            -- player position is in world coords
```

---

## Sprite Flags

```lua
fget(n)        -- get all 8 flags as number
fget(n, f)     -- get specific flag f (0–7) as true/false
fset(n, f, v)  -- set flag f on sprite n to v (true/false)
fset(n, v)     -- set all flags at once (as bitfield number)
```

```lua
-- common: flag 0 = solid
if fget(mget(tx, ty), 0) then
  -- solid tile
end
```

---

## Color Reference

| # | Color | | # | Color |
|---|-------|-|---|-------|
| 0 | Black | | 8 | Red |
| 1 | Dark Blue | | 9 | Orange |
| 2 | Dark Purple | | 10 | Yellow |
| 3 | Dark Green | | 11 | Green |
| 4 | Brown | | 12 | Blue |
| 5 | Dark Gray | | 13 | Indigo |
| 6 | Light Gray | | 14 | Pink |
| 7 | White | | 15 | Peach |
