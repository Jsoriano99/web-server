# Learnings — running notes

Gotchas and concepts from each session, with the verification angle.
Interview material in progress. One section per session; self-check questions at the end
of each section (answers below them, no peeking).

## 2026-09-10 — Phase 0: first firmware on the chip

### Concepts

- **Multimeter modes**: continuity (buttons), diode mode → shows the LED's forward voltage
  in **mV** (not resistance), Ω ranges (200 / 2k / 200k …) = the ceiling of each range;
  pick the smallest range that fits.
- **LED + series resistor**: `R = (Vsupply − Vf) / I`. Measured Vf = 1.48 V (red LED),
  target ~10 mA → 220 Ω. The resistor is mandatory; electrically it can sit on either side
  of the LED (series loop, same current).
- **ESP32 GPIO**: 3.3 V logic, keep ≤ ~20 mA per pin. For inputs: avoid strapping pins
  (GPIO0, GPIO2, GPIO5, GPIO12, GPIO15) and input-only pins (GPIO34–39, no internal pull-up).
- **Breadboard**: columns of 5 holes are internally connected; the side rails (red = +,
  blue = −) are isolated strips — connected to nothing until you jumper them. Colors are
  a convention, not a fact.
- **ESP-IDF project anatomy**: project `CMakeLists.txt` (entry) → `main/CMakeLists.txt`
  (`idf_component_register`) → `main.c` with `app_main()` (no classic `main()`).
- **Build chain**: CMake + Ninja → cross-compiler (`xtensa-esp-elf-gcc`) → linker →
  3 binaries: bootloader (`0x1000`), partition table (`0x8000`), app (`0x10000`).
- **Flash vs monitor**: `flash` writes the binaries into the chip's flash; `monitor` is
  the serial console.
- **FreeRTOS**: `vTaskDelay()` yields the CPU to the scheduler (other tasks run) — unlike
  a busy-wait `delay()`.

### Gotchas (and their verification counterpart)

- `CMakeLists.text` ≠ `CMakeLists.txt` — exact names matter; `ls` was already telling us.
  → *Read the tool's complaint literally; check the obvious first.*
- CMake comments are `#`, not `//`.
- WSL caches the Windows PATH: after installing `usbipd`, `usbipd.exe` is not found until
  `export PATH="$PATH:/mnt/c/Program Files/usbipd-win"` (or a WSL restart).
- `usbipd attach` drops on Windows sleep / reboot / replug → re-run
  `usbipd.exe attach --wsl --busid 3-1`. `dmesg` tells the whole story.
- `/dev/ttyUSB0` is `root:dialout`; `newgrp dialout` grants access per shell (the group
  applies fully after a WSL restart).
- Monitor exit on a Spanish keyboard: `Ctrl+]` needs AltGr and never reaches the terminal
  → use `Ctrl+T` then `Ctrl+X`. Also: `Ctrl+T Ctrl+H` = help menu;
  `Ctrl+T Ctrl+F` = rebuild + reflash from inside the monitor.
- Boot-log warning `Detected size(4096k) larger than the size in the binary image
  header(2048k)` → firmware configured for 2 MB on a 4 MB board; fixed via `menuconfig`.
  → *Warnings are findings: reconcile configuration with hardware.*
- The boot log's `App version:` is the **git commit hash** → firmware traceability.

### Self-check (try first, answers below)

1. Why does diode mode show `1480` and not `1480 Ω`?
2. Compute R for a blue LED (Vf 3.0 V) at 8 mA. Which standard value would you use?
3. What is the difference between `idf.py flash` and `idf.py monitor`?
4. What are the three flashed binaries for, and at which addresses do they go?
5. Why doesn't `vTaskDelay` block the processor like a busy-wait would?
6. The board is unplugged and replugged — what must be repeated, and with which command?
7. Why was the flash-size warning important, and how was the fix verified?
8. Where in the boot log can you see which firmware version the chip is running?

### Answers

1. Diode mode measures **voltage** across the diode, in millivolts: 1480 mV = 1.48 V = Vf.
2. R = (3.3 − 3.0) / 0.008 = 37.5 Ω → next standard value up: 39 Ω (47 Ω also fine).
3. `flash` writes the binaries into the chip's flash memory; `monitor` opens the serial console.
4. Bootloader (`0x1000`) starts the chip; partition table (`0x8000`) maps the flash;
   app (`0x10000`) is your program.
5. `vTaskDelay` blocks the *task* and yields the CPU to the FreeRTOS scheduler; a busy-wait
   burns cycles doing nothing.
6. `usbipd` loses the attach → repeat `usbipd.exe attach --wsl --busid 3-1`; if permissions
   complain, `newgrp dialout`.
7. Config said 2 MB but the board has 4 MB (half the flash wasted). Verified: the warning
   disappeared and the log shows `SPI Flash Size : 4MB`.
8. `app_init: App version:` — the git commit hash the firmware was built from.
