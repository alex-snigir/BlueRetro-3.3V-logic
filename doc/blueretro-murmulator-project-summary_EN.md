# Project Summary: BlueRetro — connecting 2 NES/Dendy joysticks to Murmulator

**Document version:** 1.0 · **Date:** 2026-09-30

## Related documents
- **Base project**: [BlueRetro](https://github.com/darthcloud/BlueRetro) (darthcloud) — an open-source Bluetooth controller adapter for retro consoles based on ESP32, whose firmware and HW1 specification this build is based on. Documentation: [BlueRetro Wiki](https://github.com/darthcloud/BlueRetro/wiki).
- **Related host project**: `murmulator-vga-project-summary.md` (the "Murmulator" project) — one of the Murmulator builds (RP2040 Black Perfboard: VGA, Joystick, AudioOut, AudioIn) used for the current BlueRetro integration; not the only possible target platform.

## Project goal
Connect Bluetooth gamepads (Xbox One, PS/Switch, and other HID devices) to the Murmulator (an RP2040-based device) as two NES/Dendy-compatible joysticks via the BlueRetro device, which emulates the standard NES/Dendy shift-register protocol (9-pin, CLOCK/LATCH/DATA) on its output, acting as a bridge between Bluetooth controllers and the Murmulator. Physical wired NES/Dendy joysticks are not used in this project.

## Component context

### Murmulator (retro computer emulator)
- Base: RP2040 black clone, a homemade board (perfboard).
- Functionality (per `murmulator-vga-project-summary.md`): VGA output, joystick support, audio output, audio input.
- This project extends the base platform with support for two NES/Dendy joysticks via BlueRetro.

### BlueRetro (adapter)
- Open hardware and firmware based on ESP32, licensed under CERN-OHL-P-2.0 (hardware) and Apache-2.0 (software).
- Originally designed as a "Bluetooth controller → console" adapter: supports Wii, Switch, PS3, PS4, PS5, Xbox One, Xbox Series X|S, and generic HID Bluetooth (BR/EDR and LE) devices, and emulates the protocols of many consoles on the output, including NES.
- For NES/Famicom: can emulate a standard 8-button NES controller (GamePad), the Four Score multitap for 4 controllers (Dual), the Japanese 4P adapter (Alt), the Famicom keyboard, or the Hori Trackball (mouse).
- This project uses the **standard** BlueRetro operating scenario: the adapter is physically connected to the joystick port of the host (in this case, the Murmulator) and emulates the standard NES/Dendy shift-register protocol on its output, while accepting Bluetooth/HID gamepads (Xbox One, etc.) as input. Wired NES/Dendy joysticks are not used on the BlueRetro input.
- NES cable wiring diagrams and PAL pull-up specifics are described in [BlueRetro Cables Build Instructions](https://github.com/darthcloud/BlueRetro/wiki/BlueRetro-Cables-Build-Instructions): for PAL systems, 3.6K pull-up resistors to NES 5V (pin 5) need to be added to pads IO18, IO5, and IO32.
- The overall build is described in [BlueRetro DIY Build Instructions](https://github.com/darthcloud/BlueRetro/wiki/BlueRetro-DIY-Build-Instructions): flashing BlueRetro firmware onto the ESP32, building the adapter cable for the target system, optional configuration via Web Bluetooth.
- **In this project**: the BlueRetro board has 2 JST XH 1x04 connectors (J1/J2) soldered on, to which homemade adapter cables (JST XH 4-pin male → DE-9 female) are connected, terminating in DE-9 female connectors — these plug directly into the DE-9 male joystick ports on the Murmulator.
- Configuration is done via BLE Web Config (Desktop/Android Chrome), which allows flexible button mapping, turbo mode configuration, etc.

## Dev board (ESP32) selection

**Final decision: ESP32 D1 mini MH-ET LIVE (CH9102).**

**Alternatives considered:**

| Board | Verdict |
|---|---|
| ESP32-WROOM-32 DevKit, **30 pin**, CP2102 | Rejected — on this form factor the **GPIO0 pin is physically not broken out**, and it is required for BOOT/pairing and all BlueRetro reset modes (checked against the pinouts of the specific boards on hand) |
| ESP32-WROOM-32 DevKit, **38 pin**, CP2102 | Technically suitable and matches the official recommendation of the BlueRetro author (see below) — GPIO0 is broken out. Not chosen because it would require reworking the already finished and hardware-verified KiCad schematic (different footprint, different board dimensions); at the current stage (PCB close to finalization) this was deemed impractical |
| ESP32 D1 mini MH-ET LIVE, CH9102 (chosen) | The schematic and pinout are already designed and hardware-verified for this board; no conflicts between on-board peripherals and the pins used (IO2, IO4, IO5, IO17, IO18, IO19, IO22, IO32, IO0) were found |

**Board on-board LED:** on the ESP32 D1 mini MH-ET LIVE the built-in LED is connected to **GPIO2** and lights up when GPIO2 is **HIGH**. In this build GPIO2 is used for port 1 indication (see the "LED indication subsystem" section below) — when IO2 is HIGH (port 1 LED lit via MOSFET Q1), the board's built-in LED mirrors this state (lights up together with the external port 1 LED). This creates no electrical conflict (GPIO2 remains an output; the on-board LED is just an additional, somewhat redundant indication of the same signal), but keep it in mind when assembling into the enclosure — the board's built-in LED may be visible from outside if not covered.

**Official position of the BlueRetro community:** the project author (darthcloud) names a specific recommended board in the official documentation — **ESP32-DevKitC-32E/D with the ESP-WROOM-32 module** (38-pin form factor). Reasons:
- Cheap clones sometimes have on-board peripherals that conflict with pins used by BlueRetro (a documented case — an LED on IO5 on the Wemos Lolin32 board, which makes the firmware hang in a reset loop).
- The module inside must be a **WROOM** (not WROVER) — WROVER has 2 fewer GPIOs because of its built-in PSRAM.
- Third-party makers of ready-made BlueRetro kits (e.g., RetroRosetta) follow the same recommendation — use only official DevKit boards.

**Why the "built-in BOOT button" argument played no role:** in this project the BOOT and RESET buttons are brought out to the device enclosure (3D-printed box) on separate wires anyway — so whether the board itself has a BOOT button does not matter for the choice.

**Known trade-off of the chosen board — regulator heating:** the built-in **AMS1117** linear regulator (5V→3.3V) on the D1 mini heats up to ~54°C in operation. This is not unique to the D1 mini — the same AMS1117 is used on WROOM-32 DevKit boards, but the small D1 mini board has less PCB copper for heat dissipation. 54°C is not critical for the regulator itself, but it is close to the softening temperature of PLA (used for the 3D-printed enclosure) — with a tight layout, the plastic may deform during long operation without ventilation.

**Risk accepted** — the build is used as is. If heating ever becomes a problem, two options are possible:
1. A ventilation hole in the enclosure above the regulator.
2. Bypassing/desoldering the built-in AMS1117 and supplying ready 3.3V from a separate external DC-DC regulator (similar to the SD module modification in `murmulator-vga-project-summary.md`, section 2).

## BlueRetro hardware build
- A DIY build based on the **ESP32 D1 mini (MH-ET LIVE, CH9102 USB-serial chip)** is used — homemade, not the ready-made BlueRetroHW board.
- **⚠️ There is no BOOT/IO0 button on the D1 mini board.** Unlike the full-size ESP32 DevKitC, the D1 mini physically has only the **RST/EN** button (module reset) — there is no separate BOOT button. BlueRetro functions (pairing, configuration reset, factory reset — see the "BOOT button (IO0)" section below) rely on IO0, so a **separate button between IO0 and GND must be added**.
  - In theory an external pull-up resistor is not required — the ESP32 enables an internal pull-up on IO0 (a strapping pin) in hardware at startup. The button simply shorts IO0 to GND when pressed.
  - **In practice the internal pull-up on the specific module proved unstable**, so in the final KiCad schematic the external resistor **R1 = 10 kΩ (from IO0 to +3.3V) is always fitted**, as a standard component rather than an optional fallback.

### BOOT button (IO0) — BlueRetro functions
⚠️ The timings and actions below apply to the firmware version in use (`v25.04_hw1`, see the "Firmware" section). In other BlueRetro versions the hold durations and results may differ — check the BlueRetro Wiki for the specific installed version.
- Short press (< 3 sec) — normal system reset.
- Hold 3–6 sec (IO17 LED blinks slowly) — start pairing mode.
- Hold 6–10 sec (IO17 LED blinks fast) — reset to default configuration.
- Hold > 30 sec — full ESP32 factory reset (factory firmware + settings reset).
- In pairing mode, a short press stops pairing / disconnects all Bluetooth devices.

## Pinout (consistent with murmulator-vga-project-summary.md)

**Topology:** ESP32 (BlueRetro) → J1/J2 connectors (JST XH 1x04, on the BlueRetro board) → adapter cable (JST XH 4-pin male → DE-9 female) → J6/J7 connectors (DE-9 male, on the Murmulator board) → Pico GPIO.

| Signal | ESP32 pin (BlueRetro) | JST XH J1 (P1) | JST XH J2 (P2) | DE-9 Murmulator J6 (P1) | DE-9 Murmulator J7 (P2) | Pico GPIO |
|---|---|---|---|---|---|---|
| LATCH (shared) | IO32 | pin 1 | pin 1 | pin 3 | pin 3 | GP15 |
| CLOCK port 1 | IO5 | pin 2 | — | pin 4 | — | GP14 |
| CLOCK port 2 | IO18 | — | pin 2 | — | pin 4 | GP14* |
| DATA port 1 | IO19 | pin 3 | — | pin 2 | — | GP16 |
| DATA port 2 | IO22 | — | pin 3 | — | pin 2 | GP17 |
| GND | ESP32 GND | pin 4 | pin 4 | pin 8 | pin 8 | GND |
| VCC (+5V) | — | not routed | not routed | pin 6 | pin 6 | **do not connect** |

**Adapter cable wiring (JST XH 4-pin male → DE-9 female), identical for both ports:**

| JST XH pin | Signal | DE-9 pin |
|---|---|---|
| 1 | LATCH | 3 |
| 2 | CLOCK | 4 |
| 3 | DATA | 2 |
| 4 | GND | 8 |

\* **CLOCK and LATCH are generated by the RP2040 (the Murmulator acts as host/console)** — each is a single physical signal (GP14/GP15) routed to both J6 and J7 connectors simultaneously. IO5 and IO18 on the ESP32 (BlueRetro emulates the joystick/slave) are two **inputs** listening to this signal, not two independent generators. No electrical conflict (bus contention) arises from joining them, since the RP2040 is the only active driver of the line. The BlueRetro firmware internally treats IO5/IO18 as two independent CLOCK inputs (for consoles where the P1/P2 CLOCK lines are not physically joined) — when connected to the Murmulator both inputs necessarily receive the same physical signal.

VCC (pin 6, Murmulator DE-9) is intentionally left unconnected: the ESP32 inside BlueRetro runs on 3.3V logic, and the signal lines go directly to the Pico GPIO without level shifters; BlueRetro power is provided separately, through the board's own J3 connector (see the "BlueRetro power" section). The JST XH J1/J2 connectors (joystick ports on the BlueRetro side) have no +5V line at all — only 4 signals (LATCH/CLOCK/DATA/GND).

## BlueRetro power (ESP32 module)
- The ESP32 D1 mini (with a built-in AMS1117 regulator, 5V→3.3V) can be powered in two ways — separately or simultaneously:
  1. **Via J3 on the BlueRetro board** — from the Murmulator's J2 connector (a tap of "raw" +5V from the external PSU, upstream of the Murmulator's diode D1, see `murmulator-vga-project-summary.md`, section 10) → Schottky diode D1 = 1N5819 on the BlueRetro board → D1 mini 5V pin. Because of the diode drop, the D1 mini 5V pin receives less than 5V (~4.5–4.7V).
  2. **Via the micro-USB port on the D1 mini** (from a PC or a USB adapter).
- **Connecting J3 and micro-USB at the same time is acceptable.** The D1 mini has no protection diode: its 5V pin is connected directly to the micro-USB VBUS and to the input of the 3.3V regulator. Isolation is provided only by the Schottky diode D1 = 1N5819 at the J3 input: it blocks current from USB into the Murmulator PSU, and the voltage after it is below 5V, so when both are connected the load is taken by USB. Safety condition — the PSU voltage minus the drop across D1 (~0.15–0.25V at low current) must stay below the USB port voltage; otherwise the PSU will back-feed the computer's VBUS. With a PSU of ~5.0V the condition is met with margin.
- **⚠️ Limitation when powered only via J3:** BlueRetro and both joysticks work only when the external PSU is connected to the Murmulator. The YD-RP2040 has no accessible USB VBUS upstream of the BAT54C combining diode, so BlueRetro cannot be powered from the Murmulator's USB. When powered via the D1 mini micro-USB, this limitation does not apply.
- **Measured (multimeter):** peak current draw of the ESP32 on the 5V bus with the BT radio active — **~155 mA**. Power via J2 (on the Murmulator board) → J3 (on the BlueRetro board) bypasses the Murmulator's BAT54C diode and Vout node (see `murmulator-vga-project-summary.md`), so this load does not share the current budget with the PS/2 keyboard and is not limited by the BAT54C rating (~150–300 mA per leg) — the current margin is sufficient.
- The GND of the signal lines (LATCH/CLOCK/DATA) is shared with the Murmulator in any power configuration — via the J1/J2 connectors (JST XH) and the adapter cables to J6/J7.

## Input power protection, RESET, and decoupling

**External 5V connector (J3) and protection diode D1:**
- **J3** — a separate 2-pin connector ("External 5V Power Connector") through which power from the external PSU 5V bus is brought onto the board (see the "BlueRetro power" section above).
- **D1 = 1N5819** (Schottky, 40V/1A) is placed between J3 and the +5V bus: anode on the J3 side, cathode on the +5V side. Function — reverse-polarity protection / input isolation (analogous to the BAT54C diode-OR on the Murmulator itself, see `murmulator-vga-project-summary.md`, section 10).

**RESET button (SW2):**
- A separate button **SW2** that shorts the ESP32 module's **RST** to GND — a standard hardware reset of the module, independent of the BOOT/IO0 logic (see the BOOT button function list above).

**Decoupling capacitors (bulk + bypass on both power rails):**

| Rail | Bulk (electrolytic) | Bypass (ceramic) |
|---|---|---|
| +5V (input, after J3/D1) | C5 = 470 µF | C6 = 100 nF |
| +3V3 (ESP32 module regulator output) | C8 = 220 µF | C7 = 100 nF |

Both capacitors on each rail are connected in parallel between the rail and GND — the classic pair of "bulk filtering + high-frequency decoupling".

---

## LED indication subsystem (final, hardware-verified)

According to the BlueRetro Wiki (LED usage sections), indication consists of two types of LEDs: **one global-status LED on IO17** and up to **four port-status LEDs** (one per port). This build implements the global LED + two port LEDs (for both joystick ports, P1/P2).

**Note on IO17 vs the board's built-in LED:** on some ESP32 DevKitC/D1 mini boards the built-in LED sits on IO2, but in the BlueRetro firmware this pin is used for the auto-programming circuit — so indication requires a separate external LED on IO17 specifically, rather than relying on the board's built-in power/status LED.

**ESP32 pin assignment:**

| ESP32 IO | Function | LED | Connection |
|---|---|---|---|
| IO17 | Global status (adapter status / error / pairing) | D3, blue | two-resistor pull-up circuit (direct) |
| IO2 | Port 1 status | D4, green | via MOSFET Q1 (2N7000) |
| IO4 | Port 2 status | D5, green | via MOSFET Q2 (2N7000) |

IO2/IO4/IO17 are all ESP32 strapping pins. Only the port LEDs D4 (IO2) and D5 (IO4) are driven through MOSFETs — a requirement of this BlueRetro hardware revision so that the LED load does not interfere with booting; IO17 (D3) uses a separate pull-up circuit without a MOSFET (see above). Ports 3/4 (if expanded) are IO12/IO15, but in PlayStation mode they are reassigned to analog-mode indication; not relevant for NES mode.

**Behavior (confirmed by a real test):**
- Power on, no error → auto-pairing: the global LED (IO17) **pulses** + the LED of the first available port (IO2) **pulses**; the second port (IO4) is off.
- 1st gamepad connected → IO17 goes off, port 1 (IO2) **solid**.
- 2nd gamepad connected → port 2 (IO4) **solid**, IO17 stays off.
- Gamepad disconnected → its port LED goes off; when a port is freed, the adapter returns to pairing (global + first free port pulse again).

### Global-status LED circuit IO17 (two-resistor variant)

⚠️ Differs from the official single-resistor circuit in the wiki (R36 ≈ 65 Ω–1 kΩ, LED directly between the node and GND). This build uses **two resistors**, which improves behavior:

```
+3V3 --[R2 360R]--+-- IO17 (U1 pin 25)
                  |
                  +--[R3 150R]--[D3 anode >|< cathode]-- GND
```
IO17 is connected to the node between R2 and R3 (on the LED anode side), not to the GND side.

**Resistor roles:**
- **R2 = 360R** — pull-up 3.3V→IO17. Provides fail-safe behavior (in high-Z the LED glows dimly) and determines the parasitic current sunk into the GPIO at LOW. It **does not affect** the operating brightness.
- **R3 = 150R** — series current-limiting resistor for the LED. Sets the brightness in HIGH-PWM (pulsing/solid). Limits the current when actively driven push-pull HIGH.

**Logic (fail-safe by default):**
- IO17 by default (not configured, high-Z/input) → R2 pulls the node up to 3.3V → **the LED glows dimly on its own**, without any firmware command.
- After a successful start, the firmware **explicitly sets IO17 as an output LOW** → node to 0V → **the LED turns off**.
- Purpose of the inversion: if the firmware fails to boot (crash before initialization), IO17 stays high-Z and the LED **keeps glowing** = error indication. In the classic circuit such a failure would be indistinguishable from the normal "off" state.
- Pulsing/status: the firmware switches IO17 to PWM via LEDC (~200 µs, duty 0–20%), creating a "breathing" effect. Confirmed with an oscilloscope.

### Port-status LED circuit (IO2/IO4, via MOSFET)

```
+3V3 --[D4/D5 anode >|< cathode]--[R4/R5 510R]-- drain Q1/Q2
                                                 gate  -- IO2/IO4
                                                 source-- GND
```
With IO2/IO4 HIGH → MOSFET on → LED lit. With the MOSFET off, no current flows at all — there is **no** "current sunk into the GPIO" issue here (unlike IO17), so the port LEDs can run at operating current without regard to GPIO load.

### Final values and currents (measured Vf)

Vf was measured with a multimeter (voltmeter) across the LED through 510R to 3.3V (high input impedance → true operating point): **blue Vf = 2.6 V @ ~1.4 mA, green Vf = 2.4 V @ ~1.76 mA.**

| LED | Color | Vf | Circuit | Current | State |
|---|---|---|---|---|---|
| D3 (IO17) | blue | 2.6 V | R3 = 150R | ~4.0–4.5 mA | HIGH-PWM (pulsing/status) |
| D3 (IO17) | blue | 2.6 V | R2 = 360R | ~9.2 mA | LOW — current sunk into GPIO (depends only on R2) |
| D3 (IO17) | blue | 2.6 V | R2+R3 = 510R | ~1.37 mA | high-Z — fail-safe (start-up error) |
| D4/D5 (ports) | green | 2.4 V | R4/R5 = 510R | ~1.76 mA | solid (port connected) |

**Brightness balance:** the green port LEDs run at a lower current (~1.76 mA) than the blue status LED (~4.5 mA), but because the eye is most sensitive to green (~555 nm) and weakly sensitive to blue, they look comparable. R4/R5 = 510R — with a lower value (150R) the green would become brighter than the blue, breaking the balance.

**Visual check:** when blue (IO17) and green (port 1) are lit at the same time during pairing, their brightness is comparable. If needed, R4/R5 can be lowered to ~330R (~2.7 mA) — but not lower.

**Voltage headroom:** the blue LED has little headroom (3.3 − 2.6 = 0.7 V), so its brightness is sensitive to dips on the 3.3V rail — at 3.2 V the HIGH current drops to ~4 mA, which is still fine.

## Oscilloscope signal verification (confirmed)

📄 The full set of oscillograms (LATCH, "no button pressed", and pressing each of the 8 Xbox One gamepad buttons in turn), with a description of the oscilloscope connection and settings, is in the document **`blueretro-xbox-one-nes-oscillograms_EN.docx`** (the original screenshots are in the `XBox One as NES Joystick Debugging` folder).

**CLOCK (GP14 → IO5/IO18):** period 80 µs (≈12.5 kHz), 50% duty cycle (HIGH = LOW = 40 µs), amplitude 0V ↔ 3.3V, clean edges.

**LATCH (GP15 → IO32):** repetition period 1200 µs (≈833 Hz), pulse width 40 µs, amplitude 0V ↔ 3.3V.

- The LATCH pulse width (40 µs) equals one CLOCK phase — typical for latching the parallel-loaded button state before a series of shift clocks.
- One LATCH period fits 1200 / 80 = 15 full CLOCK cycles — enough to read the 8-bit register with margin.
- Polling rate ~833 Hz (confirmed with an oscilloscope) — noticeably higher than the classic NES/Famicom (~60 Hz, synchronized with the frame).
- The 0V/3.3V amplitude on both lines confirms correct operation of the "one driver (Pico) — receivers (ESP32)" topology, without bus contention.

**DATA (IO19/IO22 → GP16/GP17):** when buttons are pressed on the Xbox One gamepad (A, X, View, Menu, D-pad Up/Down/Left/Right), a **LOW (~0V) pulse 80 µs long** is seen on the DATA line — exactly one CLOCK period.
**The first button is output without a clock pulse:** as soon as the Murmulator asserts LATCH, the chip immediately places the value of button A on the DATA line (this is bit 0). The following buttons (X, View, Menu, etc.) are output one by one on each subsequent CLOCK pulse.

- The protocol uses **inverted logic**: LOW = button pressed, HIGH (3.3V) = button released — the standard convention for an NES/Dendy-compatible shift register.
- The pulse width (80 µs) exactly matches the CLOCK period — confirming that each bit is placed on DATA for exactly one shift clock, synchronously and without phase shift.
- The correspondence "physical Xbox button pressed → LOW pulse in the expected clock slot" confirms the correct BlueRetro mapping (Xbox One HID → NES bits) and correct reading by the Murmulator on the RP2040 side.

**DATA bit order in the NES shift register and Xbox One mapping (confirmed):**

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

The bit order **A, B, Select, Start, Up, Down, Left, Right** is the standard, canonical bit output order of the original NES controller (the same order in which the NES console historically read the 4021 register). The match confirms that BlueRetro in `GamePad`/NES mode emulates the classic protocol one-to-one, without changing the bit order, and that the Murmulator reads them in the same order.

The verification confirms both the electrical integrity of the CLOCK/LATCH signals and the correct phase alignment of DATA relative to CLOCK — data is sampled at the right point of the clock cycle.

## Firmware (verified, working configuration)

**Firmware version:** `v25.04_hw1.zip` (archive from [GitHub Releases](https://github.com/darthcloud/BlueRetro/releases)) — the HW1 specification is used (see the "BlueRetro hardware build" section).

**Files from the archive:**
- `bootloader.bin`
- `partition-table.bin`
- `ota_data_initial.bin`
- `BlueRetro_hw1_nes.bin` (system-specific firmware for NES)

**Flashing command (Windows, via PlatformIO Core):**

```
pio pkg exec -- esptool.py -p COM6 -b 460800 --before default_reset --after hard_reset --chip esp32 write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x1000 bootloader.bin 0x8000 partition-table.bin 0xD000 ota_data_initial.bin 0x10000 BlueRetro_hw1_nes.bin
```

Port `COM6` is for a specific board instance — check it in Windows Device Manager each time you flash.

**Note:** when calling `python esptool.py ...` directly (not via `pio pkg exec`), esptool dependencies may be missing, because the system Python does not see the environment where PlatformIO installed them — the `pio pkg exec --` command avoids this problem by using the correct environment automatically.

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

After flashing, configuration via Web Config may be needed — in this case the build worked with the default settings; the configuration below is given for verification/reference.

**Address:** [https://blueretro.io/](https://blueretro.io/)

⚠️ BlueRetro configuration is available only when **no controller is connected** — this is intentional, so as not to take airtime away from the gamepads during play.

**Procedure:**
1. Open **"BlueRetro Advance config"**.
2. Click **[Connect BlueRetro]** — repeat the click if needed until the connection is established.
3. A pop-up window with a list of Bluetooth devices opens — select the BlueRetro device and click **[Pair]**.
4. **Global Config:**
   - System: `NES`
   - Multitap: `None`
5. **Output Config:**
   - Output 1 → Mode: `GamePad`, Accessories: `None`
   - Output 2 → Mode: `GamePad`, Accessories: `None`
6. **[Save]**

## Pairing and disconnecting gamepads

Practical instructions for pairing and disconnecting/unpairing Bluetooth gamepads are in a separate document, **`blueretro-murmulator-user-guide_EN.md`**.

## Open questions (require clarification/verification)
*(none at the moment)*
