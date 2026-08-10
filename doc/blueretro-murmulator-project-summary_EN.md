# Project Summary: BlueRetro — connecting 2 NES/Dendy joysticks to Murmulator

## Related documents
- **Base project**: [BlueRetro](https://github.com/darthcloud/BlueRetro) (darthcloud) — an open-source Bluetooth controller adapter for retro consoles based on ESP32, whose firmware and HW1 specification this build is based on. Documentation: [BlueRetro Wiki](https://github.com/darthcloud/BlueRetro/wiki).
- **Related host project**: `murmulator-vga-project-summary.md` (the "Murmulator" project) — one of the Murmulator builds (RP2040 Black Perfboard: VGA, Joystick, AudioOut, AudioIn) used for the current BlueRetro integration; not the only possible target platform. This file is loaded separately in Project Knowledge.

## Project goal
Connect Bluetooth gamepads (Xbox One, PS/Switch, and other HID devices) to the Murmulator (an RP2040-based device) as two NES/Dendy-compatible joysticks via the BlueRetro device, which emulates the standard NES/Dendy shift-register protocol (9-pin, CLOCK/LATCH/DATA) on its output, acting as a bridge between Bluetooth controllers and the Murmulator. Physical wired NES/Dendy joysticks are not used in this project.

## Component context

### Murmulator (retro computer emulator)
- Base: RP2040 black clone, a homemade board (perfboard).
- Functionality (per `murmulator-vga-project-summary.md`): VGA output, joystick support, audio output, audio input.
- This project extends the base platform with support for two NES/Dendy joysticks via BlueRetro.

### BlueRetro (adapter)
- Open hardware and firmware based on ESP32, licensed under CERN-OHL-P-2.0 (hardware) and Apache-2.0 (software).
- Originally designed as a "Bluetooth controller → console" adapter: <cite index="5-1">supports Wii, Switch, PS3, PS4, PS5, Xbox One, Xbox Series X|S, and generic HID Bluetooth (BR/EDR and LE) devices, and emulates the protocols of many consoles on the output, including NES</cite>.
- For NES/Famicom: <cite index="2-1">can emulate a standard 8-button NES controller (GamePad), the Four Score multitap for 4 controllers (Dual), the Japanese 4P adapter (Alt), the Famicom keyboard, or the Hori Trackball (mouse)</cite>.
- This project uses the **standard** BlueRetro operating scenario: the adapter is physically connected to the joystick port of the host (in this case, the Murmulator) and emulates the standard NES/Dendy shift-register protocol on its output, while accepting Bluetooth/HID gamepads (Xbox One, etc.) as input. Wired NES/Dendy joysticks are not used on the BlueRetro input.
- NES cable wiring diagrams and PAL pull-up specifics are described in [BlueRetro Cables Build Instructions](https://github.com/darthcloud/BlueRetro/wiki/BlueRetro-Cables-Build-Instructions): <cite index="4-1">for PAL systems, pull-up resistors of 3.6K to NES 5V (pin 5) need to be added to pads IO18, IO5, and IO32</cite>.
- The overall build is described in [BlueRetro DIY Build Instructions](https://github.com/darthcloud/BlueRetro/wiki/BlueRetro-DIY-Build-Instructions): flashing BlueRetro firmware onto the ESP32, building the adapter cable for the target system, optional configuration via Web Bluetooth.
- **In this project**: the BlueRetro board has 2 JST XH 1x04 connectors (J1/J2) soldered on, to which homemade adapter cables (JST XH 4-pin male → DE-9 female) are connected, terminating in DE-9 female connectors — these plug directly into the DE-9 male joystick ports on the Murmulator.
- Configuration is done via BLE Web Config (Desktop/Android Chrome), which allows flexible button mapping, turbo mode configuration, etc.

## Dev board (ESP32) selection

**Final decision: ESP32 D1 mini MH-ET LIVE (CH9102).**

**Alternatives considered:**

| Board | Verdict |
|---|---|
| ESP32-WROOM-32 DevKit, **30 pin**, CP2102 | Rejected — this form factor physically **does not break out GPIO0**, which is required for BOOT/pairing and all BlueRetro reset modes (verified against the pinouts of the specific boards on hand) |
| ESP32-WROOM-32 DevKit, **38 pin**, CP2102 | Technically suitable and matches the official recommendation from the BlueRetro author (see below) — GPIO0 is broken out. Not chosen, as it would require reworking the already finished and hardware-verified KiCad schematic (different footprint, different board dimensions); at the current stage (PCB nearing finalization) this was deemed impractical |
| ESP32 D1 mini MH-ET LIVE, CH9102 (chosen) | The schematic and pinout are already designed and hardware-verified for this board; no on-board peripheral conflicts were found with the pins used (IO2, IO4, IO5, IO17, IO18, IO19, IO22, IO32, IO0) |

**Board on-board LED:** on the ESP32 D1 mini MH-ET LIVE, the built-in LED is connected to **GPIO2** and lights up when GPIO2 is **HIGH**. In this build, GPIO2 is used for port 1 indication (see "LED indicator subsystem" section below) — when IO2 is HIGH (port 1 LED via MOSFET Q1 is on), the board's built-in LED will mirror this state in parallel (lit at the same time as the external port 1 LED). This does not create an electrical conflict (GPIO2 remains an output; the on-board LED is just additional, somewhat redundant indication of the same signal), but it's worth keeping in mind when assembling in an enclosure — the board's built-in LED may be visible from outside if not covered.

**Official position of the BlueRetro community:** the project author (darthcloud) specifies a particular recommended board in the official documentation — the **ESP32-DevKitC-32E/D with the ESP-WROOM-32 module** (38-pin form factor). Reasons:
- Cheap clones sometimes have on-board peripheral conflicts with the pins used by BlueRetro (a documented case — an LED on IO5 on the Wemos Lolin32 board, which causes the firmware to hang in a reset loop).
- It's important that the module inside is specifically **WROOM** (not WROVER) — WROVER has 2 fewer GPIOs due to built-in PSRAM.
- Third-party manufacturers of ready-made BlueRetro kits (e.g., RetroRosetta) follow the same recommendation — use only official DevKit boards.

**Why the "built-in BOOT button" argument didn't matter:** the BOOT and RESET buttons in this project are routed out to the device enclosure (a 3D-printed box) via separate wires regardless — so the presence or absence of a built-in BOOT button on the board itself did not factor into the board choice.

**Known trade-off of the chosen board — regulator heating:** the built-in linear regulator **AMS1117** (5V→3.3V) on the D1 mini heats up to ~54°C during operation. This isn't a problem unique to the D1 mini — the same AMS1117 is on WROOM-32 DevKit boards too, but the D1 mini's small board has less PCB copper for heat dissipation. 54°C isn't critical for the regulator itself, but it's close to the softening temperature of PLA (used for the 3D-printed enclosure) — with a tight layout, the plastic could deform during long operation without ventilation.

⚠️ **The heating issue has been deliberately deferred** (risk accepted, not a build blocker). If needed in the future, two options:
1. A ventilation hole in the enclosure above the regulator.
2. Bypassing/desoldering the built-in AMS1117 and supplying ready-made 3.3V from a separate external DC-DC regulator (similar to the SD module modification in `murmulator-vga-project-summary.md`, section 2).

## BlueRetro hardware build
- A DIY build based on the **ESP32 D1 mini (MH-ET LIVE, CH9102 USB-serial chip)** is used — homemade, not a ready-made BlueRetroHW board.
- **⚠️ No BOOT/IO0 button on the D1 mini board.** Unlike the full-size ESP32 DevKitC, the D1 mini only has a **RST/EN** button (module reset) physically wired — there is no separate BOOT button. BlueRetro functions (pairing, config reset, factory reset — see "BOOT button (IO0)" section below) are tied to IO0, so a **separate button between IO0 and GND had to be added**.
- An external pull-up resistor is not theoretically required — the ESP32 enables an internal pull-up on IO0 (a strapping pin) in hardware at boot. The button simply shorts IO0 to GND when pressed.
  - **In practice, the built-in pull-up on this particular module turned out to be unstable** (see the "LED subsystem" section — a known glitch of spontaneous 6–10 sec reset triggering), so in the final KiCad schematic an external resistor **R1 = 10 kΩ (from IO0 to +3.3V) is soldered in unconditionally**, as a standard component rather than an optional fallback.

### BOOT button (IO0) — BlueRetro functions
⚠️ The timings and actions below correspond to the firmware version in use (`v25.04_hw1`, see the "Firmware" section). In other BlueRetro versions, the hold durations and results may differ — check the BlueRetro Wiki for the specific installed version.
- Short press (< 3 sec) — regular system reset.
- Hold 3–6 sec (IO17 LED blinks slowly) — enters pairing mode.
- Hold 6–10 sec (IO17 LED blinks fast) — resets to default configuration.
- Hold > 30 sec — full ESP32 factory reset (factory firmware + settings reset).
- In pairing mode, a short press stops pairing / disconnects all Bluetooth devices.

## Pinout (coordinated with murmulator-vga-project-summary.md)

**Topology:** ESP32 (BlueRetro) → J1/J2 connectors (JST XH 1x04, on the BlueRetro board) → adapter cable (JST XH 4-pin male → DE-9 female) → J6/J7 connectors (DE-9 male, on the Murmulator board) → Pico GPIO.

| Signal | ESP32 pin (BlueRetro) | JST XH J1 (P1) | JST XH J2 (P2) | DE-9 Murmulator J6 (P1) | DE-9 Murmulator J7 (P2) | Pico GPIO |
|---|---|---|---|---|---|---|
| LATCH (shared) | IO32 | pin 1 | pin 1 | pin 3 | pin 3 | GP15 |
| CLOCK port 1 | IO5 | pin 2 | — | pin 4 | — | GP14 |
| CLOCK port 2 | IO18 | — | pin 2 | — | pin 4 | GP14* |
| DATA port 1 | IO19 | pin 3 | — | pin 2 | — | GP16 |
| DATA port 2 | IO22 | — | pin 3 | — | pin 2 | GP17 |
| GND | GND ESP32 | pin 4 | pin 4 | pin 8 | pin 8 | GND |
| VCC (+5V) | — | not connected | not connected | pin 6 | pin 6 | **do not connect** |

**Adapter cable wiring (JST XH 4-pin male → DE-9 female), the same for both ports:**

| JST XH pin | Signal | DE-9 pin |
|---|---|---|
| 1 | LATCH | 3 |
| 2 | CLOCK | 4 |
| 3 | DATA | 2 |
| 4 | GND | 8 |

\* **CLOCK and LATCH are generated by the RP2040 (the Murmulator acts as host/console)** — this is a single physical signal on GP14/GP15, routed to both J6 and J7 connectors simultaneously. IO5 and IO18 on the ESP32 (BlueRetro emulates the joystick/slave) are two **inputs** listening to this signal, not two independent generators. There is no electrical conflict (bus contention) from combining them, since the RP2040 is the sole active driver of the line. The BlueRetro firmware internally handles IO5/IO18 as two independent CLOCK inputs (for consoles where CLOCK P1/P2 are not physically joined) — when connected to the Murmulator, both inputs inevitably receive the same physical signal. An oscilloscope check before soldering is still recommended — not to compare "two generators," but to confirm the signal reaches both ESP32 inputs cleanly, without distortion from wires/connectors.

VCC (pin 6, DE-9 Murmulator) is deliberately not connected: the ESP32 inside BlueRetro runs on 3.3V logic, the signal lines go directly to the Pico GPIOs without level shifters; BlueRetro's power supply is arranged separately, via its own connector J3 on the board (see "BlueRetro power supply" section). On the JST XH J1/J2 connectors (joystick ports on the BlueRetro side), there is no +5V line at all — only 4 signals (LATCH/CLOCK/DATA/GND).

## BlueRetro power supply (ESP32 module)
- The ESP32 module (DevKit with a built-in AMS1117 regulator, 5V→3.3V) is powered by a separate wire from the Murmulator's **external PSU 5V bus** (not from Vout/USB) — the YD-RP2040 has no "clean" USB VBUS available before the BAT54C combining diode, so 5V from USB is not available for the ESP32.
- **⚠️ Limitation (when powered via the PSU 5V bus):** BlueRetro and both joysticks only work when an external PSU is connected to the Murmulator — if the board is powered only from the Murmulator's USB (without an external PSU), the joysticks will not work. This limitation does not apply with the alternative ESP32 power option via its own USB port (see below).
- **Measured (multimeter):** peak current draw of the ESP32 module on the 5V bus with active BT radio — **~155 mA**. Since the ESP32 is powered by a separate wire directly from the PSU bus (bypassing the BAT54C diode and the Vout node, see `murmulator-vga-project-summary.md`), this load does not share the current budget with the PS/2 keyboard and is not limited by the BAT54C rating (~150–300 mA per leg) — there is sufficient current headroom.
- **Alternative power option:** it's possible to power BlueRetro via the ESP32 module's own USB port (micro-USB on the D1 mini board) instead of the Murmulator's PSU 5V bus. The GND of the signal lines (LATCH/CLOCK/DATA) remains shared with the Murmulator either way — through the J1/J2 (JST XH) connectors and the adapter cable to J6/J7, regardless of the 5V source for the module itself. With this option, the limitation above (mandatory external PSU on the Murmulator) is removed — ESP32/BlueRetro power no longer depends on whether a PSU is connected to the Murmulator.

## Input power protection, RESET, and decoupling

**External 5V connector (J3) and protection diode D1:**
- **J3** — a separate 2-pin connector ("External 5V Power Connector") through which power from the external PSU's 5V bus is fed to the board (see "BlueRetro power supply" section above).
- **D1 = 1N5819** (Schottky, 40V/1A) is placed between J3 and the +5V bus: anode on the J3 side, cathode on the +5V side. Function — reverse-polarity protection / input isolation (analogous to the diode-OR BAT54C on the Murmulator itself, see `murmulator-vga-project-summary.md`, section 10).

**RESET button (SW2):**
- A separate **SW2** button, shorting the ESP32 module's **RST** to GND — a standard hardware module reset, independent of the BOOT/IO0 logic (see the BOOT button function table above).

**Decoupling capacitors (bulk + bypass on both power rails):**

| Rail | Bulk (electrolytic) | Bypass (ceramic) |
|---|---|---|
| +5V (input, after J3/D1) | C5 = 470 µF | C6 = 100 nF |
| +3V3 (output of the ESP32 module's regulator) | C8 = 220 µF | C7 = 100 nF |

Both capacitors on each rail are connected in parallel, between the rail and GND — the classic "bulk filtering + high-frequency decoupling" pair.

---

## LED indicator subsystem (final, hardware-verified)

Per the BlueRetro Wiki documentation (LED usage sections), indication consists of two types of LEDs: **one global-status LED on IO17** and up to **four port-status LEDs** (one per port). This build implements the global LED plus two port LEDs (for both joystick ports, P1/P2).

**Note on IO17 vs. the board's built-in LED:** on some ESP32 DevKitC/D1 mini boards, the built-in LED sits on IO2, but in the BlueRetro firmware this pin is used for the auto-programming circuit — so a separate external LED on IO17 specifically is needed for indication, rather than relying on the board's built-in power/status LED.

**ESP32 pin assignment:**

| ESP32 IO | Function | LED | Connection method |
|---|---|---|---|
| IO17 | Global status (adapter status / error / pairing) | D3, blue | two-resistor pull-up circuit (direct) |
| IO2 | Port 1 status | D4, green | via MOSFET Q1 (2N7000) |
| IO4 | Port 2 status | D5, green | via MOSFET Q2 (2N7000) |

IO2/IO4/IO17 are all ESP32 strapping pins. Only the port LEDs D4 (IO2) and D5 (IO4) are routed through MOSFETs — a requirement of this BlueRetro hardware revision, so the LED load doesn't interfere with boot; IO17 (D3) uses a separate pull-up circuit without a MOSFET (see above). Ports 3/4 (if expanded) are IO12/IO15, but in PlayStation mode they're reassigned for analog-mode indication; not relevant for NES mode.

**Behavior logic (confirmed by real testing):**
- Power on, no error → auto-pairing: global LED (IO17) **pulses** + LED of the first available port (IO2) **pulses**; the second port (IO4) is off.
- 1st gamepad connected → IO17 turns off, port 1 (IO2) **solid**.
- 2nd gamepad connected → port 2 (IO4) **solid**, IO17 remains off.
- Gamepad disconnected → its port LED turns off; once the port frees up, the adapter returns to pairing mode (global + first free port pulse again).

### Global-status LED IO17 circuit (two-resistor variant)

⚠️ Differs from the official single-resistor wiki circuit (R36 ≈ 65 Ω–1 kΩ, LED directly between the node and GND). In this build — **two resistors**, which improves behavior:

```
+3V3 --[R2 360R]--+-- IO17 (U1 pin 25)
                  |
                  +--[R3 150R]--[D3 anode >|< cathode]-- GND
```
IO17 is connected to the node between R2 and R3 (on the LED anode side), not to the GND side.

**Roles of the resistors:**
- **R2 = 360R** — pull-up 3.3V→IO17. Provides fail-safe behavior (the LED glows dimly at high-Z) and sets the parasitic sink current into the GPIO at LOW. Does **not** affect operating brightness.
- **R3 = 150R** — series current-limiting resistor for the LED. Sets the brightness during HIGH-PWM (pulsing/solid). Limits the current when actively driven push-pull HIGH.

**Logic (fail-safe by default):**
- IO17 by default (unconfigured, high-Z/input) → R2 pulls the node toward 3.3V → **the LED glows dimly on its own**, without any firmware command.
- After a successful boot, the firmware explicitly **drives IO17 as an output LOW** → the node goes to 0V → **the LED turns off**.
- The point of the inversion: if the firmware fails to boot (crashes before initialization) — IO17 stays high-Z, and the LED **keeps glowing** = error indication. In a classic circuit, such a failure would be indistinguishable from a normal "off" state.
- Pulsing/status: after boot, the firmware uses LEDC to switch IO17 into PWM (~200 µs period, 0–20% duty cycle), creating a "breathing" effect. Confirmed with an oscilloscope.

### Port-status LED circuit (IO2/IO4, via MOSFET)

```
+3V3 --[D4/D5 anode >|< cathode]--[R4/R5 510R]-- drain Q1/Q2
                                                 gate  -- IO2/IO4
                                                 source-- GND
```
When IO2/IO4 is HIGH → the MOSFET opens → the LED lights up. When the MOSFET is closed, no current flows at all — unlike IO17, there's **no** "sink into GPIO" issue here, so the port LEDs can run at full operating current without worrying about GPIO loading.

### Final component values and measured currents

Vf values were measured with a multimeter on the LEDs through a 510R resistor to 3.3V (high input impedance → an honest operating point): **blue Vf = 2.6 V @ ~1.4 mA, green Vf = 2.4 V @ ~1.76 mA.**

| LED | Color | Vf | Circuit | Current | State |
|---|---|---|---|---|---|
| D3 (IO17) | blue | 2.6 V | R3 = 150R | ~4.0–4.5 mA | HIGH-PWM (pulsing/status) |
| D3 (IO17) | blue | 2.6 V | R2 = 360R | ~9.2 mA | LOW — sink into GPIO (depends only on R2) |
| D3 (IO17) | blue | 2.6 V | R2+R3 = 510R | ~1.37 mA | high-Z — fail-safe (boot error) |
| D4/D5 (ports) | green | 2.4 V | R4/R5 = 510R | ~1.76 mA | solid (port connected) |

**Brightness balance:** the green port LEDs run at a lower current (~1.76 mA) than the blue status LED (~4.5 mA), but thanks to the eye's peak sensitivity to green (~555 nm) and lower sensitivity to blue, they appear visually comparable. R4/R5 = 510R — at a lower value (150R) the green would become brighter than the blue, breaking the balance.

**Visual check:** with the blue (IO17) and green (port 1) LEDs lit simultaneously during pairing, the brightnesses are comparable. If needed, R4/R5 can be lowered to ~330R (~2.7 mA) — but no lower.

**Voltage headroom:** the blue LED has little headroom (3.3 − 2.6 = 0.7 V), so brightness is sensitive to dips in the 3.3V rail — at 3.2 V, the HIGH current drops to ~4 mA, which remains within normal range.

## Oscilloscope signal verification (confirmed)

**CLOCK (GP14 → IO5/IO18):** period 80 µs (≈12.5 kHz), 50% duty cycle (HIGH = LOW = 40 µs), amplitude 0V ↔ 3.3V, clean edges.

**LATCH (GP15 → IO32):** repeat period 1200 µs (≈833 Hz), pulse width 40 µs, amplitude 0V ↔ 3.3V.

- The LATCH pulse width (40 µs) equals one CLOCK phase — typical for latching a parallel-loaded button state before a series of shift clocks.
- One LATCH period contains 1200 / 80 = 15 full CLOCK cycles — enough to read out an 8-bit register with margin.
- The polling rate (~833 Hz) is noticeably higher than classic NES/Famicom (~60 Hz, synced to the frame) — presumably a deliberate decision in the Murmulator firmware to reduce input lag.
- The 0V/3.3V amplitude on both lines confirms proper operation of the "one driver (Pico) — receivers (ESP32)" topology, without bus contention.

**DATA (IO19/IO22 → GP16/GP17):** when buttons are pressed on the Xbox One gamepad (A, X, View, Menu, D-pad Up/Down/Left/Right), the DATA line shows a **LOW (~0V) pulse lasting 80 µs** — exactly one CLOCK period.
**The first button is output without a clock pulse:** as soon as the Murmulator issues the LATCH signal (latch), the chip immediately puts the A button value on the DATA line (this is bit 0). The following buttons (X, View, Menu, etc.) are output in turn on each subsequent CLOCK pulse.

- The protocol uses **inverted logic**: LOW = button pressed, HIGH (3.3V) = button released — the standard convention for an NES/Dendy-compatible shift register.
- The pulse width (80 µs) exactly matches the CLOCK period — confirming that each bit is placed on DATA for exactly one shift clock, in sync and without phase shift.
- The correspondence between "a physically pressed Xbox button → a LOW pulse in the expected clock cycle" confirms correct BlueRetro mapping (Xbox One HID → NES bits) and correct reading by the Murmulator on the RP2040 side.

**DATA bit order in the NES shift register and correspondence to Xbox One (confirmed):**

| Clock | Bit | Xbox One button | Role (NES bit) |
|---|---|---|---|
| 1 | 0 | A | A |
| 2 | 1 | X | B |
| 3 | 2 | View | Select |
| 4 | 3 | Menu | Start |
| 5 | 4 | D-pad Up | Up |
| 6 | 5 | D-pad Down | Down |
| 7 | 6 | D-pad Left | Left |
| 8 | 7 | D-pad Right | Right |

The bit order **A, B, Select, Start, Up, Down, Left, Right** is the standard, canonical bit output order of the original NES controller (the same order in which the NES console historically read the 4021 register). The match confirms that BlueRetro in `GamePad`/NES mode emulates the classic protocol one-to-one, without altering bit order, and the Murmulator reads them in the same order.

The verification confirms both the electrical integrity of the CLOCK/LATCH signals and correct phase alignment of DATA relative to CLOCK — data is sampled at the correct point in the clock cycle.

## Firmware (verified, working version)

**Firmware version:** `v25.04_hw1.zip` (archive from [GitHub Releases](https://github.com/darthcloud/BlueRetro/releases)) — using the HW1 specification (see "BlueRetro hardware build" section).

**Files from the archive:**
- `bootloader.bin`
- `partition-table.bin`
- `ota_data_initial.bin`
- `BlueRetro_hw1_nes.bin` (system-specific firmware for NES)

**Flashing command (Windows, via PlatformIO Core):**

```
pio pkg exec -- esptool.py -p COM6 -b 460800 --before default_reset --after hard_reset --chip esp32 write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x1000 bootloader.bin 0x8000 partition-table.bin 0xD000 ota_data_initial.bin 0x10000 BlueRetro_hw1_nes.bin
```

The port `COM6` is specific to the particular board instance — check in Windows Device Manager each time you flash.

**Note:** calling `python esptool.py ...` directly (not through `pio pkg exec`) may be missing dependencies for esptool, since the system Python doesn't see the environment where PlatformIO installed them — the `pio pkg exec --` command works around this by automatically using the correct environment.

**Full flash erase:**
```
pio pkg exec -- esptool.py -p COM6 --chip esp32 erase_flash
```

**Serial Monitor Debug:**
```
pio device monitor -p COM6 -b 921600
```

**SecureCRT Serial Monitor settings:**
```
Connection:
  Serial:
    Port: COM6
    Baud rate: 921600
Terminal:
  Emulation:
    Modes:
      [v] New line mode
```

### Initial setup after flashing (BlueRetro Web Configurator)

After flashing, configuration via Web Config may be needed — in this case the build worked with default settings; the configuration below is provided for reference/verification.

**Address:** [https://blueretro.io/](https://blueretro.io/)

⚠️ BlueRetro configuration is only accessible when **no controller is connected** — this is intentional, so as not to take airtime away from gamepads during gameplay.

**Steps:**
1. Open **"BlueRetro Advance config"**.
2. Click **[Connect BlueRetro]** — repeat the click if needed until a connection is established.
3. A pop-up window with a list of Bluetooth devices will open — select the BlueRetro device and click **[Pair]**.
4. **Global Config:**
   - System: `NES`
   - Multitap: `None`
5. **Output Config:**
   - Output 1 → Mode: `GamePad`, Accessories: `None`
   - Output 2 → Mode: `GamePad`, Accessories: `None`
6. **[Save]**

## Pairing and disconnecting gamepads

Practical instructions for pairing and disconnecting/unpairing Bluetooth gamepads are provided in a separate document, **`blueretro-murmulator-user-guide.md`** (see Project Knowledge).

## Open questions (require clarification/verification)
*(none at this time)*
