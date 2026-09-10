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
  - [ ] Component lab: characterize LEDs, resistors, buttons with the multimeter
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

<!-- One entry per session: date, what was done, findings, gotchas. -->
