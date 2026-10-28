# Pinout and pin budget

Assignments are provisional until phase 1 confirms the LCD ribbon signals.

## GBC LCD ribbon — TBD (phase 1)

Fill this in from a logic-analyzer capture on a donor. Record for each pin: signal, direction, idle level, logic voltage, and what it does during a frame.

| Ribbon pin | Signal | Notes |
|---|---|---|
| | R0–R4 | |
| | G0–G4 | |
| | B0–B4 | |
| | Dot clock | active edge? |
| | Line sync | polarity, width |
| | Frame sync | polarity, width |
| | Other LCD control | |
| | GND / VCC | voltage |

Method: an 8-channel analyzer is enough, done in passes — first find clock and syncs (frequency + timing tells you which is which), then sweep the data lines in groups of ~6 while triggering on frame sync.

## RP2350B (video) — budget

| Function | GPIO count | Constraint |
|---|---|---|
| LCD data (R, G, B) | 15 | Consecutive pins (PIO `in`) |
| Dot clock + line sync + frame sync | ~3 | Adjacent to data block helps PIO |
| HSTX DVI | 8 | **Fixed: GPIO 12–19** |
| CGB clock out | 1 | GPOUT-capable pin, or PIO |
| UART RX from input chip | 1 | |
| I2C to Si5351 (fallback clock) | 2 | Optional |
| I2S ADC (phase 8) | 3 | |
| Debug UART TX/RX | 2 | |
| Status LED | 1 | |
| **Total** | **~36 of 48** | |

Because HSTX occupies 12–19, put the 18-pin capture block in the upper bank (e.g. GPIO 20–37) so it stays contiguous.

## Pico 2 W (input) — budget

| Function | GPIO count |
|---|---|
| Analog switch controls (8 buttons) | 8 |
| UART TX to video chip | 1 |
| USB-A VBUS load-switch enable | 1 |
| USB-A VBUS fault input | 1 |
| Pair button | 1 |
| Status LED (external) | 1 |
| **Total** | **13 of 26** |

USB host uses the native USB pins (not GPIO). The CYW43 radio uses internal pins that are not broken out.
