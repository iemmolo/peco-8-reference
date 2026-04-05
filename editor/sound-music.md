# Sound & Music Editor

## Sound Effects (SFX)

- 64 SFX slots (0–63)
- Each SFX has **32 notes**
- Each note has: **frequency, instrument, volume (0–7), effect (0–7)**
- **Speed** = ticks per note (lower = faster)

### Two Modes

| Mode | Use |
|------|-----|
| Pitch mode | Simple left-to-right sound effects |
| Tracker mode | Music composition with note names |

Click the mode button in the top-left of the Sound Editor to switch.

---

## Instruments

| # | Sound |
|---|-------|
| 0 | Triangle wave |
| 1 | Tilted saw |
| 2 | Sawtooth |
| 3 | Square |
| 4 | Pulse |
| 5 | Organ |
| 6 | Noise |
| 7 | Phaser |

Instrument 6 (noise) + effect 3 (drop) = great drum kick.
Instrument 7 (phaser) = engine / rumble sound for thrust.

---

## Effects

| # | Effect |
|---|--------|
| 0 | None |
| 1 | Slide (pitch glide from previous note) |
| 2 | Vibrato |
| 3 | Drop (falls in pitch — good for drums) |
| 4 | Fade in |
| 5 | Fade out |
| 6 | Fast arpeggio (4 notes) |
| 7 | Slow arpeggio (4 notes) |

---

## Shortcuts

```
SPACE           play / stop
- / +           previous / next SFX
< / >           slower / faster speed
SHIFT+LMB       change all notes in column at once
SHIFT+SPACE     play current 8-note group
BACKSPACE       delete note
A               release loop
CTRL+C/V        copy / paste SFX
```

---

## Playing SFX in Code

```lua
sfx(n)                  -- play SFX n on any free channel
sfx(n, channel)         -- play on specific channel (0–3)
sfx(n, channel, offset) -- start from note offset (0–31)
sfx(-1)                 -- stop all SFX
sfx(-1, channel)        -- stop specific channel
sfx(-2, channel)        -- release loop on channel
```

---

## Music Editor

- 64 patterns (0–63)
- Each pattern = **4 channels**, each playing one SFX
- Patterns chain together for full music tracks

### Playback Markers

| Marker | Meaning |
|--------|---------|
| → (right arrow on) | Start point — music begins here |
| ← (left arrow on) | Loop point — playback loops back to nearest start |
| ■ (stop on) | Stop point — playback stops here |

### Music Flow

```
Pattern 0 → Pattern 1 → Pattern 2 [loop] → Pattern 1 → ...
```

Set the start marker on pattern 0, loop marker on pattern 1, stop marker on final pattern.

---

## Playing Music in Code

```lua
music(n)                     -- play pattern n
music(n, fade_ms)            -- fade in over fade_ms milliseconds
music(n, fade_ms, channel_mask) -- reserve specific channels
music(-1)                    -- stop music
music(-1, fade_ms)           -- fade out over fade_ms milliseconds
```

**Channel mask** (to reserve channels for SFX):
```lua
music(0, 0, 0b0111)  -- reserve channels 0,1,2 for music; ch3 free for SFX
-- bits: ch0=1, ch1=2, ch2=4, ch3=8
```

---

## Filters (Advanced)

Toggle these in tracker mode to shape sound further:

| Filter | Effect |
|--------|--------|
| NOIZ | White noise (instrument 6 only) |
| BUZZ | Buzzy waveform |
| DETUNE-1 | Flange / chorus effect |
| DETUNE-2 | Octave stacking |
| REVERB | Echo (2 or 4 tick delay) |
| DAMPEN | Low-pass filter (softer sound) |

---

## Quick SFX Design Tips

- **Jump:** short, rising pitch, instrument 0 (triangle), speed 4–6
- **Land/thud:** short drop effect (effect 3), instrument 6 (noise)
- **Coin/pickup:** high rising notes, instrument 0 or 1, speed 6–8
- **Hurt:** descending noise burst, instrument 6, effect 5 (fade out)
- **Game over:** slow descending melody, instrument 2 or 3
- **Shoot/laser:** sawtooth (inst 2), effect 5 (fade out), high pitch dropping fast
- **Explosion:** noise (inst 6), effect 3 (drop), multiple notes, medium speed

---

## Tracker Mode Keyboard (Piano Layout)

In tracker mode, your keyboard acts as a piano:

```
Black keys: Q W   T Y U   O P
White keys:  A S D   G H J   L
             C D E   G A B   D
             3 3 4   4 4 5   5 (octave)
```

`Z` and `X` shift octave down/up.

---

## Checking What's Playing

```lua
stat(16)  -- SFX playing on channel 0 (-1 if none)
stat(17)  -- SFX playing on channel 1
stat(18)  -- SFX playing on channel 2
stat(19)  -- SFX playing on channel 3
stat(57)  -- music playing (true/false)
stat(54)  -- current pattern index
```
