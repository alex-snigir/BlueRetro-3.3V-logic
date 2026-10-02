# BlueRetro + Murmulator — User Guide (Pairing / Unpairing)

**Document version:** 1.1 · **Date:** 2026-10-02

A step-by-step guide for everyday use: how to connect a Bluetooth gamepad to BlueRetro on the Murmulator, how to disconnect it, how to fully forget a pairing (unverified), and which gamepads are compatible. All the technical parts of the project — pinout, electrical schematic, firmware, and build — are covered in `blueretro-murmulator-project-summary_EN.md`; this document covers only the user-facing interaction.

---

## 1. Pairing a Bluetooth gamepad (verified, Xbox One)

**Controller compatibility:** only Xbox One gamepad models with a **Sync** button (the black round button on top, next to the USB port) can pair via Bluetooth — typically model **1708** and newer ("Controller S"). The older model **1537** only works over proprietary Xbox Wireless (no Bluetooth) and will not pair with BlueRetro.

**Inquiry (search) mode — default behavior (Auto):** if no controller is connected, BlueRetro **automatically** enters inquiry mode as soon as it powers on — pressing the BOOT/IO0 button for the first pairing is **not required**.

The BOOT/IO0 button (held 3–6 sec) is only needed in the following cases:
- pairing has already finished/timed out and needs to be re-enabled;
- a **second** controller needs to be paired while the first is already connected (pairing on the first port turns off after a successful connection);
- Inquiry mode has been switched from **Auto** to **Manual** in BLE Web Config — in that case auto-entry into pairing at startup is disabled, and it must be enabled manually each time with the same button.

**Pairing procedure (first connection, Auto mode — default):**

1. **Make sure BlueRetro is in search mode:**
   - Power on BlueRetro — with no controllers connected, it enters inquiry mode by itself.
   - In this state **two LEDs pulse**: the global LED (IO17) and the LED of the first free port (port 1, IO2) — this confirms that the search is active. The second port (IO4) is off.
   - If no LED is pulsing (for example, Manual mode is enabled or pairing was already cancelled), hold BOOT/IO0 for **3–6 seconds** to start the search manually.
2. **Put the Xbox One gamepad into pairing mode:**
   - Turn the gamepad on with the **Xbox** button (in the center).
   - Press and hold the black **Sync** button until the Xbox logo on the gamepad starts blinking.
3. **Completing the pairing:**
   - The logo on the gamepad stops blinking — pairing is complete.
   - The global LED (IO17) **turns off**, and the LED of the occupied port (port 1, IO2) changes from pulsing to **steady (solid)** — this indicates a successful connection to that specific port.
4. **Stick initialization:** press the **A** button several times — this is needed to calibrate the center value of the analog stick correctly.
5. **Reconnecting:** no need to pair again — a short press of the Xbox button on the gamepad reconnects it to BlueRetro automatically.

**Second gamepad:** after the first controller connects, the global LED (IO17) is off, so for the second gamepad the search must be enabled manually (hold BOOT/IO0 for 3–6 sec, see above) — the port 2 LED (IO4) then starts pulsing and, once pairing completes, also changes to steady.

**Web Config setup:** after pairing, check in BLE Web Config (blueretro.io or the local config) that the output config of the relevant port (#1 or #2) is set to **GamePad** mode (not Dual/Four Score).

**The DATA bit ↔ Xbox One button mapping is confirmed with an oscilloscope** — see the "Oscilloscope signal verification" section in `blueretro-murmulator-project-summary_EN.md` (A→A, X→B, View→Select, Menu→Start, D-pad→Up/Down/Left/Right).

---

## 2. Disconnecting / removing a pairing (Unpairing)

Two different scenarios — just disconnect the gamepad now, or forget it completely (delete the pairing key).

### 2.1. Quick disconnect (without removing the pairing)

A short press of the BOOT/IO0 button **outside of inquiry mode** disconnects all Bluetooth devices from the adapter.

⚠️ This breaks the current connection but does not delete the pairing key — the gamepad stays in BlueRetro's memory and can reconnect automatically next time (just by pressing the Xbox button), without pairing again.

**After the gamepad disconnects**, the LED of the freed port turns off, and the adapter returns to search mode on that port by itself: the global LED (IO17) and the LED of the freed port start pulsing together again — just like at first power-on.

### 2.2. Fully deleting pairing keys (true unpair, unverified)

Needed if the gamepad will no longer be used with this adapter, or to free a slot (up to 16 classic BT keys and 16 BLE keys are available in total).

**Via the BOOT/IO0 button** — hold for **6–10 seconds** (LED blinks fast):
- Resets the configuration to default **and clears all stored BT pairing keys**.

**Via Web Config:** the **System manager** page (https://blueretro.io/) offers a **Factory Reset** function, which also erases the stored pairings.

⚠️ **Important:** both methods reset **all** stored controller pairings at once — BlueRetro does not support removing one specific device from the list.

### 2.3. BOOT/IO0 button summary

| Action | Press duration | Result |
|---|---|---|
| Normal reset | < 3 sec | System reset |
| Enter pairing mode | 3–6 sec (LED blinks slowly) | Starts inquiry mode |
| Configuration reset + clear pairing keys | 6–10 sec (LED blinks fast) | Configuration reset + unpair all devices |
| ESP32 factory reset | > 30 sec | Full firmware and settings reset |
| Short press in pairing mode | — | Stop pairing / disconnect all BT devices |

---

## 3. Gamepad compatibility

Verified on firmware `v25.04 hw1 nes`.

| Gamepad | Protocol | Result |
|---|---|---|
| Xbox One (model 1708 and newer, with the Sync button) | Classic BT | Works (confirmed) — see section 1 |
| No-name GamePadPlus V3 (AliExpress), **X + Home** mode | BLE | Works (confirmed) |
| No-name GamePadPlus V3, other modes (A / B / Y + Home) | BLE / Classic BT | Connects, but buttons are not transmitted (confirmed) |

### 3.1. No-name GamePadPlus V3 gamepad

The gamepad has four connection modes, selected by a button combination at power-on. **Only the X + Home mode** works with BlueRetro.

**Connecting (first time):**

1. Make sure BlueRetro is in search mode (IO17 and the free port LED are pulsing — see section 1).
2. With the gamepad **turned off**, press and hold the **X** button and, without releasing it, press **Home** (the center button).
3. The gamepad pairs with BlueRetro: the global LED (IO17) turns off, and the port LED changes to steady.

**Reconnecting:** the gamepad remembers the last selected mode — just press **Home** (confirmed).

**If the gamepad has connected but the buttons do not work** (the port LED is steady, but there is no signal on the DATA line): the gamepad is in a different mode — most often the factory A + Home mode. Turn the gamepad off and turn it on again with the **X + Home** combination.

BlueRetro sees each mode as a separate device (with its own Bluetooth address), so after a mode change the gamepad pairs again and occupies a separate pairing key.

**Gamepad modes and reasons for incompatibility** (from the ESP32 log, see section 4):

| Combination | Protocol | Identifies itself as (VID:PID) | Result with BlueRetro |
|---|---|---|---|
| A + Home (factory) | BLE | `1949:0402` | HID descriptor is not parsed (`Unknown HID marker: 36`) — no data |
| B + Home | Classic BT | Sony DualShock 4 `054C:09CC` | The PlayStation driver is enabled, the clone does not respond like a real DS4 — no data |
| **X + Home** | BLE | `5803:BDF8` (generic HID gamepad) | **Works** |
| Y + Home | BLE | `1603:FCF1` (HID mouse) | Mouse mode: no gamepad sticks or buttons — not applicable to NES |

**Sign of a working mode in the log:** after connecting, an input report layout line appears, for example:

```
4 I 0130 0 8 0131 8 8 0132 16 8 0135 24 8 0139 32 4 0901 40 16 02C4 56 8 02C5 64 8 rtype: 2
```

If instead there is `Unknown HID marker`, or the line contains only mouse buttons and X/Y axes, the mode is not suitable.

---

## 4. Getting the connection log from the ESP32

The BlueRetro serial port log shows how the gamepad connects and is recognized. It can be used to find out why a connected gamepad does not transmit buttons (see section 3.1).

1. **Connect to the Serial Terminal.** Connect the ESP32 to the computer via USB and open the port in SecureCRT or with `pio device monitor`. The connection parameters (port, baud rate, SecureCRT settings) are given in the "Firmware → Serial Monitor Debug" section of `blueretro-murmulator-project-summary_EN.md`.
2. **Restart the ESP32** with the RESET button (SW2), so the log starts from boot.
3. **Connect the gamepad** — turn it on and pair it if needed (section 1 or 3.1).
4. **Watch the log in the terminal window.** Connection, gamepad parsing, and disconnection are printed line by line. To save the log for analysis, copy the terminal output into a text file.

⚠️ Unreadable characters at the beginning of the output, right after boot (while the Bluetooth radio initializes), are normal behavior, not a firmware fault.

**What to look for in the log:**

| Log line | Meaning |
|---|---|
| `LE ADV ... bdaddr: ...` | The gamepad connects via BLE |
| `Page ...` / `Inquiry ... bdaddr: ...` | The gamepad connects via Classic BT |
| `dev: 0 type: ... <name>` | The name reported by the gamepad |
| `VID: 0x.... PID: 0x....` | The identifier the gamepad presents itself as |
| `1 I 0130 0 16 0131 16 16 ...` | The input report layout has been built — buttons will be transmitted |
| `Unknown HID marker: ..` | BlueRetro could not parse the gamepad description — buttons will not be transmitted |
| `DISCONN from dev: 0` | The gamepad has disconnected |
