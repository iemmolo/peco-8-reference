# Audio API

## Play SFX

```lua
sfx(n)                          -- play SFX n on any free channel
sfx(n, channel)                 -- play on specific channel (0–3)
sfx(n, channel, offset)         -- start from note offset (0–31)
sfx(n, channel, offset, length) -- play only `length` notes

sfx(-1)                         -- stop all SFX on channel 0
sfx(-1, channel)                -- stop SFX on specific channel
sfx(-1, -1)                     -- stop all channels
sfx(-2, channel)                -- release loop on channel
```

**Channels:** 0–3. Use `-1` for auto-select (pick any free channel).

```lua
-- common patterns
sfx(0)          -- play jump sound
sfx(1, 0)       -- play hurt sound on channel 0
sfx(-1, 0)      -- stop channel 0 (e.g. when landing)
```

---

## Play Music

```lua
music(n)                         -- play pattern n
music(n, fade_ms)                -- fade in over milliseconds
music(n, fade_ms, channel_mask)  -- reserve channels for music

music(-1)                        -- stop music
music(-1, fade_ms)               -- fade out
```

**Channel mask:** which channels music uses (other channels free for SFX)
```lua
music(0, 0, 7)     -- music uses ch 0,1,2 (bits 1+2+4=7); ch 3 free
music(0, 0, 0xf)   -- music uses all 4 channels
music(0, 500)      -- start pattern 0, fade in over 500ms
```

---

## Check What's Playing

```lua
stat(16)   -- SFX index playing on channel 0 (-1 if none)
stat(17)   -- channel 1
stat(18)   -- channel 2
stat(19)   -- channel 3

stat(20)   -- note number (0-31) on channel 0
stat(21)   -- channel 1
stat(22)   -- channel 2
stat(23)   -- channel 3

stat(54)   -- currently playing pattern index
stat(55)   -- total patterns played
stat(56)   -- ticks elapsed on current pattern
stat(57)   -- music playing (true/false)
```

---

## SFX Design Reference

### Instruments

| # | Sound |
|---|-------|
| 0 | Triangle — clean, soft |
| 1 | Tilted saw — warm |
| 2 | Sawtooth — buzzy |
| 3 | Square — classic chip |
| 4 | Pulse — hollow |
| 5 | Organ — full |
| 6 | Noise — percussion |
| 7 | Phaser — robotic/engine |

### Effects

| # | Effect |
|---|--------|
| 0 | None |
| 1 | Slide (glide from prev note) |
| 2 | Vibrato |
| 3 | Drop (falling pitch — drums) |
| 4 | Fade in |
| 5 | Fade out |
| 6 | Fast arpeggio |
| 7 | Slow arpeggio |

### Quick SFX Recipes

```
Jump:     inst=0, rising notes, speed 5, short
Land:     inst=6, effect=3 (drop), 1-2 notes
Shoot:    inst=2, effect=5 (fade out), high pitch
Coin:     inst=0 or 1, 2-3 notes rising, fast
Hurt:     inst=6, effect=5, mid pitch
Explosion: inst=6, effect=3, low pitch, slow
Win:      inst=0, rising scale, medium speed
Game over: inst=2, descending notes, slow
Engine:   inst=7, looped, low pitch
```

---

## Music Structure Tips

- Pattern 0–7: main gameplay music
- Pattern 8–15: boss music
- Pattern 16+: stings/jingles

```lua
-- play music on level start
function level_start()
  music(0, 500)   -- fade in over 500ms
end

-- game over jingle, no loop
function game_over_music()
  music(16)
end

-- stop music and play SFX
function player_wins()
  music(-1, 300)  -- fade out over 300ms
  sfx(5)          -- play win SFX
end
```

---

## SFX Playback via P8SCII (Text)

You can trigger SFX inside `print()` strings:

```lua
?"\\a"        -- beep
?"\\a12"      -- play SFX 12
```

---

## Volume

Set from command line or config:
```
config volume 128     -- 0 to 256
```

Or at runtime:
```lua
-- no direct API — use config or command line
```

Mute toggle: `CTRL+M` in PICO-8.
