# ESP-Files

MicroPython code for the ESP32 in each detector box. The ESP32 reads timestamps for detector pulses from the u-blox F9 TIM GPS over UART (UBX TIM-TM messages), buffers them, and talks to the DAQ server over Wi-Fi/TCP (port 12345).

## Deploying to a board

1. Flash MicroPython first ([Flashing-ESP](../../Flashing-ESP/README.md)).
2. Copy every file in `Main Accessories/` to the ESP's root filesystem under the same names.
3. Copy **one** file from `Main/`, matching the box's role, and rename it to **`main.py`** on the ESP.
4. In that file, set `ssid`, `password`, and `HOST` (the server's IP). They are placeholders in the repo. `PORT` is `12345`.
5. Surface boxes only: create `config.txt` on the ESP containing the detector number (e.g. `3`).

Use Thonny, `mpremote cp`, or the GUI's OTA tab to upload.

## Main/ (pick one, upload as main.py)

| File | Role |
|---|---|
| `Borehole_main.py` | **Trigger station.** Streams every ch0/ch1 rising and falling edge (inst 99) to the server. Its ch0 rising edges become the event triggers that the server broadcasts to the surface boxes. |
| `Surface_main.py` | **Surface detector.** Keeps GPS timestamps in a ring buffer. On a 5-int trigger from the server, it sends back the hits within `REQUESTED_TIME_WINDOW` (1 ms) of the trigger time. Also sends rate/stat packets (inst 94–98), reads `config.txt` for its detector number, and runs the OTA listener. |
| `Veto_main.py` | **Veto detector.** Almost the same as Borehole_main. The server gives veto events negative event numbers. |

## Main Accessories/ (upload all)

| File | Purpose |
|---|---|
| `boot.py` | Runs before `main.py`. Forces one deep-sleep reset after a cold boot to get clean RAM, then sets the CPU to 240 MHz, starts a 20 s watchdog (`wdt`), and opens `uart1` to the GPS (460800 baud, TX=12, RX=14, 24 KB RX buffer). The main files use these globals directly. |
| `wifi.py` | `con_to_wifi(ssid, password)` connects with power saving off and resets the board after repeated failures. |
| `ringBuffer.py` | Preallocated ring buffers for raw and calibration timestamps. |
| `PPS.py` | Experimental PLL that disciplines the RTC to the GPS PPS (GPIO27). Its import is currently commented out in every main file. |
| `ota_listener.py` | HTTP listener on port 8080, run in a thread. Endpoints: `/prepare` (stop DAQ before an update), `/version` and `/version/<file>`, `/ota` (replace `main.py`), and `/ota/<file>` (any other `.py`/`.txt`). |

## Device ID

Each box identifies itself by the last byte of its MAC (`wlan.config('mac')[-1]`). The server and GUI expect BH = 128 and Veto = 16. Surface detectors are listed in `DEVICE_IDS` in `Software/Server/ServerControlGUI.py`.

## Known gaps

- `ota_listener.py` imports `ota_update`, which is not in this repo.
- `Borehole_main.py` and `Veto_main.py` call `wifi.con_to_wifi(ssid, password, max_retries=10)` on reconnect, but `wifi.py`'s `con_to_wifi` doesn't accept `max_retries`. That reconnect path raises a `TypeError`.
