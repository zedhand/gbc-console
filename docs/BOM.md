# Bill of materials (candidates)

Candidate parts, not a final BOM. Quantities are per console.

## Prototyping

| Part | Qty | Purpose | Notes |
|---|---|---|---|
| Donor GBC (dead/burnt screen) | 1+ | The console | Have several. Keep one as an untouched reference. |
| RP2350B dev board with all 48 GPIO broken out | 1 | Video chip | e.g. Pimoroni Pico Plus 2. Check the specific board breaks out GPIO 12–19 cleanly. |
| HSTX → DVI/HDMI breakout | 1 | Video out during bring-up | e.g. Adafruit 22-pin FPC HSTX-to-DVI adapter, or a DIY connector + series resistors. |
| Raspberry Pi Pico 2 W | 1 | Input chip | |
| 8-ch logic analyzer | 1 | Phase 1 | Cheap Saleae-clone + PulseView is fine for these clock rates. |
| GBC LCD ribbon breakout | 1 | Phase 1 tap | Make one (milled or ordered PCB) matching the ribbon pitch. |
| USB-C power breakout | 1 | Power | |

## Production board

| Part | Qty | Purpose | Notes |
|---|---|---|---|
| RP2350B (QFN-80) + flash + 12 MHz crystal | 1 | Video | Standard RP2350 reference circuit. |
| Pico 2 W module (castellated) | 1 | Input | Avoids an RF layout. Solder down via castellations. |
| 74LVC1G66 (×8) or 74LVC4066 (×2) | 8 / 2 | Button switches | Low on-resistance, 3.3 V supply. |
| HDMI type-A receptacle | 1 | Video out | Plus ESD protection on TMDS lines. |
| USB-A receptacle + load switch (e.g. TPS2051-class) | 1 | Controller host port | Current-limited 5 V VBUS with fault flag. |
| USB-C receptacle + 5.1 kΩ CC pulldowns | 1 | Power in | |
| 3 V regulator | 1 | Feeds GBC battery input | Size after measuring donor draw. |
| 3.3 V regulator | 1 | MCUs | |
| Si5351A | 0–1 | Fallback CGB clock | Only if the RP2350 clock is too jittery. |
| I2S ADC (e.g. PCM1808-class) | 0–1 | HDMI audio (phase 8) | |
| 3.5 mm jack | 1 | Analog audio out | |
| Tactile switches | 2 | Pair, reset | |

## Mechanical
- Case: printed (prototype) → milled or resin (final).
- Heat-set inserts / screws to mount the donor board on its original screw holes.
