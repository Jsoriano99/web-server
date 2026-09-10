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
  - [x] Install ESP-IDF v6.1 (`source ~/.espressif/tools/activate_idf_v6.1.sh`)
  - [x] USB passthrough (`usbipd-win`) + serial permissions
  - [x] Project skeleton; build, flash, monitor
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

### 2026-09-10 — ESP-IDF v6.1 installed (Phase 0, step 2)

- Installed via EIM (`eim-cli` → `eim install`); IDF lives at `~/.espressif/v6.1/esp-idf`.
- Per-session activation: `source ~/.espressif/tools/activate_idf_v6.1.sh`.
- Verified: `idf.py --version` → `ESP-IDF v6.1`.

### 2026-09-10 — USB passthrough working (Phase 0, step 3)

- Board detected by Windows: CH340 (1a86:7523), usbipd BUSID `3-1`; attached to WSL as `/dev/ttyUSB0`.
- Gotcha: WSL caches the Windows PATH — after installing usbipd, `usbipd.exe` is "command not found" until `export PATH="$PATH:/mnt/c/Program Files/usbipd-win"` (or a WSL restart).
- Gotcha: `/dev/ttyUSB0` is `root:dialout`; the user was added to `dialout` (applies fully after a WSL restart) — meanwhile `newgrp dialout` grants access per shell (verified working).
- `usbipd attach` must be repeated after replugging the board or rebooting Windows.

### 2026-09-10 — Skeleton + first build (Phase 0, step 4a)

- Created by hand: `CMakeLists.txt`, `main/CMakeLists.txt`, `main/main.c`.
- Gotchas: the root file was saved as `CMakeLists.text` (extensions matter); CMake comments use `#`, not `//`; typo `cmake_minimum_version` → `cmake_minimum_required`.
- `idf.py set-target esp32` + `idf.py build` → `Project build complete.`
- Artifacts: `bootloader/bootloader.bin`, `partition_table/partition-table.bin`, `web-server.bin`.

### 2026-09-10 — Flash + monitor working (Phase 0, step 4 complete)

- `idf.py flash` wrote all three binaries; hashes verified; board hard-reset into the new firmware.
- Serial monitor shows the boot chain (ROM → 2nd stage bootloader → app) and `Hello from the web-server project main.c file!`
- App version in the boot log = the git commit hash (`6552152`) — IDF stamps the build with the repo state.
- **Finding (verification):** the board has 4 MB flash but the image declares 2 MB — `W spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k)`. Fix pending: menuconfig → Serial flasher config → Flash size → 4 MB.
