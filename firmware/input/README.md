# firmware/input — Pico 2 W

Read controllers and press the GBC's buttons through analog switches.

Planned modules:
- `usbhost` — TinyUSB HID host (+ XInput driver)
- `bt` — Bluepad32 / BTstack, pairing, link-key storage
- `mapper` — controller → 8 GBC buttons, SOCD cleaning, hotkey detection
- `switches` — analog switch outputs
- `link` — UART TX to the video chip (menu nav, status)

Toolchain: pico-sdk (CMake).
