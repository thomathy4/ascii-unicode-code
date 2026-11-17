# Hardware extension sketch — ASCII/Unicode escape code

**Status: draft sketch only, 2026-11-17 — pushed to thomathy4/ascii-unicode-code as HARDWARE-EXTENSION-SKETCH.md.** Not part of code v1.1.0, not implemented. The current code encodes characters; it does not touch hardware. This sketch asks: what would it take for the same ESC mechanism to reach machine fundamentals?

## The one principle

An encoding is representation, not access. A sequence only reaches hardware if something on the other end reads it and acts. So every row below needs an **interpreter** — a small program on the device that parses the escape sequence and calls the real hardware function. The code supplies the language; the interpreter supplies the hands.

Two facts make the code a good fit for that language:

1. **It is ASCII-only (bytes 0x00–0x7F).** It travels over any 7-bit channel: serial UART, pipes, old terminals, embedded links that cannot carry UTF-8.
2. **It already uses the hardware-control shape.** `ESC[ ... m` is the exact form terminals have used since VT100 to control display hardware. The code rides a channel devices already parse byte by byte.

## Proposed sequence family (sketch)

Keep the existing `ESC[Uxxxxm` (Unicode character) untouched. Add a second family with a device letter, so a parser can tell text from command at the first byte after `ESC[`:

| Family | Shape | Meaning |
|---|---|---|
| `ESC[Uxxxxm` | existing | a Unicode character (unchanged) |
| `ESC[H<device><op>...m` | new | a hardware command for device `<device>` |
| `ESC[Q...m` | new | a query — device replies in the same code |

Details of `<device>` and `<op>` are per machine (below). Nothing here is final; the point is one shape for all machines, so the same encoder and the same parser skeleton work everywhere.

## Machines, at a glance

| Machine | Channel | What sequences could do | Interpreter needed |
|---|---|---|---|
| Terminal / display (VT100, ANSI, xterm) | stdout / pty | Cursor, colour, clear, bell, title — the classic set; our `ESC[U...m` already lives beside these | The terminal itself (already exists) |
| Serial UART device (generic) | RS-232 / USB-serial, 7-bit | Carry any Unicode text to a device that only speaks ASCII; send control codes (BEL, DC1–DC4) | A serial parser on the device |
| Microcontroller — Arduino / Pico / ESP32 | UART / USB CDC | Set a pin high/low, read a pin or ADC, blink an LED, read a sensor, reset | Firmware loop reading the stream (e.g. CircuitPython / Arduino sketch) |
| Single-board computer — Raspberry Pi | GPIO via Linux, UART, I2C/SPI | GPIO read/write, PWM, I2C sensor reads, SPI transfers | A small daemon translating sequences to sysfs / libgpiod calls |
| Desktop OS — Linux / macOS / Windows console | pty / console API | Terminal control, clipboard, notifications, beep | A shell-side reader (Python stdlib only, as now) |
| Android phone (e.g. Cricket Magic 2 5G) | intents / USB / ADB | Vibrate, flash, notification, clipboard, read battery/sensors | An app or ADB-side bridge; Android sandboxes raw hardware, so this is the thinnest path |
| Printer (ESC/P, PCL) | USB / network | ESC/P already *is* an escape-sequence printer language — same idea, different dialect | The printer firmware (already exists) |
| CNC / 3D printer | USB-serial | G-code is ASCII text sent over serial — the closest existing cousin: text commands driving physical motion | The printer's G-code interpreter (already exists) |
| MIDI hardware (keyboard, synth) | MIDI DIN / USB-MIDI | Notes, control change, clock — MIDI messages are short byte sequences, like escape codes | A MIDI bridge mapping sequences to MIDI messages |
| Network router / switch (CLI) | SSH / serial console | Configure interfaces, read status — CLIs already accept ASCII command text | The device CLI (already exists) |
| Mainframe terminal (3270 / VT) | TN3270 / serial | Field navigation, screen writes, attention keys | The terminal emulator (already exists) |
| Game console / retro computer | Serial / cartridge | Text-mode screen control, sound chip registers, controller reads | A homebrew ROM or serial monitor |

## What is shared across all of them

- **One encoder.** The existing `ascii_unicode_code.py` already emits ASCII-only output; no change needed to carry hardware commands as text.
- **One parser skeleton.** Read bytes, watch for ESC, dispatch on the letter after `ESC[`: `U` = character, `H` = hardware command, `Q` = query.
- **One safety rule.** Hardware commands must be opt-in and allowlisted per device — a sequence that sets a pin on a Pico must never be accepted blindly by a printer. The interpreter decides what it honours; the code never assumes.
- **One limit, stated plainly.** Text can describe an action; only the interpreter performs it. There is no sequence in this sketch that bypasses a device's own permissions, sandbox, or firmware.

## What would come next (if approved)

1. Pick one machine to prove it on — the cheapest is a terminal (interpreter already exists) or a Pico over USB-serial (smallest firmware).
2. Fix the `H` family for that machine only: device letter, op codes, reply format.
3. Write the interpreter (tens of lines), test it against the existing encoder, and only then generalise to a second machine.

No hardware was touched, no device was changed, and no code was written for this sketch.
