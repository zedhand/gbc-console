# Prior art

## GBHD Color — Gamebox Systems
FPGA-based GBC consolizer: 720p HDMI, digitized audio, SNES controller port.
- **Borrow:** proof that tapping the GBC LCD signals for clean digital video works; 5× integer scale to 720p.
- **Beat:** requires cutting the donor motherboard in half; uses a single frame buffer, so it isn't lag-free; one wired controller type.
- https://retrorgb.com/gbhd-color-consolizer-by-gamebox-systems.html

## GBADVI — Gamebox Systems
Open-source Spartan-6 FPGA consolizer for the GBA (40-pin LCD revision), DVI out, SNES port, USB-C power.
- **Borrow:** open design files, firmware and BOM to read through; overall consolizer board structure.
- **Note:** taps the GBA via a flex cable soldered to the CPU legs. The GBC approach here taps the LCD ribbon instead.
- https://github.com/GameboxSystems/GBADVI

## PicoDVI-DMG — joeostrander
RP2040 + PicoDVI capturing an original Game Boy (DMG) and outputting HDMI.
- **Borrow:** Pico-class LCD capture → DVI pipeline, and the HDMI-audio work.
- **Note:** HDMI audio was the hard part and took years to get working. Our design treats it as a late, optional phase.
- https://github.com/joeostrander/PicoDVI-DMG

## GBA consolizer kit (2018)
FPGA GBA consolizer, 720p over mini-HDMI, SNES port. Notable trick: removed the GBA's crystal to run the console at an HDMI-friendly 60 Hz — the same clock-lock idea used here.
- https://hackaday.com/2018/12/02/

## RP2350 video ceilings
- Pico-class chips have reached 720p (RP2040) and 1080p (RP2350 at a 372 MHz overclock), but only well outside spec. Relevant to the 720p stretch goal.
- https://hackaday.com/2026/09/21/the-rp2350-does-1080p/

## Libraries to use
- **Raspberry Pi pico-sdk** — HSTX, PIO, DMA.
- **Bluepad32** — Bluetooth gamepad support on Pico W / Pico 2 W (Apache-2.0).
- **BTstack** — Bluetooth stack under Bluepad32; free to use on RP2xxx.
- **TinyUSB** — USB host (HID).
