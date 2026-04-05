# Top-Down RPG / Adventure

Full annotated top-down game with tile movement, rooms, and interaction.

---

## Quick Tips

- 4-dir or 8-dir movement with tile collision
- Camera snaps to rooms (screen-sized chunks) or follows player
- Use `mget()`/`fget()` for tile-based collision and interaction
- Dialog: draw a text box over the game, use coroutines for letter-by-letter
- Game state pattern: `_update`/`_draw` swapped per mode

---

## Map Setup

```
Flag 0 = solid (walls, trees, rocks)
Flag 1 = interact (doors, signs, NPCs on map)
Flag 2 = water / hazard
```

- Room size: 16×16 tiles (= one screen)
- Rooms arranged on map: room(0,0) at tiles(0,0), room(1,0) at tiles(16,0), etc.

---

## Full Cartridge

```lua
-- === CONSTANTS ===
spd = 1.5

-- === INIT ===

function _init()
  make_player()
  room_x = 0
  room_y = 0
  update_camera()
  dialog = nil    -- active dialog string
  music(0)
end

-- === PLAYER ===

function make_player()
  p = {}
  p.x   = 60
  p.y   = 60
  p.dir = 2   -- facing: 0=up 1=right 2=down 3=left
  p.spr = {
    [0] = {5, 6},   -- up: walk frame 1 & 2
    [1] = {3, 4},   -- right
    [2] = {1, 2},   -- down
    [3] = {7, 8},   -- left
  }
  p.anim = 0
  p.step = 0
end

function update_player()
  if dialog then return end   -- freeze player during dialog

  local dx, dy = 0, 0
  if btn(0) then dx = -1  p.dir = 3 end
  if btn(1) then dx =  1  p.dir = 1 end
  if btn(2) then dy = -1  p.dir = 0 end
  if btn(3) then dy =  1  p.dir = 2 end

  -- move and check collision
  if dx ~= 0 or dy ~= 0 then
    local nx = p.x + dx * spd
    local ny = p.y + dy * spd
    if not solid_at(nx + 2, ny + 4) and
       not solid_at(nx + 6, ny + 4) and
       not solid_at(nx + 2, ny + 7) and
       not solid_at(nx + 6, ny + 7) then
      p.x = nx
      p.y = ny
    end
    p.anim += 1
    p.step = flr(p.anim / 8) % 2   -- alternate walk frames
  else
    p.step = 0
    p.anim = 0
  end

  -- interact
  if btnp(4) then interact() end

  -- check room transition
  check_room_exit()
end

function draw_player()
  local s = p.spr[p.dir][p.step + 1]
  spr(s, p.x, p.y)
end

-- === TILE HELPERS ===

function solid(tx, ty)
  return fget(mget(tx, ty), 0)
end

function solid_at(wx, wy)
  local tx = flr(wx / 8) + room_x * 16
  local ty = flr(wy / 8) + room_y * 16
  return solid(tx, ty)
end

-- === INTERACTION ===

function interact()
  -- check tile in front of player
  local dirs = {
    [0] = {0, -8},   -- up
    [1] = {8,  0},   -- right
    [2] = {0,  8},   -- down
    [3] = {-8, 0},   -- left
  }
  local d = dirs[p.dir]
  local tx = flr((p.x + 4 + d[1]) / 8) + room_x * 16
  local ty = flr((p.y + 4 + d[2]) / 8) + room_y * 16

  if fget(mget(tx, ty), 1) then
    -- interactable tile — show dialog
    show_dialog("...")   -- replace with actual dialog lookup
  end
end

-- === DIALOG ===

function show_dialog(text)
  dialog = {text=text, chr=0, timer=0}
  _update_old = _update
  _draw_old   = _draw
  _update = dialog_update
  _draw   = dialog_draw
end

function dialog_update()
  _update_old()   -- still run world (frozen because dialog ~= nil check)

  dialog.timer += 1
  if dialog.timer % 2 == 0 and dialog.chr < #dialog.text then
    dialog.chr += 1
  end

  if btnp(4) then
    if dialog.chr < #dialog.text then
      dialog.chr = #dialog.text   -- skip to end
    else
      dialog = nil                -- close
      _update = _update_old
      _draw   = _draw_old
    end
  end
end

function dialog_draw()
  _draw_old()
  -- draw text box
  rectfill(4, 96, 123, 123, 0)
  rect(4, 96, 123, 123, 7)
  -- draw text up to current character
  print(sub(dialog.text, 1, dialog.chr), 8, 100, 7)
end

-- === ROOM TRANSITIONS ===

function check_room_exit()
  if p.x < -4 then
    room_x -= 1
    p.x = 120
    update_camera()
  elseif p.x > 128 then
    room_x += 1
    p.x = 0
    update_camera()
  end
  if p.y < -4 then
    room_y -= 1
    p.y = 120
    update_camera()
  elseif p.y > 128 then
    room_y += 1
    p.y = 0
    update_camera()
  end
end

function update_camera()
  cam_x = room_x * 128
  cam_y = room_y * 128
end

-- === GAME LOOP ===

function _update()
  update_player()
end

function _draw()
  cls()
  camera(cam_x, cam_y)
  map(room_x*16, room_y*16, 0, 0, 16, 16)
  draw_player()
  camera()
end
```

---

## Extension Ideas

### NPCs (table-based)
```lua
npcs = {
  {x=64, y=48, spr=10, text="hello traveller!"},
  {x=96, y=80, spr=11, text="the dungeon is east."},
}

function draw_npcs()
  for n in all(npcs) do
    spr(n.spr, n.x + cam_x_offset, n.y + cam_y_offset)
  end
end

-- in interact(): check proximity to NPCs
for n in all(npcs) do
  if dist(p.x, p.y, n.x, n.y) < 16 then
    show_dialog(n.text)
    break
  end
end
```

### Simple Inventory
```lua
inventory = {}

function pick_up(item)
  add(inventory, item)
  show_dialog("found " .. item .. "!")
end

function has_item(item)
  for i in all(inventory) do
    if i == item then return true end
  end
  return false
end

-- gate a door:
if has_item("key") then open_door() end
```

### Day/Night Cycle
```lua
function get_darkness()
  local h = stat(93)   -- local hour
  if h >= 20 or h < 6 then
    return 0.6   -- dark
  else
    return 0.0   -- bright
  end
end

-- after drawing world, overlay with semi-transparent dark:
-- use pal() shifts or fillp() for dithered darkness effect
```

### Save / Load (cartdata)
```lua
function _init()
  cartdata("myrpg_v1")
  -- load saved position
  room_x = dget(0)
  room_y = dget(1)
  p.x    = dget(2)
  p.y    = dget(3)
  gold   = dget(4)
end

function save_game()
  dset(0, room_x)
  dset(1, room_y)
  dset(2, p.x)
  dset(3, p.y)
  dset(4, gold)
  show_dialog("game saved!")
end
```
