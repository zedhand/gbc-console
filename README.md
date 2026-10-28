# gbc-console

A Game Boy Color consolizer: an unmodified-as-possible GBC motherboard from a donor unit (dead screen, burnt polarizer, cracked shell — doesn't matter) running as a home console with HDMI-compatible video out and USB / Bluetooth controllers.

**Status:** planning. Nothing built yet. Next step is phase 1 (scoping the LCD ribbon on a donor).

## Goals

- **Keep the donor board whole.** No cutting the motherboard in half; the case is designed around it.
- **Low latency.** Lock the GBC's clock to the video output and beam-race instead of buffering a full frame.
- **Clean picture.** Integer scaling only, optional GBC color correction.
- **Modern controllers.** USB (wired pads, 8BitDo receivers) and Bluetooth (Classic + BLE pads).
- **Open.** Firmware, hardware and case are all published here.

## Architecture at a glance

Two chips, so the video timing loop never shares a core with a radio stack:

| Chip | Job |
|---|---|
| **RP2350B** (video) | Capture the GBC's LCD bus → frame buffer → HSTX DVI out; generate the GBC CPU clock; on-screen menu; later, HDMI audio |
| **Pico 2 W** (input) | Bluepad32 (Bluetooth) + TinyUSB host (USB) → drives analog switches across the 8 GBC button pads; sends menu commands to the video chip over UART |

Full detail: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md). Pin budget: [docs/PINOUT.md](docs/PINOUT.md). Parts: [docs/BOM.md](docs/BOM.md). What already exists: [docs/PRIOR_ART.md](docs/PRIOR_ART.md).

## Phases

- [ ] **1. Scope the LCD ribbon** on a donor: pinout, logic levels, dot clock, sync timing/polarity. Fills in `docs/PINOUT.md`.
- [ ] **2. Video bring-up:** RP2350B capture → HSTX DVI at 480p, free-running, GBC on its own crystal.
- [ ] **3. Input bring-up:** Pico 2 W USB host first, then Bluepad32, then the analog-switch array on a donor.
- [ ] **4. Clock lock:** remove the GBC crystal, clock the CGB from the RP2350B, beam-race the output.
- [ ] **5. OSD over UART:** scaling, color correction, pairing status.
- [ ] **6. Custom PCB:** interposer on the LCD connector, RP2350B, Pico 2 W, power, analog audio out.
- [ ] **7. Case** (printed / CNC).
- [ ] **8. HDMI audio.**
- [ ] *Stretch:* 720p output (see the video-mode note in ARCHITECTURE.md).

## Repo layout

```
docs/              architecture, pinout, BOM, prior art
firmware/video/    RP2350B firmware
firmware/input/    Pico 2 W firmware
hardware/          KiCad projects
case/              enclosure CAD
```

## Licenses

- Firmware and documentation: [MIT](LICENSE)
- Hardware (`hardware/`) and case (`case/`): [CERN-OHL-S-2.0](hardware/LICENSE) — modified hardware designs that are distributed must stay open.
