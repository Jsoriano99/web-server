# Verification Lab — Design Doc

Date: 2026-09-09
Status: Approved (with user adjustments)

## Objective

Personal daily-practice lab (30–45 min/day) to develop the user's profile as a chip
verification engineer. Every exercise follows the verification cycle:

    stimulus -> DUT -> monitor -> check

## Context

- User: starting from zero (no microcontroller programming, no HDL).
- Board: ESP32-WROOM-32D (DevKit with USB-UART).
- Two tracks decided: physical ESP32 (bare-metal C via ESP-IDF) + RTL simulation
  (SystemVerilog + Verilator), alternating days.

## Repo Layout

```
esp32/
├── README.md          # Lab rules + how to run exercises
├── pace.md            # Monthly exercise calendar
├── esp32-lab/         # Physical track (ESP-IDF, C bare-metal)
│   └── e01-blinky/    # src/ + README.md log
├── sim-lab/           # Simulation track (SystemVerilog + Verilator)
│   └── e01-and-gate/  # dut.sv, tb.sv, Makefile
└── notes/learnings.md # Gotchas + their verification counterpart
```

> **Update (2026-09-10):** the physical track's first mini-project lives in `web-server/`
> and absorbs `e01`–`e02` (see [web-server design](2026-09-10-web-server-design.md)).
> The home of later physical exercises is TBD.

## Tooling

- ESP-IDF v6.1 (current stable) — real register access + datasheet reading.
- Verilator — free RTL simulator on Linux.
- No Arduino/MicroPython shortcuts; bare-metal first.

## Routine (alternating days)

- Mon/Wed/Fri: ESP32 track — read a datasheet section, write minimal driver,
  verify with LED/UART.
- Tue/Thu: simulation track — simple DUT + testbench with explicit checks,
  later simple coverage.
- Sunday: weekly log in notes/learnings.md.

## First Slice (Week 1)

1. Toolchain setup: ESP-IDF + Verilator (one-time).
2. `e01-blinky` (board): GPIO registers, clock, reset -> log.
3. `e01-and-gate` (sim): first testbench with $display checks.
4. `e02` both tracks: button+UART on board; counter+checker in sim.

## Learning Rules (user-specified)

1. Explain everything: what each piece is and why it is used.
2. Start with very basic programs; progress incrementally.
3. User ALWAYS types the code by hand. No copy-and-paste unless the user says
   otherwise.

## Verification Mindset (the value)

Each exercise ends with: "what would a verification engineer do here?"
Detect boot flakiness, glitches, and race conditions; record them in the log.
These notes become interview material.
