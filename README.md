# BlueRetro + Murmulator: Bluetooth Joysticks for a Perfboard Retro Computer

Connect Bluetooth gamepads (Xbox One, PS/Switch, and other HID controllers) to a [Murmulator](https://murmulator.ru/howto) retro-computer emulator (RP2040) as two NES/Dendy-compatible joysticks, using a DIY [BlueRetro](https://github.com/darthcloud/BlueRetro) (ESP32) adapter as the bridge.

BlueRetro emulates the standard NES/Dendy shift-register protocol (CLOCK/LATCH/DATA) on its output and plugs directly into the Murmulator's DE-9 joystick ports — no wired NES controllers involved.

## Hardware

- **Host:** Murmulator (RP2040, perfboard build) — VGA, PS/2 keyboard, audio, joystick ports.
- **Adapter:** DIY BlueRetro on ESP32 D1 mini (MH-ET LIVE, CH9102), HW1 firmware spec, custom KiCad schematic with global + per-port status LEDs, input power protection, and a BOOT/IO0 button added off-board.
- **Cabling:** JST XH 4-pin (BlueRetro) → DE-9 (Murmulator), pinout and wiring detailed in the project summary.
- **Schematics:** KiCad project for the BlueRetro adapter board — [`schematics/BlueRetro3V3/`](schematics/BlueRetro3V3/) (plotted PDF: [`BlueRetro3V3.pdf`](schematics/BlueRetro3V3/BlueRetro3V3.pdf)).

## Docs

| File | Description |
|---|---|
| [`doc/blueretro-murmulator-project-summary.md`](doc/blueretro-murmulator-project-summary.md) / [`_EN`](doc/blueretro-murmulator-project-summary_EN.md) | Full technical writeup: board selection, pinout, power, LED indication, firmware, oscilloscope verification |
| [`doc/blueretro-murmulator-user-guide.md`](doc/blueretro-murmulator-user-guide.md) / [`_EN`](doc/blueretro-murmulator-user-guide_EN.md) | Day-to-day usage: pairing and unpairing Bluetooth gamepads |
| [`murmulator-vga-project-summary.md`](https://github.com/alex-snigir/Murmulator-RP2040-Black-Perfboard-VGA/blob/main/doc/murmulator-vga-project-summary.md) / [`_EN`](https://github.com/alex-snigir/Murmulator-RP2040-Black-Perfboard-VGA/blob/main/doc/murmulator-vga-project-summary_EN.md) | Base Murmulator perfboard build (host platform) — separate repo |

## Status

Hardware built and verified: pairing, dual-port switching, and the CLOCK/LATCH/DATA protocol confirmed with an oscilloscope against real Xbox One button presses.

## Credits

Built on [BlueRetro](https://github.com/darthcloud/BlueRetro) by darthcloud (CERN-OHL-P-2.0 / Apache-2.0) and the [Murmulator](https://murmulator.ru/howto) platform.
