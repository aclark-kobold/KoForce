# Flashing-ESP

Firmware and instructions for putting MicroPython on a new ESP32, and a helper that reads the board's MAC address.

## Files

| File | Purpose |
|---|---|
| `ESP32_GENERIC-20250415-v1.25.0.bin` | MicroPython v1.25.0 for the classic ESP32 |
| `ESP32_GENERIC_S3-SPIRAM_OCT-20260406-v1.28.0.bin` | MicroPython v1.28.0 for ESP32-S3 boards with octal SPIRAM |
| `Flashing ESP32.txt` | Short flashing notes |
| `ESP_Mac_ID.py` | Prints the ESP32's MAC address and its last byte (the device ID) |

Firmware downloads: https://micropython.org/download/ESP32_GENERIC/

## Flashing

```
pip install esptool

# Find the port (e.g. COM3 on Windows, /dev/ttyUSB0 on Linux)
esptool --port <PORT> erase-flash

# Classic ESP32: write at offset 0x1000
esptool --port <PORT> --baud 460800 write-flash 0x1000 ESP32_GENERIC-20250415-v1.25.0.bin

# ESP32-S3: write at offset 0x0
esptool --port <PORT> --baud 460800 write-flash 0 ESP32_GENERIC_S3-SPIRAM_OCT-20260406-v1.28.0.bin
```

## Getting the device ID

Run `ESP_Mac_ID.py` on the board, e.g. through Thonny or `mpremote run ESP_Mac_ID.py`. It prints the full MAC and `mac[-1]`.

- `mac[-1]`, the last MAC byte, is the **device ID** used by the server, GUI (`DEVICE_IDS`), and analysis scripts. Record it for each box.
- On a network that needs MAC registration (e.g. the campus network), register the full MAC before the board can connect.

After flashing, upload the detector code from [Software/ESP-Files](../Software/ESP-Files/README.md).
