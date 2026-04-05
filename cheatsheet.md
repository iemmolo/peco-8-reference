# PICO-8 Cheatsheet

## Game Loop

```lua
function _init()  end   -- runs once at start
function _update() end  -- runs every frame (30fps)
function _draw()  end   -- runs every visible frame
-- use _update60() instead for 60fps
```

## Graphics

```lua
cls([col])                              -- clear screen
pset(x,y,[col])                         -- draw pixel
line(x0,y0,x1,y1,[col])                -- draw line
rect(x0,y0,x1,y1,[col])               -- rectangle outline
rectfill(x0,y0,x1,y1,[col])           -- filled rectangle
circ(x,y,r,[col])                      -- circle outline
circfill(x,y,r,[col])                  -- filled circle
spr(n,x,y,[w,h],[flip_x],[flip_y])    -- draw sprite n
print(str,[x,y,[col]])                 -- print text
color(col)                             -- set draw color
camera([x,y])                          -- set camera offset
pal(c0,c1,[p])                         -- swap colors
palt(col,t)                            -- set transparency
```

## Input

```lua
btn(b,[p])   -- held: 0=L 1=R 2=U 3=D 4=O 5=X
btnp(b,[p])  -- just pressed this frame
```

## Math

```lua
flr(x)        -- floor (round down)
ceil(x)       -- ceiling (round up)
abs(x)        -- absolute value
max(x,y)      -- maximum
min(x,y)      -- minimum
mid(x,y,z)   -- middle of three values
rnd(x)        -- random 0 <= n < x
sqrt(x)       -- square root
sin(x)        -- sine (0..1 range, y-inverted)
cos(x)        -- cosine (0..1 range)
atan2(dx,dy)  -- angle as 0..1
sgn(x)        -- -1 or 1
```

## Map

```lua
map(tx,ty,sx,sy,tw,th,[layer])  -- draw map section
mget(x,y)                        -- get tile at x,y
mset(x,y,v)                      -- set tile at x,y
fget(n,[f])                      -- get sprite flags
fset(n,[f],v)                    -- set sprite flags
```

## Audio

```lua
sfx(n,[ch],[offset])      -- play sfx (n=-1 stop)
music(n,[fade],[mask])    -- play music pattern
```

## Tables

```lua
t = {}
add(t, v)          -- append value
del(t, v)          -- delete first match
count(t)           -- length
for v in all(t) do -- iterate
foreach(t, f)      -- call f for each element
```

## Strings

```lua
#str               -- length
str1..str2         -- concat
sub(str,from,[to]) -- substring
tostr(val)         -- to string
tonum(str)         -- to number
split("a,b,c",",") -- split to table
```

## Operators

```lua
+ - * / % ^   -- math
\             -- integer division (flr(a/b))
== ~= < > <= >=
and or not
& | ^^ ~ << >> -- bitwise
.. #           -- string concat / length
+= -= *= /=    -- shorthand assignment
```

## System

```lua
time()           -- seconds since start
stat(0)          -- memory used (KB)
stat(1)          -- cpu used (0..1)
stat(7)          -- current fps
cartdata(id)     -- open save slot
dget(i)          -- read save value
dset(i,v)        -- write save value
```

## Colors

```
0=black  1=dk.blue  2=dk.purple  3=dk.green
4=brown  5=dk.gray  6=lt.gray    7=white
8=red    9=orange   10=yellow    11=green
12=blue  13=indigo  14=pink      15=peach
```

## Keyboard Shortcuts

```
CTRL+R    run / reload
CTRL+S    save
CTRL+Z    undo
CTRL+Y    redo
CTRL+F    find
CTRL+B    toggle comment
CTRL+D    duplicate line
ALT+←→   switch editors
ESC       console / editor toggle
```

## Useful One-Liners

```lua
-- clamp x between min and max
x = mid(mn, x, mx)

-- lerp from a to b by t
v = a + (b-a)*t

-- angle from point to point (0..1)
a = atan2(tx-sx, ty-sy)

-- move toward angle
dx = cos(a)*spd
dy = -sin(a)*spd

-- screen wrap
x = (x+128)%128

-- distance
d = sqrt((x2-x1)^2+(y2-y1)^2)

-- check sprite flag 0
if fget(tile,0) then -- solid

-- random from table
item = rnd(my_table)
```
