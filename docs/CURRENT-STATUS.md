# Haswing Helm: where the project stands

**Current firmware: 0.1.10**

**Status: installed and bench-tested on the touchscreen.**

Haswing Helm is a dedicated, offline touchscreen remote for a Haswing Cayman
trolling motor. It uses the motor's wired RS-485 connection for steering,
propulsion commands and live feedback. The motor's RF fob still works.

The immediate goal is a dependable manual remote with clear feedback and optional
Prop Guard. Boat/motor heading sensors and automatic navigation are the next
phase, not features claimed by the current touchscreen firmware.

## At a glance

| Item | Current setup |
| --- | --- |
| Touchscreen | Waveshare **ESP32-S3-Touch-LCD-7, revision 1.2** |
| Firmware | Standalone **ESP-IDF / LVGL**, not ESPHome |
| Display | 800 × 480, landscape |
| Motor connection | RS-485, 9600 baud, 8 data bits, no parity, 1 stop bit |
| Motor feedback | Propeller ON/OFF, speed setting 0–10, battery level 0–4 |
| Normal refresh | Approximately every two seconds while idle; incoming reports can update sooner |
| Network | No phone, Wi-Fi or internet required for touchscreen operation |
| Prop Guard | Optional, switched on from the gear menu; **off after boot** |
| Download | [Touchscreen firmware and update instructions](../firmware/README.md) |

The battery bar is the motor's coarse **0–4 level**, not measured voltage or an
accurate percentage. The speed number is a setting, not RPM or boat speed.

## Hardware and tested wiring

These connections are for the tested motor cable and **revision 1.2** panel.
Do not assume other cable colours or board revisions are identical.

| Motor cable | Touchscreen RS-485 terminal |
| --- | --- |
| Green | **A** |
| White | **B** |
| Black | **GND** |

- The panel uses **GPIO15 for RX** and **GPIO16 for TX / auto-direction control**.
  This is the verified revision 1.2 mapping.
- The RS-485 termination selector is set to **NC** in this setup.
- The panel is powered separately through USB, not from the motor's accessory supply.
- The onboard RS-485 transceiver is not galvanically isolated.

For connecting or disconnecting the motor cable, turn motor power off and unplug
panel USB first. After reconnecting, power the **panel first, then the motor**.

## Using the remote

### 1. Read the status

The header shows **PROP ON**, **PROP OFF**, or an explicit unknown/uncertain state.
A green status dot means feedback is current and valid. Stale feedback or a
communication problem is shown rather than silently assumed to mean OFF.

**A fresh motor-reported OFF is the project's verified OFF indication.**

### 2. Enable controls

Press **Enable controls**. The center padlock changes to the propeller graphic.

| Control | What it does |
| --- | --- |
| Left arrow | One bounded left steering pulse per tap |
| Right arrow | One bounded right steering pulse per tap |
| Center propeller | Switch propulsion; with Guard enabled, use fresh feedback to choose ON or verified Stop |
| Slower | Reduce the speed setting by one step |
| Faster | Increase the speed setting by one step |
| Stop & disable | Lock manual commands, stop propulsion if needed, then verify OFF |

There is one combined bottom button: **Enable controls** when locked, and
**Stop & disable** when enabled. Commands are not queued or repeatedly sent by
holding a button.

### 3. Stop

Press **Stop & disable**. The controller checks motor state, sends a toggle only
if a fresh query says ON, then queries again to verify OFF. If already OFF, it
does not toggle the propeller.

The padlock returns when controls are locked. **The padlock means manual controls
are locked; it does not itself mean the motor is OFF. Always read PROP status.**

### What the artwork tells you

- **Padlock:** manual controls are locked.
- **Propeller:** controls are enabled.
- **Spinning propeller:** enabled controls with ON feedback. It is a visual cue,
  not an RPM display.
- **Stationary propeller:** OFF or feedback that no longer supports an ON animation;
  the text explains the actual state.

Routine background queries do not flash NO FEEDBACK or interrupt the animation
while a normal reply is pending. Real timeouts and faults remain visible.

## Prop Guard

Prop Guard maintains the **touchscreen's requested OFF state**. It does not disable
the fob and it does not block deliberate touchscreen ON commands.

### Start watching

1. Open the **gear menu** and switch **Prop Guard ON**.
2. Return to the main screen.
3. Press **Enable controls**, then **Stop & disable**.
4. Confirm **OFF VERIFIED** and **GUARD WATCHING**.

If the touchscreen already has an explicit OFF request established, Guard can use
that intent. Otherwise **GUARD READY** means it is enabled but waiting for you to
request OFF. Guard settings and intent are kept in RAM and reset on reboot.

### If the fob starts propulsion

While Guard is watching OFF, a fresh query reporting ON starts another verified
Stop sequence. Once OFF is verified, Guard resumes watching. Repeated fob starts
have been bench-tested and corrected repeatedly.

### If the touchscreen starts propulsion

A deliberate touchscreen ON request is allowed. **GUARD — LOCAL ON** indicates
that this is intentional, so Guard does not fight the request. Pressing
**Stop & disable** establishes OFF intent again and returns to watching.

Guard never automatically starts propulsion to restore ON after another controller
turns it OFF. It compares reported state with local intent; motor replies do not
identify which controller issued a command.

### Guard status guide

| Display | Meaning |
| --- | --- |
| GUARD OFF | Guard is disabled |
| GUARD READY | Enabled; waiting for an explicit OFF request |
| GUARD WATCHING | Monitoring the requested OFF state |
| GUARD — LOCAL ON | A deliberate touchscreen ON request is allowed |
| GUARD CORRECTING | A Stop-and-verify sequence is running |
| GUARD WAITING | Stop is unconfirmed; status queries continue |
| GUARD — NO FEEDBACK | Fresh feedback has not returned within the normal window |
| GUARD FAULT | A transport/control fault requires attention and reboot |

### Continued monitoring: Mode A

If a Stop cannot yet be confirmed, Guard keeps making paced status queries:

- A **new fresh ON reply** permits a new conditional Stop-and-verify attempt.
- A **new OFF reply** verifies OFF without another toggle.
- **No reply** means queries only, not blind toggle retries.

Actual UART/driver faults remain locked until reboot. Switching Guard off/on does
not erase an unresolved state or blindly enable manual controls.

## Gear menu and diagnostics

- **Prop Guard:** enable or disable the optional mode.
- **Diagnostics:** current bus information and an explicit status query.
- **TX / RX history:** a bounded, timestamped RAM log of commands and replies.
- **Simulation:** an isolated demonstration; its controls do not command the motor.

Opening a menu locks manual controls but **does not stop propulsion**. Use
**Stop & disable** first if you want the motor stopped. Background monitoring and
Guard continue while other pages are open.

Normal status queries are silent. Active button commands make the motor beep.
A manual Slower action has been checked with propulsion OFF: it updates feedback
and changes the speed setting without starting the propeller. There is **no
automatic speed-changing nudge or propulsion blip** in the current firmware.

## What is confirmed now

| Bench check | Result |
| --- | --- |
| Steering, speed changes and propulsion switching | Working |
| Stop from ON and confirmation of already-OFF | Verified |
| Repeated fob starts while Guard watches OFF | Corrected repeatedly |
| Intentional touchscreen ON with Guard enabled | Allowed |
| Stop returning Guard to watching | Verified |
| Padlock / propeller transition | Working |
| Propeller animation and quiet background refresh | Working on the panel |
| Incoming status changes from fob commands | Display updates promptly |

Automated native tests also cover the parser, command gates, worker/receiver,
Guard decisions, query-driven recovery, display deadlines and button behaviour.

### Checks still to do

- Longer-duration bench soak testing.
- Missing-feedback recovery on actual hardware.
- Readability in direct sunlight and touch operation on the boat.
- Boat power stability and on-water operation.

The accepted physical fallback is disconnecting **main trolling-motor power**.
Polling is not instantaneous cutoff, and the fob can still issue commands between
queries. If the controller or motor communication stops responding while running,
cut main power rather than rely on the screen to stop it.

## Separate projects stay separate

The **classic ESP32 pedal/sensor project** and this **ESP32-S3 touchscreen project**
are independent. Do not interchange their firmware. The touchscreen does not
currently consume the other project's compass/GPS readings.

The repository's root Python driver remains the original protocol reference.
The touchscreen binary is a separate ESP-IDF/LVGL application; its application
source and private build configuration are not included in this documentation/
firmware upload. The separate pedal firmware is not published here.

## Next development phase

The next milestone is a **separate motor-head ESP32 and second compass**. Those
additional parts have **not been purchased yet**. No automatic heading control
or spot-lock is implemented in the current touchscreen firmware.

The planned sequence is:

1. Select the second ESP32 and compass hardware.
2. Build a read-only motor-head sensor project, separate from the working projects.
3. Establish the boat compass orientation and heading calibration.
4. Align the motor straight ahead and establish the motor sensor's reference.
5. Compare boat and motor headings while steering; their calibrated difference
   gives motor angle relative to the boat.
6. Send those readings to the touchscreen over **ESP-NOW**.
7. Validate the readings on the boat before using them for automatic steering.

For now, the working remote and Guard are the stable base to preserve.
