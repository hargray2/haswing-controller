# Touchscreen firmware — 0.1.10

**For Waveshare ESP32-S3-Touch-LCD-7 revision 1.2 only.**

| File | Purpose |
| --- | --- |
| [haswing_touchscreen-0.1.10.bin](haswing_touchscreen-0.1.10.bin) | ESP32-S3 touchscreen application; **816736 bytes** |
| [SHA256SUMS](SHA256SUMS) | Download integrity check |

Current features and everyday use: **[Haswing Helm setup guide](../docs/CURRENT-STATUS.md)**.

This is an **application-only update** for a panel already running this firmware
family with the matching bootloader/partition layout. It is not a complete
first-install/factory image. It does not include a bootloader, partition table,
NVS image or private configuration.

**Do not flash this to a classic ESP32 pedal, a different display, or an
unprovisioned panel. Do not erase flash or change partitions for this update.**

## 1. Download — Windows PowerShell

```powershell
curl.exe -L "https://raw.githubusercontent.com/hargray2/haswing-controller/master/firmware/haswing_touchscreen-0.1.10.bin" -o "$env:USERPROFILE\Downloads\haswing_touchscreen-0.1.10.bin"
```

## 2. Check the download

```powershell
Get-FileHash "$env:USERPROFILE\Downloads\haswing_touchscreen-0.1.10.bin" -Algorithm SHA256
```

Expected SHA-256 (letter case does not matter):

```text
555844c80282132db404312055a851439d689180799d8d4932763721dd0e9fbf
```

If it differs, do not flash the file.

## 3. Prepare the panel

1. Turn **motor power OFF**.
2. Disconnect the panel's **RS-485 motor cable**.
3. Leave panel USB connected to the computer.
4. Confirm the panel's serial port. `COM14` below is the example port for the
   tested panel; use your panel's actual port, not another ESP32's.

## 4. Flash the application only

Use a working Python/esptool installation (the tested setup uses esptool 5.6.0):

```powershell
python -m esptool --chip esp32s3 --port COM14 --baud 460800 write-flash 0x10000 "$env:USERPROFILE\Downloads\haswing_touchscreen-0.1.10.bin"
```

Only the application is written at **0x10000**. Existing NVS and partitions are
preserved; no erase operation is part of this procedure.

## 5. Reconnect and check

With motor power still off, unplug panel USB and reconnect the tested RS-485
wiring: **green → A, white → B, black → GND**. Then power **panel first, motor second**.

Confirm PROP OFF, enable controls to check the propeller graphic, then use
Stop & disable to confirm OFF VERIFIED and the padlock. Prop Guard defaults OFF
after boot; enable it through the gear menu when wanted.
