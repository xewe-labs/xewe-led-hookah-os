# HookahOS — LED light effects for a hookah flask

Personal project (Moscow) · 2018-11 → 2019-02 · Solo: Max Dokukin · Status: Completed (uploaded to GitHub 2023-11-09)

Software for LED lights that illuminate hookah's flask.
Based on arduino nano. Has various modes that can be switched with a button.

<img width="600" alt="Screenshot 2023-11-18 at 20 51 18" src="https://github.com/xeweva/HookahOS/assets/54597813/241eaf2e-f750-433a-a711-2d35541125f7">

## Overview

HookahOS is the Arduino sketch behind a lighting base for a hookah: 38 addressable LEDs set into a one-inch-thick log slice
light the glass flask from below. The LEDs are wired as one serial strip but laid out as a disc, so the sketch carries
hand-built lookup tables that translate the strip order into rings, rows, columns, a neighbour grid and falling-snow lanes.
On top of those tables it implements nine effects — fill, fade, mix, random, gazer, scan, radar, snow and rainbow — each
called with a total duration in milliseconds. The uploaded version (`HOOKAH 0S 1.0`) plays a fixed demo playlist of all
effects in a loop; the button switching mentioned above is not part of this version of the code.

## Highlights

- 38 addressable LEDs on data pin D12, driven with Adafruit NeoPixel (`NEO_GRB + NEO_KHZ800`) — `setup.h`
- Nine effects with a documented call signature and recommended parameters — header comment in `HookahOS.ino`, code in `modes.h`
- Strip-to-geometry lookup tables: a 38-entry inward spiral (18-LED outer ring, 14-LED middle ring, 6 centre LEDs), 9 scan rows
  and 11 scan columns, an 11 × 12 neighbour grid and 9 snow lanes of 6 steps — `genArr()` in `auxiliary_functions.h`
- The LED maps were drawn first, on 2018-11-26 and 2018-11-27 — [`hookah matrix/`](hookah%20matrix/)
- Duration-based API: every effect divides its total time by its own step count (fill 41.1, fade 526, radar 18, scan 38 …) — `modes.h`

## How it works

```
power (2 × 18650 Li-ion + battery management) → Arduino Nano → D12 → 38-LED strip under the flask

setup(): genArr() builds the lookup tables → strip.begin()
loop():  demo_mode() → fill ×3 → fade ×4 → random (20 s) → scan ×2 → gazer (20 s) → radar (20 s) → snow (20 s) → rainbow ×2 → mix ×6
each effect: hex colour → R/G/B (colCon) → lookup table → strip.setPixelColor() → strip.show() → delay(total / steps)
```

- **`HookahOS.ino`** — entry point; its header comment is the effect API with recommended parameters; `loop()` runs `demo_mode()`.
- **`setup.h`** — NeoPixel object (38 LEDs, pin 12), brightness constants, pattern arrays, `setup()`.
- **`auxiliary_functions.h`** — `colCon` (hex → channel), `lper` (wrap-around index), `brightnessFade`, row/column scan helpers and `genArr()`, which fills every lookup table.
- **`modes.h`** — the nine effects and the demo playlist:

| Effect | Call (recommended values) | What it does |
|---|---|---|
| fill | `fill_mode(color, 4000, 'C'/'L', 'I'/'O')` | a four-pixel comet (full, 1/5, 1/12, 1/50 brightness) runs along the strip (`L`) or spirals in/out through the rings (`C`) |
| fade | `fade_mode(color, 10000)` | all LEDs in one colour, brightness ramps up and down |
| mix | `mix_mode(120000, 10, 6, 'R/G/B', 'R/G/B')` | two colour channels sweep against each other around the spiral; green scaled down by the "green reduce" factor |
| random | `random_mode(color, 4000)` | one random LED pops in and fades out |
| gazer | `gazer_mode(color, 7000)` | a random LED and its four grid neighbours flash as a cross |
| scan | `scan_mode(color, 3000, 'T'/'F')` | a line sweeps top→bottom→top, then left→right→left; `F` adds dimmed neighbour lines |
| radar | `radar_mode(circle color, background color, 2500)` | over a background colour, a white head circles the 18-LED outer ring and repaints it in the circle colour behind itself |
| snow | `snow_mode(flakes color, background color, 400)` | flakes spawn in a random lane of 9 and fall 6 steps |
| rainbow | `rainbow_mode(10000, 4)` | colour wheel rotated around the spiral |

- **`hookah matrix/`** — the two LED maps: the physical disc (rows of 2-4-5-5-6-5-5-4-2 LEDs) and the numbered index grid that became `ledGrid`.
- **`static/media/`** — photos and a short video of the finished base (used by the project page).

## Results

| Measure | Value | Note |
|---|---|---|
| LEDs | 38 | `NUM_LEDS`, matches the 2018-11-26 map |
| Effects | 9 | plus `demo_mode()` playlist of 21 calls |
| Lookup tables | 378 bytes of `char` arrays | `circlePat` 38, `scanPat` 2 × 11 × 7, `ledGrid` 11 × 12, `snowPat` 9 × 6 |
| Code | 930 lines in 4 files | `wc -l` |
| Libraries | Adafruit NeoPixel only | |

The project is a hardware build, so its results are what the device does rather than benchmarks.

## Getting started

Requirements: Arduino IDE (or `arduino-cli`), an Arduino Nano, a 38-LED NeoPixel-compatible strip on pin D12, and the
**Adafruit NeoPixel** library.

```bash
# The Arduino IDE expects the folder to share the sketch's name, so copy it into a folder called HookahOS
mkdir HookahOS && cp HookahOS.ino *.h HookahOS/

arduino-cli lib install "Adafruit NeoPixel"
arduino-cli compile --fqbn arduino:avr:nano HookahOS
arduino-cli upload  --fqbn arduino:avr:nano -p <serial port> HookahOS
```

To run a single effect instead of the playlist, replace `demo_mode();` in `loop()` with one of the calls from the table above.
For a different LED count or layout, change `NUM_LEDS` and the tables in `genArr()` together.

## Documents

- [LED disc map, 2018-11-26](hookah%20matrix/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202018-11-26%20%D0%B2%2012.10.50.png)
- [LED index grid, 2018-11-27](hookah%20matrix/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202018-11-27%20%D0%B2%2016.11.43.png)
- Photos and video: [`static/media/resources/`](static/media/resources/)
- Project page: [maxdokukin.com/projects/xewe-led-hookah-os](https://maxdokukin.com/projects/xewe-led-hookah-os)
- Related: XeWe LED OS, a later addressable-LED firmware — [project page](https://maxdokukin.com/projects/xewe-led-os) ·
  [github.com/xewe-labs/xewe-led-os](https://github.com/xewe-labs/xewe-led-os)
