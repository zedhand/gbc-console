# firmware/video — RP2350B

Capture the GBC LCD bus, hold the frame buffer, output DVI over HSTX, generate the CGB clock, draw the OSD.

Planned modules:
- `capture` — PIO + DMA sampling of the 15-bit RGB bus
- `framebuffer` — 160×144 RGB555, beam-race read pointer
- `dvi` — HSTX TMDS output, 480p baseline (720p stretch)
- `cgbclock` — clock generation locked to the output timebase
- `osd` — menu overlay, color-correction LUTs
- `link` — UART RX from the input chip

Toolchain: pico-sdk (CMake). No code until phase 1 fixes the capture pinout.
