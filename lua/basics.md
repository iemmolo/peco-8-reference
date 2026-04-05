# Lua Basics for PICO-8

PICO-8 uses a slightly modified version of Lua. If you know any programming, this will be quick.

---

## Variables

```lua
x = 64          -- number
name = "hero"   -- string
alive = true    -- boolean
nothing = nil   -- nothing / empty
```

No type declarations. Variables are global by default. Use `local` to keep them scoped to a function.

```lua
local temp = 5  -- only exists inside current block
```

---

## Numbers

Fixed-point range: **-32768 to 32767.99**. No floats — just know there's a precision limit.

```lua
x = 10
x = 0xff        -- hex
x = 3.14
```

Integer division shorthand:
```lua
x = 7 \ 2      -- same as flr(7/2) = 3
```

---

## Conditionals

```lua
if x > 0 then
  print("positive")
elseif x < 0 then
  print("negative")
else
  print("zero")
end

-- shorthand (single line only)
if (x > 0) print("positive")
if (x > 0) x = 0  y = 1   -- multiple statements ok
```

---

## Loops

```lua
-- for loop
for i = 1, 10 do
  print(i)
end

-- with step
for i = 10, 1, -1 do
  print(i)
end

-- while loop
while x > 0 do
  x -= 1
end

-- repeat until
repeat
  x += 1
until x >= 10
```

---

## Functions

```lua
function greet(name)
  print("hello " .. name)
end

greet("world")

-- return a value
function area(w, h)
  return w * h
end

local a = area(8, 5)  -- a = 40

-- multiple return values
function get_pos()
  return px, py
end

local x, y = get_pos()
```

---

## Tables (the big one)

Tables are PICO-8's only data structure. They work as arrays AND dictionaries.

```lua
-- empty table
t = {}

-- array style (1-indexed!)
items = {"sword", "cloak", "boots"}
print(items[1])   -- "sword"
print(#items)     -- 3 (length)

-- dictionary style
player = {}
player.x = 64
player.y = 64
player.hp = 3
-- same as: player["x"] = 64

-- mixed
data = {10, 20, name="hero", 30}
```

### Looping tables

```lua
-- array loop (sequential keys)
for i = 1, #items do
  print(items[i])
end

-- or with all()
for item in all(items) do
  print(item)
end

-- dictionary loop
for key, val in pairs(player) do
  print(key .. "=" .. tostr(val))
end

-- foreach shorthand
foreach(items, print)
```

### Table of tables (common pattern)

```lua
enemies = {}

function make_enemy(x, y)
  local e = {}
  e.x = x
  e.y = y
  e.hp = 3
  add(enemies, e)
end

function update_enemy(e)
  e.x += 1
  if e.x > 128 then del(enemies, e) end
end

-- in _update():
foreach(enemies, update_enemy)
```

---

## Operators

```lua
-- math
+ - * / % ^     -- standard
\               -- integer division (flr(a/b))

-- comparison
== ~= < > <= >=
!=              -- also works for ~=

-- logical
and  or  not

-- string
..              -- concatenation: "hi" .. "!" = "hi!"
#               -- length: #"hello" = 5

-- shorthand assignment
+= -= *= /= %=

-- bitwise
&  |  ^^  ~  <<  >>
```

---

## Strings

```lua
s = "hello"
s = 'hello'     -- single quotes work too

-- concatenation
msg = "score: " .. tostr(score)

-- length
print(#s)       -- 5

-- substring
sub("hello", 2, 4)  -- "ell"

-- split
parts = split("a,b,c", ",")  -- {"a","b","c"}
```

---

## The Game Loop

```lua
function _init()
  -- runs once when cart starts
  -- set up your game state here
  px = 64
  py = 64
end

function _update()
  -- runs 30 times per second
  -- game logic goes here
  if btn(0) then px -= 1 end
end

function _draw()
  -- runs once per visible frame
  -- drawing goes here
  cls()
  spr(1, px, py)
end
```

Use `_update60()` instead of `_update()` for 60fps (uses more CPU).

---

## Comments

```lua
-- single line comment

--[[
  multi
  line comment
]]
```

---

## Common Patterns

```lua
-- toggle a boolean
flag = not flag

-- clamp a value
x = mid(0, x, 127)

-- ternary-style (no ternary operator in Lua)
val = condition and a or b

-- nil check
if obj ~= nil then
if obj then           -- nil is falsy, so this works too

-- string shorthand print
?"hello"              -- same as print("hello")
```

---

## Gotchas

- Tables are **1-indexed** (first element is `[1]`, not `[0]`)
- `sin()` and `cos()` use **0..1** range, not radians
- `sin()` is **y-inverted** (positive goes down on screen)
- No `ceil()` — use `-flr(-x)` instead
- `#t` only reliable for sequential integer-keyed tables
- Division by zero returns max number, not error
- `sgn(0)` returns `1`, not `0`
- Globals persist across functions — be careful with naming
