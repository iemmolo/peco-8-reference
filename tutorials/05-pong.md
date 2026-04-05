---
tags: [pico8, tutorial]
---
# Tutorial 05 — Pong (Your First Full Game)

**What you'll learn:** Building a complete game from scratch — game logic, collision, score, lives, and a restart screen.

**Prerequisites:** [[03-input-movement|03 - Input & Movement]]

This tutorial is adapted from the pico-guide fanzine by @TheRealMolen. A classic first game — it's simple enough to finish in one sitting but teaches every core concept you need.

---

## What We're Building

A single-player Pong (squash variant): you control a paddle at the bottom, keep the ball in play, score points each time you hit it, lose a life when you miss.

---

## Step 1 — The Paddle

Start a new cart (`reboot` in console), save it as `squashy`.

```lua
-- paddle variables
padx = 52     -- left edge x
pady = 122    -- top edge y (near bottom of screen)
padw = 24     -- width
padh = 4      -- height

function _update()
  movepaddle()
end

function _draw()
  rectfill(0, 0, 128, 128, 1)           -- dark blue background
  rectfill(padx, pady, padx+padw, pady+padh, 7)  -- white paddle
end

function movepaddle()
  if btn(0) then padx -= 3 end   -- left
  if btn(1) then padx += 3 end   -- right
  -- clamp so paddle stays on screen
  padx = mid(0, padx, 128 - padw)
end
```

**Save and run.** Your paddle should move left and right.

### What's new here?

- `rectfill(padx, pady, padx+padw, pady+padh, 7)` — note that `x1,y1` is top-left and `x2,y2` is bottom-right corner, so we add width/height to get the second corner.
- `mid(0, padx, 128-padw)` clamps the paddle — it can't go left of 0 or right of 128-24=104.

---

## Step 2 — Add the Ball

Add these variables at the top:

```lua
ballx    = 64
bally    = 64
ballsize = 3
ballxdir = 3    -- horizontal speed
ballydir = -3   -- vertical speed (negative = moving up)
```

Add the ball to `_draw()` and add a ball movement function:

```lua
function _update()
  movepaddle()
  moveball()
end

function _draw()
  rectfill(0, 0, 128, 128, 1)
  rectfill(padx, pady, padx+padw, pady+padh, 7)
  circfill(ballx, bally, ballsize, 7)   -- white ball
end

function moveball()
  ballx += ballxdir
  bally += ballydir
end
```

**Save and run.** The ball flies off. Next we'll make it bounce.

---

## Step 3 — Bounce off Walls

The ball bounces by flipping the sign of its direction when it hits an edge.

Add `bounceball()`:

```lua
function _update()
  movepaddle()
  bounceball()   -- check walls BEFORE moving
  moveball()
end

function bounceball()
  -- left wall
  if ballx < ballsize then
    ballxdir = abs(ballxdir)   -- force positive (move right)
    sfx(0)
  end
  -- right wall
  if ballx > 128 - ballsize then
    ballxdir = -abs(ballxdir)  -- force negative (move left)
    sfx(0)
  end
  -- top wall
  if bally < ballsize then
    ballydir = abs(ballydir)   -- force positive (move down)
    sfx(0)
  end
end
```

> [!TIP]
> Open the sound editor, go to SFX #0. Make a short, punchy sound — a single note with a slight drop (effect #3). Doesn't have to be perfect, just something audible.

**Save and run.** The ball bounces off three walls and eventually falls off the bottom.

### Why `abs()` instead of `-=`?

If we just did `ballxdir = -ballxdir`, the ball might clip inside the wall and bounce back and forth stuck. Using `abs()` or `-abs()` forces the direction away from the wall — always safe.

---

## Step 4 — Hit the Paddle

The ball hits the paddle if:
- Its X is within the paddle's left and right edges, **AND**
- Its Y has gone past the paddle's top edge

```lua
function _update()
  movepaddle()
  bounceball()
  bouncepaddle()   -- add this
  moveball()
end

function bouncepaddle()
  if ballx >= padx and
     ballx <= padx + padw and
     bally >= pady then
    ballydir = -abs(ballydir)   -- force upward
    sfx(0)
  end
end
```

**Save and run.** You can now keep the ball in play!

---

## Step 5 — Score

Add `score = 0` at the top, then add to score when the ball hits the paddle:

```lua
function bouncepaddle()
  if ballx >= padx and
     ballx <= padx + padw and
     bally >= pady then
    ballydir = -abs(ballydir)
    score += 10           -- score on hit
    sfx(0)
  end
end
```

Draw the score in `_draw()`:

```lua
function _draw()
  rectfill(0, 0, 128, 128, 1)
  print(score, 12, 6, 7)        -- score top-left
  rectfill(padx, pady, padx+padw, pady+padh, 7)
  circfill(ballx, bally, ballsize, 7)
end
```

---

## Step 6 — Lives

Add `lives = 3` at the top. When the ball falls off the bottom, lose a life and reset the ball:

```lua
function _update()
  movepaddle()
  bounceball()
  bouncepaddle()
  moveball()
  losedeadball()   -- add this
end

function losedeadball()
  if bally > 128 + ballsize then
    lives -= 1
    sfx(1)          -- make a lose-life sound in SFX #1

    if lives > 0 then
      -- reset ball
      bally    = 24
      ballxdir = 3
      ballydir = -3
    else
      -- stop ball (game over)
      ballxdir = 0
      ballydir = 0
      bally    = 64
    end
  end
end
```

Draw lives as heart sprites. First make a heart sprite in the editor (sprite #4), then:

```lua
function _draw()
  rectfill(0, 0, 128, 128, 1)

  -- lives (hearts)
  for i = 1, lives do
    spr(4, 90 + i*8, 4)
  end

  print(score, 12, 6, 7)
  rectfill(padx, pady, padx+padw, pady+padh, 7)
  circfill(ballx, bally, ballsize, 7)

  -- game over message
  if lives <= 0 then
    print("game over!", 40, 56, 7)
    print("score: "..score, 40, 68, 7)
  end
end
```

---

## Step 7 — Restart

After game over, let the player press X to restart:

```lua
function _update()
  if lives > 0 then
    movepaddle()
    bounceball()
    bouncepaddle()
    moveball()
    losedeadball()
  else
    if btnp(5) then
      -- restart everything
      score    = 0
      lives    = 3
      ballx    = 64
      bally    = 64
      ballxdir = 3
      ballydir = -3
    end
  end
end
```

Add a restart prompt to the game-over screen:

```lua
if lives <= 0 then
  print("game over!",  40, 56, 7)
  print("score: "..score, 40, 68, 7)
  print("press \x97 to play again", 18, 82, 6)  -- \x97 = X button glyph
end
```

---

## Full Final Code

```lua
padx = 52
pady = 122
padw = 24
padh = 4

ballx    = 64
bally    = 64
ballsize = 3
ballxdir = 3
ballydir = -3
score    = 0
lives    = 3

function _update()
  if lives > 0 then
    movepaddle()
    bounceball()
    bouncepaddle()
    moveball()
    losedeadball()
  else
    if btnp(5) then
      score=0 lives=3 ballx=64 bally=64 ballxdir=3 ballydir=-3
    end
  end
end

function _draw()
  rectfill(0,0,128,128,1)
  for i=1,lives do spr(4, 90+i*8, 4) end
  print(score, 12, 6, 7)
  rectfill(padx, pady, padx+padw, pady+padh, 7)
  circfill(ballx, bally, ballsize, 7)
  if lives <= 0 then
    print("game over!", 40, 56, 7)
    print("score: "..score, 40, 68, 7)
    print("press \x97 to retry", 18, 82, 6)
  end
end

function movepaddle()
  if btn(0) then padx -= 3 end
  if btn(1) then padx += 3 end
  padx = mid(0, padx, 128-padw)
end

function moveball()
  ballx += ballxdir
  bally += ballydir
end

function bounceball()
  if ballx < ballsize         then ballxdir =  abs(ballxdir) sfx(0) end
  if ballx > 128 - ballsize   then ballxdir = -abs(ballxdir) sfx(0) end
  if bally < ballsize         then ballydir =  abs(ballydir) sfx(0) end
end

function bouncepaddle()
  if ballx >= padx and ballx <= padx+padw and bally >= pady then
    ballydir = -abs(ballydir)
    score += 10
    sfx(0)
  end
end

function losedeadball()
  if bally > 128 + ballsize then
    lives -= 1
    sfx(1)
    if lives > 0 then
      bally=24 ballxdir=3 ballydir=-3
    else
      ballxdir=0 ballydir=0 bally=64
    end
  end
end
```

---

## Extension Ideas

1. **Speed up ball** after each hit: `ballxdir *= 1.05` in `bouncepaddle()`
2. **Angle the bounce** based on where the ball hits the paddle — hitting the edge sends it wider
3. **Two-player mode** — second paddle at the top controlled by player 2
4. **Brick layer** — add a row of colored rectfill'd bricks that the ball destroys
5. **Shrinking paddle** — make `padw` decrease by 1 every 50 points

---

## What You Learned

- Build a game in small steps — one piece at a time, run after each addition
- `abs()` and `-abs()` for safe directional bouncing
- Lives and score as simple variables
- Game over and restart logic
- `sfx(n)` to play sounds at key moments
- Short print messages for UI

**Next:** [[06-platformer|06 - Platformer Physics]] — gravity, jumping, and proper tile-based physics.
