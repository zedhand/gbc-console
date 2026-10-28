# Architecture

Anything marked **TBD** is unverified and gets settled by bench work (mostly phase 1).

## Signal flow

```
                 ┌──────────────── donor GBC motherboard (intact) ────────────────┐
  crystal ✂ ───► │ CGB CPU ──LCD ribbon (15-bit RGB + dot clock + syncs)──┐        │
  (phase 4)      │    ▲                                                   │        │
                 │    └── button pads ◄── analog switches ◄──┐           │        │
                 │ audio (pre-amp line level) ──┐             │           │        │
                 └──────────────────────────────┼─────────────┼───────────┼────────┘
                                                │             │           │
                                  3.5 mm out ◄──┤             │           ▼
                                  I2S ADC ◄─────┘ (phase 8)   │   ┌──────────────┐  HSTX
                                       │                      │   │   RP2350B    │──────► HDMI connector (DVI)
                                       └─────────────────────►│   │   (video)    │
                                                              │   └──────────────┘
                                                              │      ▲  │ CPU clock out ──► CGB clock input
                                                              │ UART │  │
                                                       ┌──────┴──────┴┐ │
             USB-A (host) ───────────────────────────► │  Pico 2 W    │
             Bluetooth pads ~~~~~~~~~~~~~~~~~~~~~~~~~► │  (input)     │
                                                       └──────────────┘
             USB-C (power in) ──► 5 V rail ──► ~3 V regulator ──► GBC battery input
```

## Why two chips

1. **Pin budget.** The video side alone needs ~30 GPIO (see PINOUT.md). A Pico 2 exposes 26. The RP2350B (QFN-80) has 48.
2. **Determinism.** Capture and output are cycle-timed. Bluetooth (BTstack) and USB host stacks are interrupt-heavy. Keeping them on separate silicon removes a whole class of glitches.
3. **Decoupled bring-up.** Video and input can be developed and tested independently.

The only link between them is one UART line (input → video) for menu commands.

## Video

### Source
- GBC native: **160×144**, one frame = 154 lines × 456 dots at 4.194304 MHz → **59.7275 Hz**.
- The CGB CPU drives the LCD over the ribbon with **15-bit parallel RGB (5:5:5)** plus a dot clock and line/frame sync signals. Exact pin mapping, logic level and sync polarity: **TBD (phase 1)**.
- Frame buffer: 160 × 144 × 2 B (RGB555 packed in 16 bits) = **46,080 B**. Fits easily in RP2350 SRAM; double-buffer is affordable (~92 KB) but the goal is to not need it.

### Capture
- PIO state machine samples the 15 data lines on the dot-clock edge, uses the syncs to track line/frame position, DMA into the frame buffer.
- Data pins must be **consecutive GPIO** for PIO `in pins` — pin assignment matters.
- Level shifting only if phase 1 finds the LCD bus is not 3.3 V.

### Output (HSTX → DVI)
HSTX is the RP2350's high-speed serial output with a built-in TMDS encoder, on fixed pins GPIO 12–19 (3 TMDS pairs + clock pair).

| Mode | Scale | Image | Per-lane rate | Status |
|---|---|---|---|---|
| 640×480 @ 60 | 3× | 480×432, bordered | ~252 Mbps | **Baseline.** Comfortably in spec. |
| 720×480 @ 59.94/60 (CEA 480p) | 3× | 480×432, bordered | 270 Mbps | Baseline alternative; TV-friendlier CEA timing. |
| 1280×720 @ 60 | 5× | 800×720, pillarboxed | 742.5 Mbps | **Stretch.** Needs a heavy overclock; Pico-class chips have reached 720p and even 1080p only well outside spec. Unproven at 60 Hz here. |

Trade-off: 480p means the TV does the final upscale, which can add lag on some sets. 720p 5× is the ideal, but it's a research item, not a dependency.

### Frame sync and latency
- Free-running (phase 2): output at 60 Hz, source at 59.7275 Hz → a repeated frame roughly every 3.7 s (visible stutter in scrolling).
- **Clock lock (phase 4):** remove the GBC's 8.388608 MHz crystal and feed the CGB a clock scaled by `f_out / 59.7275`:
  - Output exactly 60.000 Hz → **≈8.4269 MHz**
  - Output 59.94 Hz (CEA) → **≈8.4185 MHz**
  - Derive it from the same PLL/timebase as the video output so the two can't drift.
- With source and output locked, read the frame buffer a few lines behind the write pointer (**beam racing**) → sub-frame latency.
- Clock source options, in order:
  1. RP2350 GPOUT / PIO-generated clock — check jitter vs. what the CGB oscillator input tolerates (**TBD**).
  2. External programmable clock (Si5351) driven over I2C by the RP2350B, if (1) is too jittery.

### Color
GBC games were tuned for a dim, washed-out LCD. Raw RGB555 on a modern display looks oversaturated. The OSD offers: raw, and one or more correction curves (lookup table applied at output, 32 K entries or per-channel LUT).

## Input

### Controllers (Pico 2 W)
- **Bluetooth:** Bluepad32 on BTstack via the CYW43 radio. Classic + BLE → Xbox, PlayStation, Switch Pro, 8BitDo and others. Pair button on the case; link keys persist in flash.
- **USB:** native USB in host mode, TinyUSB HID host. XInput (wired Xbox pads) needs a community host driver. The USB-A port's 5 V VBUS is supplied by our board through a load switch.

### Driving the GBC
- One analog switch across each of the 8 button pads (A, B, Select, Start, Up, Down, Left, Right). Closing the switch is electrically identical to pressing the button, so there is **no matrix-timing emulation** — the CGB scans its matrix exactly as normal.
- Switch supply = GBC logic rail (~3.3 V).
- **SOCD:** block opposite d-pad directions (L+R, U+D) in firmware — physically impossible on a real d-pad and some games misbehave.

### Menu
- Hotkey combo (e.g. Select + Start + L shoulder / Home) toggles the OSD.
- While the OSD is open, the input chip stops pressing GBC buttons and sends navigation over UART instead.
- UART protocol: **TBD**; small framed messages (button state, menu nav, pairing status).

## Audio
- **Phase 1–7:** tap the line-level signal before the headphone amp → 3.5 mm jack.
- **Phase 8:** I2S ADC → RP2350B → audio data islands in the TMDS stream. That makes the output HDMI rather than plain DVI and is notoriously fiddly on Pico-class video stacks. Treat as optional.

## Power
- USB-C in (5 V) → regulator to ~3 V → GBC battery terminals. The GBC's own DC-DC still generates its internal rails, so no board rework.
- 5 V also feeds the USB-A host port (via load switch) and the two MCUs' supplies.
- Current budget: **TBD** after measuring a donor.

## Mechanical
- The donor board stays whole; the case is designed around it.
- Keep the **link port** reachable.
- Keep the **cartridge slot** top-loading and accessible.
- Power switch: either keep the GBC's slide switch or bypass it and switch the 3 V feed from the case.

## Open questions
1. LCD ribbon pinout, logic level, sync polarity (phase 1).
2. Does the CGB accept a non-crystal clock input cleanly, and how much jitter can it tolerate?
3. Can the RP2350B do 720p60 reliably at a sane overclock with a 160×144 source (narrow work per line)?
4. Analog audio tap point and level.
5. Board power draw.
