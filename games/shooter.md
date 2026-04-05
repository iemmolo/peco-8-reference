# Shooter (Pong → Space Shooter)

Full annotated vertical shooter with bullets, enemies, waves, and score.

---

## Quick Tips

- Bullets and enemies are tables of tables — loop with `foreach`
- Delete entities by `del(table, entity)` inside the loop with `all()`
- Enemy waves: use a timer to spawn from the top
- Check bullet/enemy collision with AABB
- HUD: reset `camera()` before drawing

---

## Pong (Beginner Stepping Stone)

From the pico-guide fanzine — great intro to game logic.

```lua
-- paddle
padx = 52
pady = 122
padw = 24
padh = 4

-- ball
ballx = 64
bally = 64
ballsize = 3
ballxdir = 5
ballydir = -3
score = 0
lives = 3

function _update()
  movepaddle()
  bounceball()
  bouncepaddle()
  moveball()
  losedeadball()
end

function movepaddle()
  if btn(0) then padx -= 3 end
  if btn(1) then padx += 3 end
  padx = mid(0, padx, 128 - padw)   -- clamp to screen
end

function moveball()
  ballx += ballxdir
  bally += ballydir
end

function bounceball()
  if ballx < ballsize then ballxdir = abs(ballxdir)  sfx(0) end
  if ballx > 128-ballsize then ballxdir = -abs(ballxdir)  sfx(0) end
  if bally < ballsize then ballydir = abs(ballydir)  sfx(0) end
end

function bouncepaddle()
  if ballx >= padx and ballx <= padx+padw and bally > pady then
    ballydir = -abs(ballydir)
    score += 10
    sfx(0)
  end
end

function losedeadball()
  if bally > 128 - ballsize then
    if lives > 0 then
      bally = 24
      lives -= 1
      sfx(1)
    else
      ballxdir = 0
      ballydir = 0
    end
  end
end

function _draw()
  rectfill(0, 0, 128, 128, 1)       -- dark blue bg
  for i = 1, lives do spr(4, 90+i*8, 4) end
  print(score, 12, 6, 7)
  rectfill(padx, pady, padx+padw, pady+padh, 7)
  circfill(ballx, bally, ballsize, 7)
end
```

---

## Full Space Shooter

```lua
-- === CONSTANTS ===
player_spd  = 2
bullet_spd  = 4
enemy_spd   = 1
fire_delay  = 10   -- frames between shots

-- === INIT ===

function _init()
  make_player()
  enemies  = {}
  bullets  = {}
  e_bullets = {}
  score    = 0
  wave     = 1
  spawn_t  = 0
  spawn_interval = 60
  lives    = 3
  game_over = false
  music(0)
end

-- === PLAYER ===

function make_player()
  p = {}
  p.x       = 60
  p.y       = 100
  p.fire_t  = 0
end

function update_player()
  -- move
  if btn(0) then p.x -= player_spd end
  if btn(1) then p.x += player_spd end
  if btn(2) then p.y -= player_spd end
  if btn(3) then p.y += player_spd end

  -- clamp to screen
  p.x = mid(0, p.x, 120)
  p.y = mid(64, p.y, 120)   -- stay in lower half

  -- shoot
  p.fire_t -= 1
  if btn(4) and p.fire_t <= 0 then
    shoot_bullet(p.x+3, p.y, 0, -bullet_spd, true)
    p.fire_t = fire_delay
    sfx(0)
  end
end

function draw_player()
  spr(1, p.x, p.y)   -- player sprite
end

-- === BULLETS ===

function shoot_bullet(x, y, dx, dy, is_player)
  add(bullets, {x=x, y=y, dx=dx, dy=dy, player=is_player})
end

function update_bullets()
  for b in all(bullets) do
    b.x += b.dx
    b.y += b.dy
    -- remove off-screen
    if b.y < -4 or b.y > 132 or b.x < -4 or b.x > 132 then
      del(bullets, b)
    end
  end
end

function draw_bullets()
  for b in all(bullets) do
    if b.player then
      rectfill(b.x+2, b.y, b.x+3, b.y+4, 10)   -- yellow
    else
      circfill(b.x+2, b.y+2, 2, 8)              -- red
    end
  end
end

-- === ENEMIES ===

function spawn_enemy()
  local e = {}
  e.x    = rnd(112)
  e.y    = -8
  e.hp   = 1 + flr(wave / 3)
  e.spd  = enemy_spd + wave * 0.1
  e.fire_t = flr(rnd(60)) + 30   -- stagger firing
  e.type   = (rnd(1) < 0.3) and 2 or 1   -- 30% tanky type
  add(enemies, e)
end

function update_enemies()
  -- spawn timer
  spawn_t -= 1
  if spawn_t <= 0 then
    for i = 1, wave do
      spawn_enemy()
    end
    spawn_t = spawn_interval
    spawn_interval = max(20, spawn_interval - 5)
    wave += 1
  end

  for e in all(enemies) do
    -- move down
    e.y += e.spd

    -- simple sine weave
    e.x += sin(e.y * 0.02) * 1.5

    -- remove if off-screen
    if e.y > 132 then del(enemies, e) end

    -- enemy shoots downward occasionally
    e.fire_t -= 1
    if e.fire_t <= 0 then
      shoot_bullet(e.x+3, e.y+8, 0, 2, false)
      e.fire_t = flr(rnd(90)) + 30
    end
  end
end

function draw_enemies()
  for e in all(enemies) do
    spr(2, e.x, e.y)       -- enemy sprite
    -- health bar for tanky enemies
    if e.hp > 1 then
      rectfill(e.x, e.y-3, e.x+8, e.y-2, 0)
      rectfill(e.x, e.y-3, e.x + e.hp*3, e.y-2, 8)
    end
  end
end

-- === COLLISION ===

function aabb(ax,ay,aw,ah,bx,by,bw,bh)
  return ax<bx+bw and ax+aw>bx and ay<by+bh and ay+ah>by
end

function check_collisions()
  -- player bullets vs enemies
  for b in all(bullets) do
    if b.player then
      for e in all(enemies) do
        if aabb(b.x, b.y, 4, 4, e.x, e.y, 8, 8) then
          e.hp -= 1
          del(bullets, b)
          if e.hp <= 0 then
            del(enemies, e)
            score += 100 * wave
            sfx(2)
            -- spawn explosion particles (see particles.md)
          end
          break
        end
      end
    end
  end

  -- enemy bullets vs player
  for b in all(bullets) do
    if not b.player then
      if aabb(b.x, b.y, 4, 4, p.x+1, p.y+1, 6, 6) then
        del(bullets, b)
        player_hit()
      end
    end
  end

  -- enemies vs player (ramming)
  for e in all(enemies) do
    if aabb(e.x, e.y, 8, 8, p.x+1, p.y+1, 6, 6) then
      del(enemies, e)
      player_hit()
    end
  end
end

-- === DAMAGE ===
inv_t = 0

function player_hit()
  if inv_t > 0 then return end
  lives -= 1
  inv_t = 90   -- 3 seconds invincible
  sfx(3)
  if lives <= 0 then
    gameover()
  end
end

-- === BACKGROUND ===

function _init_stars()
  stars = {}
  for i = 1, 50 do
    add(stars, {x=rnd(128), y=rnd(128), spd=rnd(1)+0.5, c=rnd({5,6,7})})
  end
end

function update_stars()
  for s in all(stars) do
    s.y += s.spd
    if s.y > 128 then s.y = 0  s.x = rnd(128) end
  end
end

function draw_stars()
  for s in all(stars) do pset(s.x, s.y, s.c) end
end

-- === GAME LOOP ===

function _update()
  if inv_t > 0 then inv_t -= 1 end
  update_player()
  update_bullets()
  update_enemies()
  check_collisions()
  update_stars()
end

function _draw()
  cls(1)    -- dark blue sky
  draw_stars()
  draw_bullets()
  draw_enemies()

  -- player flicker when invincible
  if inv_t == 0 or inv_t % 4 < 2 then
    draw_player()
  end

  -- HUD
  print("score:"..score, 2, 2, 7)
  print("wave:"..wave,   80, 2, 7)
  for i = 1, lives do
    spr(4, 2 + (i-1)*9, 118)   -- life icons
  end
end

-- === GAME OVER ===

function gameover()
  _update = function()
    if btnp(4) then _init() end
  end
  _draw = function()
    cls(1)
    print("game over",    44, 48, 8)
    print("score:"..score, 40, 60, 7)
    print("wave:"..wave,   44, 70, 6)
    print("press \x8e",    44, 84, 5)
  end
  music(-1, 500)
  sfx(5)
end
```

---

## Extension Ideas

### Power-ups
```lua
powerups = {}

function spawn_powerup(x, y)
  add(powerups, {x=x, y=y, type=rnd({"spread","shield","speed"})})
end

function update_powerups()
  for pu in all(powerups) do
    pu.y += 1
    if pu.y > 132 then del(powerups, pu) end
    if aabb(p.x, p.y, 8, 8, pu.x, pu.y, 8, 8) then
      apply_powerup(pu.type)
      del(powerups, pu)
    end
  end
end
```

### Spread Shot
```lua
function shoot_spread()
  shoot_bullet(p.x+3, p.y, -1, -bullet_spd, true)
  shoot_bullet(p.x+3, p.y,  0, -bullet_spd, true)
  shoot_bullet(p.x+3, p.y,  1, -bullet_spd, true)
end
```

### Boss Enemy
```lua
function spawn_boss()
  boss = {x=44, y=-16, hp=30, max_hp=30, phase=1}
end

function update_boss()
  if not boss then return end
  -- phase 1: oscillate
  if boss.phase == 1 then
    boss.x = 60 + cos(time()*0.2) * 40
    boss.y = approach(boss.y, 20, 0.5)
  end
  -- phase 2 at half hp
  if boss.hp <= 15 then boss.phase = 2 end
end
```
