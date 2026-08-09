# BlueRetro + Murmulator — User Guide (Pairing / Unpairing)

A step-by-step guide for everyday use: how to connect a Bluetooth gamepad to BlueRetro on the Murmulator, how to disconnect it, and how to fully forget a pairing (unverified). All the technical parts of the project — pinout, electrical schematic, firmware, and build — are covered in `blueretro-murmulator-project-summary.md`; this document covers only the user-facing interaction.

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
   - Power on BlueRetro — if no controllers are connected, it will enter inquiry mode on its own.
   - In this state, **two LEDs pulse**: the global LED (IO17) and the LED of the first free port (port 1, IO2) — this confirms active searching. The second port (IO4) is off at this time.
   - If no LED is pulsing (e.g., Manual mode is enabled, or pairing was already cancelled) — hold BOOT/IO0 for **3–6 seconds** to enable search manually.
2. **Put the Xbox One gamepad into pairing mode:**
   - Turn on the gamepad with the **Xbox** button (in the center).
   - Press and hold the black **Sync** button until the Xbox logo on the gamepad starts blinking.
3. **Completing pairing:**
   - The logo on the gamepad stops blinking — pairing is complete.
   - The global LED (IO17) **turns off**, and the LED of the occupied port (port 1, IO2) switches from pulsing to **solid** — this is the indication of a successful connection to a specific port.
4. **Stick initialization:** press the **A** button a few times — needed for correct calibration of the analog stick's center value.
5. **Reconnecting:** re-pairing is not required — a short press of the Xbox button on the gamepad automatically reconnects it to BlueRetro.

**Second gamepad:** after the first controller connects, the global LED (IO17) is off, so for the second gamepad you need to manually enable search (hold BOOT/IO0 for 3–6 sec, see above) — then the port 2 LED (IO4) will start pulsing and switch to solid once pairing completes.

**Configuration via Web Config:** after pairing, in BLE Web Config (blueretro.io or a local config) make sure the output config of the relevant port (#1 or #2) is set to **GamePad** mode (not Dual/Four Score).

**The DATA bit ↔ Xbox One button mapping has been confirmed with an oscilloscope** — see the "Oscilloscope signal verification" section in `blueretro-murmulator-project-summary.md` (A→A, X→B, View→Select, Menu→Start, D-pad→Up/Down/Left/Right).

---

## 2. Disconnecting / removing a pairing (Unpairing)

Two different scenarios — simply disconnecting the gamepad now, or fully forgetting it (deleting the pairing key).

### 2.1. Quick disconnect (disconnect, without removing the pairing)

A short press of the BOOT/IO0 button **outside of inquiry mode** — disconnects all Bluetooth devices from the adapter.

⚠️ This breaks the current connection but does not delete the pairing key — the gamepad remains stored in BlueRetro's memory and can reconnect automatically next time (just by pressing the Xbox button), without re-pairing.

**After a gamepad disconnects,** the LED of the freed port turns off, and the adapter returns to search mode on that port on its own: the global LED (IO17) and the freed port's LED start pulsing together again — just like at first power-on.

### 2.2. Fully removing pairing keys (true unpair, unverified)

Needed if the gamepad will no longer be used with this adapter, or to free up a slot (up to 16 classic BT keys and 16 BLE keys are available in total).

**Via the BOOT/IO0 button** — hold for **6–10 seconds** (LED blinks fast):
- Resets the configuration to default **and clears all saved BT pairing keys**.

**Via Web Config:** on the **System manager** page (https://blueretro.io/) there's a **Factory Reset** function, which also erases saved pairings.

⚠️ **Important:** both methods reset **all** saved controller pairings at once — there is no way to remove a single specific device from BlueRetro's list.

### 2.3. BOOT/IO0 button summary table

| Action | Press duration | Result |
|---|---|---|
| Normal reset | < 3 sec | System reset |
| Enter pairing mode | 3–6 sec (LED blinks slowly) | Starts inquiry mode |
| Configuration reset + clear pairing keys | 6–10 sec (LED blinks fast) | Configuration reset + unpair all devices |
| ESP32 factory reset | > 30 sec | Full firmware and settings reset |
| Short press in pairing mode | — | Stop pairing / disconnect all BT devices |
