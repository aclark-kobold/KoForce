# Airshower-Array

Data acquisition and analysis software for the KoForce cosmic-ray air-shower / borehole detector array.

Each detector box has an ESP32 running MicroPython and a u-blox F9 TIM GPS timing receiver. The ESP32 timestamps detector pulses with the GPS and streams them over Wi-Fi/TCP to a central server. The server writes the data to disk. Analysis scripts then read the files back.

```
 Detector + u-blox F9 TIM ──UART──> ESP32 (MicroPython)
                                       │  TCP :12345 (data / triggers)
                                       ▼
                               GPS_DAQ_Server.py ──> GPS Data/Run_*/gps_daq_*.txt ──> Analysis/
                                       ▲
                     TCP :12347 (GPS control)
                                       │
                               ServerControlGUI.py ──HTTP :8080 (OTA)──> ESP32
```

## Repository layout

| Folder | What it is |
|---|---|
| [Flashing-ESP/](Flashing-ESP/README.md) | MicroPython firmware images, flashing instructions, and a MAC-ID helper |
| [Software/ESP-Files/](Software/ESP-Files/README.md) | Code that runs on the ESP32 detector boxes (Borehole, Surface, Veto) |
| [Software/Server/](Software/Server/README.md) | DAQ server that collects data and relays triggers, plus the Tkinter control GUI |
| [Analysis/](Analysis/README.md) | Offline analysis and plotting of server data files |
| [Electronic Box Calibration Code/](Electronic%20Box%20Calibration%20Code/README.md) | Bench calibration of each electronics box with a pulse generator |
| `u-blox-F9-TIM-2.25_InterfaceDescription_*.pdf` | u-blox UBX protocol reference for the GPS receiver |
| `Legacy Code.zip` | Archive of older test and analysis scripts |

## Typical workflow

1. Flash MicroPython onto each ESP32 and record its MAC address ([Flashing-ESP](Flashing-ESP/README.md)). The last byte of the MAC is the device ID used everywhere.
2. Upload the ESP files and pick the role (Borehole / Surface / Veto) ([ESP-Files](Software/ESP-Files/README.md)).
3. Start `GPS_DAQ_Server.py`, directly or from `ServerControlGUI.py` ([Server](Software/Server/README.md)).
4. Analyze the run folders with `Analysis/gps_daq_airshower_analysis.py` ([Analysis](Analysis/README.md)).

## Configuration placeholders

Credentials, IP addresses, and personal file paths have been replaced with placeholders. Fill them in locally before use, and don't commit the real values:

| Placeholder | Where | Meaning |
|---|---|---|
| `YOUR_WIFI_SSID` / `YOUR_ALT_WIFI_SSID` | ESP main files, calibration client | Wi-Fi network name |
| `YOUR_WIFI_PASSWORD` / `YOUR_ALT_WIFI_PASSWORD` | ESP main files, calibration client | Wi-Fi password |
| `YOUR_SERVER_IP` / `YOUR_LOCAL_SERVER_IP` / `YOUR_ALT_SERVER_IP` | `HOST` in ESP main files, calibration client | IP of the machine running the DAQ server |
| `PATH/TO/...` / `PATH\\TO\\...` | Analysis script, `ServerControlGUI.py`, flashing notes | Local folders on your machine |

## Shared data format

All TCP packets from the ESPs are **10 big-endian signed 32-bit ints** (`struct` format `"!iiiiiiiiii"`):

| Field | Meaning |
|---|---|
| `inst` | Instruction / packet type (e.g. 99 = data, 100 = error, 98 = rate, 90/91 = GPS health, 201–211 = GPS control). See `Software/Server/GPS_CONTROL_IDS.txt`. |
| `ID` | Sender ID: the last byte of the ESP's MAC (the GUI uses 1) |
| `RF` | 0 = rising edge, 1 = falling edge |
| `Cal` | Calibration flag |
| `ch` | Input channel (0 or 1) |
| `w_num` | GPS week number |
| `ms` | GPS time of week, milliseconds |
| `sub_ms` | Sub-millisecond remainder, nanoseconds |
| `event_num` | Event number assigned by the server (negative for Veto) |
| `count` | GPS count |

The server sends triggers to the ESPs as a **5-int** packet `"!iiiii"`: `inst, w_num, ms, sub_ms, event_num`.

Data files are text, with a header line and then one `;`-separated packet per line:

```
Req Code; ID; RF; Cal; Ch; W#; t_ow mil; t_ow submil; Event; GPS Count
```
