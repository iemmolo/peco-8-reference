# Side-Scroller — Cave Diver

A one-button side-scrolling cave game (Flappy Bird style). Based on the Gamedev with PICO-8 zine tutorial.

---

## Quick Tips

- Cave is a table of columns, each with a `top` and `btm` height
- Scroll by removing front columns and adding new ones each frame
- Player only moves up/down — the cave scrolls past them
- `btnp(2)` (UP) for jump; gravity pulls them down
- Collision = check player bounding box against cave column heights

---

## Sprites Needed

```
Sprite #1  rising player (wings up)
Sprite #2  falling player (wings down)
Sprite #3  dead player (Xs for eyes)
```

---

## Full Cartridge

```lua
-- === INIT ===

function _init()
  game_over = false
  make_cave()
  make_player()
end

-- === PLAYER ===

function make_player()
  player = {}
  player.x      = 24
  player.y      = 60
  player.dy     = 0
  player.rise   = 1    -- sprite when going up
  player.fall   = 2    -- sprite when going down
  player.dead   = 3    -- sprite when dead
  player.speed  = 2    -- cave scroll speed
  player.score  = 0
end

function move_player()
  player.dy += 0.2          -- gravity

  if btnp(2) then            -- jump on UP button
    player.dy = -5
    sfx(0)
  end

  player.y      += player.dy
  player.score  += player.speed
end

function draw_player()
  local s
  if game_over then
    s = player.dead
  elseif player.dy < 0 then
    s = player.rise
  else
    s = player.fall
  end
  spr(s, player.x, player.y)
end

-- === CAVE ===

function make_cave()
  -- start with one column
  cave = {{["top"]=5, ["btm"]=119}}
  top = 45   -- how low ceiling can go
  btm = 85   -- how high floor can get
end

function update_cave()
  -- remove old columns from the front
  if #cave > player.speed then
    for i = 1, player.speed do
      del(cave, cave[1])
    end
  end

  -- add new columns at the back
  while #cave < 128 do
    local col = {}
    local up  = flr(rnd(7) - 3)
    local dwn = flr(rnd(7) - 3)
    col.top = mid(3,   cave[#cave].top + up,  top)
    col.btm = mid(btm, cave[#cave].btm + dwn, 124)
    add(cave, col)
  end
end

function draw_cave()
  for i = 1, #cave do
    line(i-1, 0,   i-1, cave[i].top, 5)
    line(i-1, 127, i-1, cave[i].btm, 5)
  end
end

-- === COLLISION ===

function check_hit()
  for i = player.x, player.x + 7 do
    if cave[i+1].top > player.y or
       cave[i+1].btm < player.y + 7 then
      game_over = true
      sfx(1)
    end
  end
end

-- === GAME LOOP ===

function _update()
  if not game_over then
    update_cave()
    move_player()
    check_hit()
  else
    if btnp(5) then _init() end   -- press X to restart
  end
end

function _draw()
  cls()
  draw_cave()
  draw_player()

  if game_over then
    print("game over!",         44, 44, 7)
    print("score: "..player.score, 34, 54, 7)
    print("press \x97 to retry", 20, 68, 6)
  else
    print("score: "..player.score, 2, 2, 7)
  end
end
```

---

## How to Extend

### Make it harder over time
```lua
function _update()
  if not game_over then
    -- increase scroll speed and tighten cave over time
    player.speed = 2 + flr(player.score / 500)
    top = mid(20, 45 - flr(player.score/800), 45)
    -- ... rest of update
  end
end
```

### Add speed boost pickups
```lua
pickups = {}

function add_pickup()
  if rnd(1) < 0.01 then   -- 1% chance per frame
    add(pickups, {x=128, y=rndb(30, 90)})
  end
end

function update_pickups()
  for p in all(pickups) do
    p.x -= player.speed
    if p.x < -8 then del(pickups, p) end
    -- check hit
    if abs(p.x - player.x) < 8 and abs(p.y - player.y) < 8 then
      player.score += 200
      sfx(2)
      del(pickups, p)
    end
  end
end
```

### Increase difficulty as score climbs
```lua
-- tighten the gap between top and btm
top = mid(20, 45 - flr(player.score / 1000), 45)
btm = mid(80, 85 + flr(player.score / 1000), 95)
```
