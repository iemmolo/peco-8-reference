# Camera Snippets

## Basic Follow Camera

```lua
-- center camera on player (player is at screen center)
function update_camera()
  cam_x = px - 60
  cam_y = py - 60
end

function _draw()
  cls()
  camera(cam_x, cam_y)
  -- draw world here (map, enemies, etc.)
  map(0, 0, 0, 0, 128, 64)
  spr(1, px, py)
  camera()         -- reset before drawing HUD
  print("score:"..score, 2, 2, 7)
end
```

---

## Camera Clamped to World Bounds

```lua
-- world size in pixels
local world_pw = 128 * 8   -- e.g. 128 tiles wide
local world_ph = 64 * 8    -- e.g. 64 tiles tall

function update_camera()
  cam_x = px - 60
  cam_y = py - 60

  -- clamp so we don't show outside the world
  cam_x = mid(0, cam_x, world_pw - 128)
  cam_y = mid(0, cam_y, world_ph - 128)
end
```

---

## Smooth Follow Camera (Lerp)

Camera lazily drifts toward player — feels more organic.

```lua
function _init()
  cam_x = px - 60
  cam_y = py - 60
end

function update_camera()
  local target_x = px - 60
  local target_y = py - 60
  local smooth = 0.1  -- lower = slower follow (0.05-0.2)

  cam_x += (target_x - cam_x) * smooth
  cam_y += (target_y - cam_y) * smooth

  -- clamp to world bounds
  cam_x = mid(0, cam_x, world_pw - 128)
  cam_y = mid(0, cam_y, world_ph - 128)
end
```

---

## Camera with Deadzone

Player can move within a zone before the camera follows.

```lua
local dead_w = 32    -- deadzone width
local dead_h = 24    -- deadzone height

function update_camera()
  -- only move camera if player exits the center zone
  local cx = cam_x + 64  -- center of screen in world coords
  local cy = cam_y + 64

  if px < cx - dead_w then cam_x = px - 64 + dead_w end
  if px > cx + dead_w then cam_x = px - 64 - dead_w end
  if py < cy - dead_h then cam_y = py - 64 + dead_h end
  if py > cy + dead_h then cam_y = py - 64 - dead_h end

  cam_x = mid(0, cam_x, world_pw - 128)
  cam_y = mid(0, cam_y, world_ph - 128)
end
```

---

## Screen Shake

```lua
function _init()
  shake_t = 0
  shake_mag = 0
  cam_x = 0
  cam_y = 0
end

function shake(mag, duration)
  shake_mag = mag
  shake_t = duration
end

function update_camera()
  -- base camera logic (e.g. follow player)
  local bx = px - 60
  local by = py - 60

  -- add shake offset
  local ox, oy = 0, 0
  if shake_t > 0 then
    ox = (rnd(shake_mag*2) - shake_mag)
    oy = (rnd(shake_mag*2) - shake_mag)
    shake_t -= 1
    shake_mag *= 0.9  -- fade shake
  end

  camera(bx + ox, by + oy)
end

-- trigger with:
shake(4, 15)   -- magnitude 4, 15 frames
```

---

## Parallax Scrolling

Multiple layers moving at different speeds for depth effect.

```lua
-- in _draw()
function draw_world()
  -- background layer (moves at 25% of camera speed)
  camera(flr(cam_x * 0.25), flr(cam_y * 0.25))
  map(0, 48, 0, 0, 16, 4)   -- background tiles at map row 48

  -- midground layer (50%)
  camera(flr(cam_x * 0.5), flr(cam_y * 0.5))
  map(0, 52, 0, 0, 16, 4)   -- midground tiles at map row 52

  -- foreground (full camera speed)
  camera(cam_x, cam_y)
  map(0, 0, 0, 0, 16, 32)   -- main map
  spr(1, px, py)
end
```

---

## Room-to-Room Transition

```lua
local room_x = 0
local room_y = 0

function update_camera()
  -- snap camera to current room (rooms are 16x16 tiles = 128x128 px)
  cam_x = room_x * 128
  cam_y = room_y * 128
end

function check_room_transition()
  if px < 0 then
    room_x -= 1
    px = 120
  elseif px > 127 then
    room_x += 1
    px = 0
  end
  if py < 0 then
    room_y -= 1
    py = 120
  elseif py > 127 then
    room_y += 1
    py = 0
  end
end
```

---

## Draw HUD Over Camera

Always reset camera before drawing UI elements:

```lua
function _draw()
  cls()
  camera(cam_x, cam_y)

  -- world drawing here
  map(0, 0, 0, 0, 128, 64)
  spr(1, px, py)

  -- reset camera for HUD
  camera()

  -- draw HUD at fixed screen positions
  rectfill(0, 0, 127, 8, 0)    -- black bar
  print("hp:"..hp, 2, 1, 8)
  print("score:"..score, 60, 1, 7)
end
```
