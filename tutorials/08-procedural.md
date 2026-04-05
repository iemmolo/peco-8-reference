# Tutorial 08 — Procedural Generation

**What you'll learn:** How to generate random terrain, caves, dungeons, and backgrounds. Using `rnd()` and `srand()` to create worlds that are different every run.

**Prerequisites:** [07-entities.md](07-entities.md)

---

## What Is Procedural Generation?

Instead of hand-designing a level, you write code that **generates** one using randomness and rules. Every run is different (or you can seed it to get the same world each time).

The key idea: **controlled randomness**. You don't just randomly place tiles everywhere — you use rules to make it feel natural.

---

## srand() — Reproducible Randomness

`rnd()` gives you a random number, but the sequence is determined by a **seed**.
`srand(n)` sets that seed. Same seed = same random sequence every time.

```lua
srand(42)
print(rnd(100))   -- always the same number
print(rnd(100))   -- always the same second number

srand(42)         -- reset seed
print(rnd(100))   -- same as the first number again
```

**Use case:** generate the same world layout each run for a given level number:
```lua
srand(level_number * 1337)
generate_level()
```

Or use `time()` for a different world every run:
```lua
srand(time())
generate_level()
```

---

## Step 1 — Random Platforms

The simplest procedural level: place platforms at random positions.

```lua
function _init()
  platforms = {}
  srand(time())   -- random each run

  -- place 8 random platforms
  for i = 1, 8 do
    add(platforms, {
      x = flr(rnd(14)) * 8,   -- snap to 8px grid
      y = 24 + i * 12,         -- each below the last
      w = flr(rnd(5)) + 3      -- width 3-7 tiles
    })
  end
end

function _draw()
  cls(1)
  for p in all(platforms) do
    rectfill(p.x, p.y, p.x + p.w*8, p.y + 4, 6)
  end
  spr(1, px, py)
end
```

The problem: sometimes platforms are too far apart to reach. To fix this, we need **rules** on top of randomness.

---

## Step 2 — Walkable Platform Layout

Generate platforms that are always reachable by controlling how far apart they can be:

```lua
function make_platforms()
  platforms = {}
  local x = 8
  local y = 100

  for i = 1, 12 do
    add(platforms, {x=x, y=y, w=flr(rnd(4))+3})

    -- next platform: move right and vary height
    x  = x + (flr(rnd(4)) + 3) * 8 + flr(rnd(3)+1)*8
    y  = mid(40, y + flr(rnd(5)*2 - 4)*8, 108)
    -- mid() ensures y stays on screen
  end
end
```

---

## Step 3 — Random Terrain (Heightmap)

A heightmap is an array of ground heights, one per column. Great for side-scrollers and cave games.

```lua
function make_terrain()
  ground = {}    -- ground[i] = y pixel where ground starts at column i
  local h = 80   -- starting height

  for i = 1, 128 do
    -- vary height by a small random amount each column
    h = mid(60, h + flr(rnd(5)) - 2, 110)
    ground[i] = h
  end
end

function draw_terrain()
  for i = 1, 128 do
    -- draw a vertical line from ground[i] down to bottom of screen
    line(i-1, ground[i], i-1, 127, 3)   -- dark green
  end
end
```

**Smoother terrain:** average the current step with the previous one:

```lua
for i = 1, 128 do
  local step = flr(rnd(7)) - 3    -- -3 to +3
  h = (h + h + h + step) / 4     -- blend: mostly previous, slight change
  h = mid(50, h, 110)
  ground[i] = h
end
```

---

## Step 4 — Scrolling Cave (From the Side-Scroller Tutorial)

This is the cave system from the Cave Diver game — a scrolling cave that generates as you go:

```lua
function make_cave()
  cave = {{top=5, btm=119}}    -- start with one column
  cave_top = 45   -- how far down ceiling can go
  cave_btm = 85   -- how far up floor can come
end

function update_cave(scroll_spd)
  -- remove columns that scrolled off screen
  for i = 1, scroll_spd do
    del(cave, cave[1])
  end

  -- add new columns at the right edge
  while #cave < 128 do
    local last = cave[#cave]
    local col  = {}
    col.top = mid(3,         last.top + flr(rnd(7)-3), cave_top)
    col.btm = mid(cave_btm,  last.btm + flr(rnd(7)-3), 124)
    add(cave, col)
  end
end

function draw_cave()
  for i = 1, #cave do
    -- ceiling
    line(i-1, 0,   i-1, cave[i].top, 5)
    -- floor
    line(i-1, 127, i-1, cave[i].btm, 5)
  end
end
```

The key insight: we only store 128 columns (one per screen pixel). New columns are generated from the last one + a small random variation. This keeps the cave smooth and natural.

---

## Step 5 — Random Dungeon Rooms

Divide the map into rooms and connect them with corridors:

```lua
function make_dungeon()
  -- clear map to walls
  for x = 0, 127 do
    for y = 0, 63 do
      mset(x, y, 2)   -- sprite 2 = wall tile
    end
  end

  rooms = {}

  -- carve 6 random rooms
  for i = 1, 6 do
    local rw = flr(rnd(5)) + 4    -- room width 4-8 tiles
    local rh = flr(rnd(4)) + 3    -- room height 3-6 tiles
    local rx = flr(rnd(128-rw-2)) + 1
    local ry = flr(rnd(64-rh-2))  + 1

    add(rooms, {x=rx, y=ry, w=rw, h=rh})
    carve_room(rx, ry, rw, rh)
  end

  -- connect rooms with corridors
  for i = 1, #rooms - 1 do
    carve_corridor(rooms[i], rooms[i+1])
  end
end

function carve_room(rx, ry, rw, rh)
  for x = rx, rx+rw-1 do
    for y = ry, ry+rh-1 do
      mset(x, y, 1)   -- sprite 1 = floor tile
    end
  end
end

function carve_corridor(a, b)
  -- center of each room
  local ax = a.x + flr(a.w/2)
  local ay = a.y + flr(a.h/2)
  local bx = b.x + flr(b.w/2)
  local by = b.y + flr(b.h/2)

  -- horizontal then vertical (L-shaped corridor)
  local mx = min(ax, bx)
  local mw = abs(bx - ax)
  for x = mx, mx+mw do
    mset(x, ay, 1)
  end
  local my = min(ay, by)
  local mh = abs(by - ay)
  for y = my, my+mh do
    mset(bx, y, 1)
  end
end
```

---

## Step 6 — Scattering Objects

After generating terrain/rooms, scatter enemies, items, and decorations:

```lua
function scatter_objects()
  enemies = {}
  coins   = {}

  for i = 1, 10 do
    -- find a random open floor tile
    local attempts = 0
    repeat
      local tx = flr(rnd(128))
      local ty = flr(rnd(64))
      attempts += 1
      if not fget(mget(tx, ty), 0) then   -- not a wall
        if rnd(1) < 0.5 then
          make_enemy(tx*8, ty*8)
        else
          make_coin(tx*8, ty*8)
        end
        break
      end
    until attempts > 50   -- give up if no open tile found
  end
end
```

---

## Step 7 — Procedural Background (Stars / Clouds)

Static decorative backgrounds using `srand()` so they don't flicker:

```lua
function draw_bg_stars()
  -- save current random state, use a fixed seed, then restore
  srand(1)   -- always the same stars
  for i = 1, 60 do
    local x = rnd(128)
    local y = rnd(128)
    local c = rnd(1) < 0.3 and 7 or 5   -- mostly dim, some bright
    pset(x, y, c)
  end
  srand(time())   -- restore time-based randomness
end

function draw_clouds(scroll_x)
  srand(42)
  for i = 1, 8 do
    local cx = (rnd(200) - scroll_x * 0.2) % 140 - 8   -- parallax scroll
    local cy = rnd(50)
    local cw = flr(rnd(20)) + 10
    rectfill(cx, cy, cx+cw, cy+6, 7)
    rectfill(cx+4, cy-4, cx+cw-4, cy, 7)
  end
  srand(time())
end
```

---

## Step 8 — Noise-Like Variation

PICO-8 doesn't have built-in Perlin noise, but you can fake smooth variation:

```lua
-- smooth a heightmap by averaging neighbors
function smooth_terrain(ground, passes)
  passes = passes or 2
  for p = 1, passes do
    for i = 2, #ground-1 do
      ground[i] = (ground[i-1] + ground[i] + ground[i+1]) / 3
    end
  end
end

-- use after generating:
make_terrain()
smooth_terrain(ground, 3)
```

Or use layered sine waves for an interesting landscape:

```lua
function wave_terrain()
  ground = {}
  for i = 1, 128 do
    local h = 70
    h += sin(i * 0.03) * 20      -- big slow waves
    h += sin(i * 0.08) * 8       -- medium waves
    h += sin(i * 0.2)  * 3       -- small detail
    ground[i] = flr(h)
  end
end
```

---

## Challenge

1. Generate a **different dungeon** each run using `srand(time())`
2. Add **guaranteed player start and exit positions** to the dungeon (first room = spawn, last room = exit)
3. Make the cave **get harder** over time (narrow the ceiling and floor limits)
4. Generate a **treasure room** — a room with more coins than usual, guarded by more enemies

---

## What You Learned

- `srand(n)` seeds the RNG for reproducible or random-each-run generation
- Controlled randomness: small variations from previous values = natural feeling
- Heightmap: array of y values, one per column
- Cave generation: rolling window of columns, delete old, add new
- Dungeon: carve rooms then connect with L-shaped corridors
- `mset()` writes to the map at runtime
- Use `srand(fixed)` / `srand(time())` to bracket decorative drawing

**Next:** [09-polish.md](09-polish.md) — sound effects, music, game states, particles, and save data.
