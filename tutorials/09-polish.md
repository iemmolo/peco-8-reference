---
tags: [pico8, tutorial]
---
# Tutorial 09 — Polish

**What you'll learn:** Sound effects, music, game states (menu/play/game-over), particles, screen shake, save data, and publishing tips.

**Prerequisites:** All previous tutorials.

---

## Why Polish Matters

A game with good feel is 10× more enjoyable than the same game without it. Polish isn't making it look pretty — it's the **feedback** the player gets from every action. A jump with a sound, a screen shake when you land, a particle burst when you collect a coin — these make the game feel alive.

---

## Part 1 — Sound Effects

### Quick SFX Setup

Open the Sound Editor (ALT+→ to switch editors). In SFX slot #0:
- Pick instrument 0 (triangle)
- Draw a short rising pattern of 2-3 notes
- Set speed to 5

That's your jump sound. Make a few more:
```
SFX #0 = jump
SFX #1 = land / footstep
SFX #2 = hit / hurt
SFX #3 = coin pickup
SFX #4 = game over
SFX #5 = level complete
```

### Playing SFX

```lua
sfx(0)          -- play SFX 0 on any free channel
sfx(0, 0)       -- play on channel 0 specifically
sfx(-1, 0)      -- stop channel 0
```

### When to play sounds

Add sounds to actions, not states:

```lua
-- jump
if btnp(4) and on_ground then
  pvy = jump_force
  sfx(0)          -- JUMP sound
end

-- land (when on_ground becomes true)
-- add  was_on_ground = false  to your _init()
if not was_on_ground and on_ground then
  sfx(1)          -- LAND sound
end
was_on_ground = on_ground   -- put this at END of _update()

-- take damage
function player_hurt()
  hp -= 1
  sfx(2)          -- HURT sound
  inv_t = 60      -- invincibility frames
end

-- collect coin
if aabb(px,py,8,8, c.x,c.y,8,8) then
  score += 10
  sfx(3)          -- COIN sound
  del(coins, c)
end
```

---

## Part 2 — Music

### Setup

In the Music Editor, build a pattern:
- Pattern #0 = main gameplay loop
- Pattern #1 = menu music
- Pattern #2 = game over sting

### Playing Music

```lua
music(0)          -- play pattern 0 (loops automatically if loop marker set)
music(0, 500)     -- fade in over 500ms
music(-1)         -- stop
music(-1, 1000)   -- fade out over 1 second
```

### Switching music with game states

```lua
function game_init()
  music(0)        -- start gameplay music
end

function gameover_init()
  music(-1, 500)  -- fade out
  sfx(4)          -- game over sting
end

function menu_init()
  music(1)        -- menu music
end
```

---

## Part 3 — Game States

Most games have at least three states: **menu**, **gameplay**, **game over**. The cleanest way is to swap what `_update` and `_draw` point to.

```lua
-- === MENU STATE ===

function menu_init()
  _update = menu_update
  _draw   = menu_draw
  music(1)
end

function menu_update()
  if btnp(4) then game_init() end
end

function menu_draw()
  cls(1)
  print("my game", 44, 40, 7)
  print("press \x8e to start", 28, 60, 6)
  -- animated logo, etc.
end

-- === GAMEPLAY STATE ===

function game_init()
  -- reset everything
  score    = 0
  hp       = 3
  make_player()
  enemies  = {}
  coins    = {}
  music(0)
  _update  = game_update
  _draw    = game_draw
end

function game_update()
  update_player()
  foreach(enemies, update_enemy)
  foreach(coins,   update_coin)
  check_hits()
  if hp <= 0 then gameover_init() end
end

function game_draw()
  cls()
  camera(cam_x, 0)
  map(0, 0, 0, 0, 128, 16)
  foreach(enemies, draw_enemy)
  foreach(coins,   draw_coin)
  draw_player()
  camera()
  draw_hud()
end

-- === GAME OVER STATE ===

function gameover_init()
  _update = gameover_update
  _draw   = gameover_draw
  music(-1, 500)
  sfx(4)
end

function gameover_update()
  if btnp(4) then game_init() end   -- retry
  if btnp(5) then menu_init() end   -- back to menu
end

function gameover_draw()
  cls(0)
  print("game over",     44, 48, 8)
  print("score:"..score, 40, 60, 7)
  print("press \x8e retry", 30, 80, 6)
  print("press \x97 menu",  30, 90, 5)
end

-- === BOOT ===

function _init()
  menu_init()
end
```

### The trick

`_update` and `_draw` are variables that hold functions. You can reassign them. When PICO-8 calls `_update()`, it calls whatever function is currently stored in `_update`. Swapping the state = swapping the function.

---

## Part 4 — Invincibility Frames

After taking damage, the player can't take damage again for a short time. Flash the sprite to signal it.

```lua
function _init()
  hp    = 3
  inv_t = 0      -- invincibility timer (frames)
end

function player_hurt()
  if inv_t > 0 then return end   -- still invincible, ignore hit
  hp    -= 1
  inv_t  = 60   -- 2 seconds invincible at 30fps
  sfx(2)
end

-- in _update():
if inv_t > 0 then inv_t -= 1 end

-- in _draw(): flash the player sprite
function draw_player()
  -- visible every other 4-frame group when invincible
  if inv_t > 0 and inv_t % 8 < 4 then return end
  spr(1, px, py)
end
```

---

## Part 5 — Screen Shake

A tiny shake makes hits, landings, and explosions feel impactful.

```lua
function _init()
  shake_t   = 0
  shake_mag = 0
end

function shake(mag, duration)
  shake_mag = mag
  shake_t   = duration
end

-- in _update():
if shake_t > 0 then
  shake_t   -= 1
  shake_mag *= 0.9   -- decay magnitude
end

-- in _draw(), before camera():
local ox, oy = 0, 0
if shake_t > 0 then
  ox = rnd(shake_mag*2) - shake_mag
  oy = rnd(shake_mag*2) - shake_mag
end
camera(cam_x + ox, cam_y + oy)
-- ... draw world ...
camera()

-- trigger with:
shake(4, 12)   -- magnitude 4, 12 frames — for a hit
shake(2, 8)    -- magnitude 2, 8 frames  — for landing
shake(8, 20)   -- magnitude 8, 20 frames — for explosion
```

---

## Part 6 — Particles (Quick Version)

See `snippets/particles.md` for the full system. Here's the minimum:

```lua
ps = {}   -- particle system

function burst(x, y, col, n)
  n = n or 8
  for i = 1, n do
    add(ps, {
      x = x, y = y,
      dx = rnd(4)-2,
      dy = rnd(3)*-1,
      life = flr(rnd(15))+8,
      col = col
    })
  end
end

function update_ps()
  for p in all(ps) do
    p.x += p.dx
    p.y += p.dy
    p.dy += 0.1    -- gravity
    p.life -= 1
    if p.life <= 0 then del(ps, p) end
  end
end

function draw_ps()
  for p in all(ps) do pset(p.x, p.y, p.col) end
end

-- trigger:
burst(ex, ey, 9, 12)   -- orange burst at enemy position
```

---

## Part 7 — HUD (Heads-Up Display)

Draw health bars, score, lives, and timers. Always reset `camera()` first.

```lua
function draw_hud()
  camera()   -- always reset before HUD

  -- health bar
  rectfill(2, 2, 2 + max_hp*8, 6, 5)     -- background bar (dark gray)
  rectfill(2, 2, 2 + hp*8,     6, 8)     -- fill bar (red to green)

  -- lives as icons
  for i = 1, lives do
    spr(4, 2 + (i-1)*9, 8)
  end

  -- score (right-aligned)
  local s = "score:"..score
  print(s, 127 - #s*4, 2, 7)

  -- wave/level
  print("wave "..wave, 54, 2, 6)

  -- timer
  local t = flr(time_limit - time())
  local col = t < 10 and 8 or 7   -- red when low
  print(t, 60, 2, col)
end
```

---

## Part 8 — Save Data

`cartdata()` gives you 64 persistent number slots that survive cart restarts.

```lua
function _init()
  cartdata("mygame_v1")   -- unique ID string — call only once, in _init

  -- load saved values
  hi_score       = dget(0)
  levels_cleared = dget(1)
end

function save_progress()
  if score > hi_score then
    hi_score = score
    dset(0, hi_score)     -- save to slot 0
  end
  dset(1, levels_cleared) -- save to slot 1
end

-- call save_progress() when appropriate (level complete, game over, etc.)
```

**Important rules:**
- Call `cartdata()` once, in `_init()`, never again
- Use a version suffix in your ID (`"mygame_v1"`) — change it if you restructure your save slots or you'll get corrupted data
- Slots 0–63, each holds one PICO-8 number

---

## Part 9 — Publishing

### Set a cart title

```lua
-- first two lines of code (as comments):
-- my game title
-- by your name
```

### Capture the label image

While your game looks good, press `CTRL+7` (F7). This captures a screenshot to use as the cart's label image. Save after.

### Save as PNG (shareable cart)

In console:
```
save mygame.p8.png
```

This creates a PNG image that contains your entire game (code, art, music) hidden in its pixels. Anyone with PICO-8 can load it.

### Export as HTML

```
export mygame.html
```

Creates two files (`mygame.html` + `mygame.js`). Open the HTML in any browser to play. Upload both files to itch.io to publish.

---

## Polish Checklist

Before calling a game done, go through this:

```
[ ] Every action has a sound
[ ] Screen shake on impactful events (hits, explosions, landing)
[ ] Particle burst on deaths, pickups, impacts
[ ] Invincibility frames after taking damage
[ ] Score displayed clearly
[ ] Game over and restart work reliably
[ ] Music that fits the mood
[ ] Menu screen (even a simple one)
[ ] High score saved with cartdata
[ ] Cart label image set (CTRL+7)
```

---

## Complete Game Template

```lua
-- mygame v1
-- by you

function _init()
  cartdata("mygame_v1")
  hi = dget(0)
  shake_t, shake_mag = 0, 0
  ps = {}
  menu_init()
end

function _update() end   -- replaced by state init
function _draw()   end

-- paste menu_init, game_init, gameover_init functions here
-- paste update/draw functions for each state here
-- paste player, enemy, bullet, coin functions here
-- paste burst(), update_ps(), draw_ps() here
-- paste shake() here

-- call:
-- menu_init() in _init()
-- shake(mag, dur) on impacts
-- burst(x, y, col, n) on deaths/pickups
-- sfx(n) on every key moment
-- music(n) when state changes
```

---

## Challenge

1. Add a **pause menu** using `menuitem()` — "resume", "restart", "quit to menu"
2. Add a **high score screen** that shows top 5 scores (store them in cartdata slots 0–4)
3. Make your **menu animated** — spinning logo, pulsing text, particle rain
4. Add a **combo multiplier** — collect coins quickly to multiply score

---

## What You Learned

- `sfx(n)` on every impactful action — this matters more than any visual
- Game states via `_update`/`_draw` function swapping
- Invincibility frames: ignore damage + flash sprite
- Screen shake: random camera offset, decaying magnitude
- Particles: table of short-lived moving pixels
- HUD: always reset `camera()` first
- `cartdata()` + `dget()`/`dset()` for persistent saves
- Publish as `.p8.png` (share) or `.html` (web)

---

## You're Done!

You've now covered everything needed to build a complete game:

| Tutorial | Covered |
|----------|---------|
| 01 | Game loop, shapes, colors |
| 02 | Sprites, animation, flip |
| 03 | Input, velocity, friction |
| 04 | Tilemap, camera, scrolling |
| 05 | Pong — first full game |
| 06 | Gravity, jump, tile physics |
| 07 | Entities, enemies, bullets |
| 08 | Procedural generation |
| 09 | Polish, states, sound, saving |

From here: pick a game idea you like, start with the simplest possible version, and build it up one step at a time — exactly like you did with Pong. Publish early, get feedback, iterate.

Good luck!
