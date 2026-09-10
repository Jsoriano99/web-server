# Web Server Project — Control & Monitoring

ESP32-WROOM-32D firmware that serves a web page to toggle a physical LED and show the
live state of a physical push button.

Design doc: [`../docs/superpowers/specs/2026-09-10-web-server-design.md`](../docs/superpowers/specs/2026-09-10-web-server-design.md)

## How we work

- The user types all code by hand; the assistant explains, guides, and reviews.
- One phase at a time; each phase closes with a verification-minded check.
- Every finding (glitches, flakiness, timings) gets logged below.

## Phases

- [ ] **Phase 0 — Toolchain + blinky (GPIO output)**
  - [x] Component lab: characterize LEDs, resistors, buttons with the multimeter
  - [ ] Install ESP-IDF v6.1 (record the exact version)
  - [ ] USB passthrough (`usbipd-win`) + serial permissions
  - [ ] Project skeleton; build, flash, monitor
  - [ ] Blinky from a FreeRTOS task
- [ ] **Phase 1 — Digital input: button, pull-up, debounce**
- [ ] **Phase 2 — Wi-Fi station**
- [ ] **Phase 3 — HTTP server**
- [ ] **Phase 4 — Web UI**
- [ ] **Phase 5 — Real-time + concurrency**
- [ ] **Phase 6 — Extras (optional)**

## Log

### 2026-09-10 — Component lab (Phase 0, step 1)

- Buttons: checked with the multimeter in continuity mode.
- LEDs: red LED measured in diode mode → Vf ≈ 1.48 V.
- Resistors: 220 Ω found (red-red-brown-gold bands; measured 215–240 Ω on the 2 kΩ range).
- Blinky combo locked: red LED + 220 Ω → expected current ≈ 8 mA.
- Gotcha: diode mode displays a voltage (mV), not resistance; Ω mode is meaningless on a diode.
