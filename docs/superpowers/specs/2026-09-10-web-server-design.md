# Web Server Project — Design Doc

Date: 2026-09-10
Status: Approved (2026-09-10)
Parent: [Verification Lab design](2026-09-09-verification-lab-design.md)

## Objective

Firmware for the ESP32-WROOM-32D that joins the home Wi-Fi network and serves a web page
from which the user can:

1. Turn a physical LED on and off from any browser (phone or PC).
2. See the live state of a physical push button (pressed / released).

**Acceptance criteria — the project is done when:**

- [ ] The LED toggles from the web UI within ~200 ms of tapping the virtual button.
- [ ] The physical button state appears in the UI within ~200 ms of press/release.
- [ ] Neither action blocks the other: the page stays responsive while the button is spammed.
- [ ] No phantom button events — debounce verified over hundreds of presses.
- [ ] Wi-Fi credentials never appear in the repository.

## Context

- First mini-project of the Verification Lab's physical track. It absorbs exercises
  e01 (GPIO output) and e02 (GPIO input) as Phases 0–1, so no work is duplicated.
- This project supersedes the parent's `esp32-lab/` exercise layout for the physical track:
  e01–e02 are absorbed here at `web-server/`, and the home of later exercises is TBD — the
  parent doc's layout will be reconciled once approved (see Open Items).
- This project's `README.md` is its running log; distilled gotchas feed the parent lab's
  weekly `notes/learnings.md` once that file exists.
- Lab rules apply: explain everything, progress incrementally, **the user types all
  code by hand**, and each phase closes with a verification-minded check.

## Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Toolchain | Native ESP-IDF v5.x on WSL2 (exact version pinned in `README.md` during Phase 0) | Coherent with the lab design: bare-metal, real APIs, FreeRTOS visible. Official examples map 1:1. |
| Framework | ESP-IDF only — no Arduino | `digitalWrite()` / `WiFi.h` would hide exactly the layers we want to learn. |
| HTTP server | `esp_http_server` | Handles connections in dedicated FreeRTOS tasks; the main task never busy-waits on sockets. |
| Project home | `web-server/` | Folder already created; keeps the lab root clean. |
| Secrets | Kconfig → `sdkconfig` (git-ignored); `sdkconfig.defaults` holds non-secret defaults | Credentials never enter git. |
| Language | C (ESP-IDF style) | The lab's physical-track language. |

## Repo Layout

```
web-server/
├── README.md            # phase roadmap + running log (bitácora)
├── .gitignore           # build/, sdkconfig, .cache/, managed_components/
├── CMakeLists.txt       # ESP-IDF project definition
├── sdkconfig.defaults   # tracked, non-secret defaults
├── main/
│   ├── CMakeLists.txt   # component registration
│   └── main.c           # grows phase by phase
└── components/          # (later) own drivers, e.g. the debounced button
```

## Phase Roadmap

Only Phase 0 is planned in detail in this document; each later phase gets its own detailed
plan when we reach it. Phase 6 is explicitly optional.

### Phase 0 — Toolchain + blinky (GPIO output)

- **Concepts:** GPIO output mode, logic levels, series resistor, build/flash/monitor cycle.
- **Deliverable:** LED blinking at a configured rate; toolchain installed and repeatable.
- **Verification:** measure the blink period — does it match exactly? Any jitter or boot flakiness?

### Phase 1 — Digital input: button, pull-up, debounce

- **Concepts:** floating pins, internal pull-up, active-low logic, mechanical bounce, non-blocking timing.
- **Deliverable:** press counter on the serial monitor — raw bounce first, then debounced.
- **Verification:** N presses = N counts over hundreds of presses; try to provoke phantom counts.

### Phase 2 — Wi-Fi station

- **Concepts:** station mode, events, connection lifecycle, credentials handling.
- **Deliverable:** connects to the home router and logs its IP; reconnects after outages.
- **Verification:** kill the AP and confirm recovery; measure reconnection time.

### Phase 3 — HTTP server (port 80)

- **Concepts:** request/response cycle, handlers, which task your code runs in, JSON.
- **Deliverable:** `/` (page), `/led/on`, `/led/off`, `/status` endpoints. `/status` returns
  JSON: `{"led": true|false, "button": true|false}` (button = currently pressed).
  No toggle endpoint: the UI derives the next state from the last `/status` poll, and
  two-client races are explored in Phase 5.
- **Verification:** curl the endpoints; invalid routes; burst requests; check the device stays alive.

### Phase 4 — Web UI

- **Concepts:** minimal HTML/CSS/JS, `fetch()`, polling, same-origin requests.
- **Deliverable:** page with a virtual LED button and live physical-button state.
- **Verification:** control from the phone; two clients at once; observe polling latency.

### Phase 5 — Real-time + concurrency

- **Concepts:** polling vs SSE/WebSocket, FreeRTOS tasks/queues/mutexes, race conditions.
- **Deliverable:** physical button reflected instantly in the UI; shared state made safe.
- **Verification:** hammer rapid events; hunt a race on purpose; document what you find.

### Phase 6 — Extras (pick as interest dictates)

- mDNS (`esp32.local`), OTA updates, persistence in NVS, basic authentication.

## Phase 0 — First Slice (detailed)

1. Component lab: characterize LEDs, resistors, and buttons with the multimeter; confirm the pin plan (LED → GPIO2, button → GPIO4).
2. Install ESP-IDF v5.x on WSL2 (official installer); verify `idf.py --version` and record the exact version in `README.md`.
3. Enable USB passthrough for the board (`usbipd-win`) and fix serial permissions.
4. Create the project skeleton; `idf.py set-target esp32`, build, flash, monitor.
5. Blink the LED from a FreeRTOS task; log the boot banner.

**Exit criteria:** the flash/monitor workflow is repeatable from a fresh terminal, and the
blink period matches the configured value when measured. Log entry written in `README.md`.

## Risks and Gotchas

| Risk | Mitigation |
|------|------------|
| WSL2 cannot see USB serial by default | `usbipd-win` attach; documented during Phase 0 |
| User not in the `dialout` group | add group + re-login, or a udev rule (Phase 0) |
| DevKit may have no user LED | confirm hardware at Phase 0; use an external LED if needed |
| GPIO0 (BOOT) is a strapping pin | last resort for the project button; prefer GPIO4 |
| GPIO2 (LED pin) is also a strapping pin | standard DevKit LED pin, low-risk with a series resistor; verify boot is unaffected |
| GPIO34–39 are input-only, no pull-up | avoid for the button |
| IDF version vs online docs drift | pin the installed version; read versioned docs |
| Credentials leaking into git | Kconfig + git-ignored `sdkconfig`; review before first commit |

## Open Items

- [x] Hardware confirmed (2026-09-10): board + LEDs + resistors + buttons on hand; old buttons to be characterized in the Phase 0 component lab.
- [x] Repo initialized at the lab root (2026-09-10); design docs committed.
- [ ] Confirm pin plan when wiring: LED → GPIO2, button → GPIO4 (internal pull-up).
- [x] Parent layout reconciled (2026-09-10): dated note added to the parent design doc.

## Verification Mindset

Each phase closes with: *"what would a verification engineer do here?"* Look for glitches,
flakiness, timing violations, and race conditions; record findings in the log. These notes
become interview material.
