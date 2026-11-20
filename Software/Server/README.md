# Server

Runs on the data-collection computer. The server accepts TCP connections from every ESP32 detector, writes their data to rotating files, and relays Borehole triggers to the surface detectors. The control GUI starts and stops the server, sends GPS configuration commands, plots health telemetry, and pushes OTA updates.

## Files

| File | Purpose |
|---|---|
| `GPS_DAQ_Server.py` | **Current** DAQ server (selectors-based, with a GUI control channel and health telemetry) |
| `KoForce_GPS_DAQ_Server.py` | Earlier server version (select-based, no control channel). Kept for reference. |
| `ServerControlGUI.py` | Tkinter + matplotlib control GUI |
| `GPS_CONTROL_IDS.txt` | Spec for the GUI→GPS control instruction codes (201–211) and TP1/PPS flag bits |

Requirements: Python 3. The server uses only the standard library. The GUI also needs `matplotlib`.

## GPS_DAQ_Server.py

```
python GPS_DAQ_Server.py [--file-hours 1.0] [--max-cycles 0]
```

- `--file-hours`: hours per data file before rotating to a new cycle.
- `--max-cycles`: stop after N cycles (0 = run forever).
- Listens on `0.0.0.0:12345` for ESPs and `12347` for the GUI control channel.
- A `GPS Data/` folder must exist in the working directory. Each run creates `GPS Data/Run_<NNNN>_<YYYYMMDD>/`, which contains:
  - `gps_daq_*.txt`: the data, one packet per line (see the root README for the format).
  - `connection_log.txt`, `error_log.txt`, `device_log.txt`
  - `gps_cpu_log.txt`, `stats_log.txt`: health telemetry (inst 90/91, 94–98) used by the GUI plots.

**Event logic**
- The server assigns event numbers itself.
- When the Borehole (ID 128) reports a ch0 rising edge, the server broadcasts a 5-int trigger (`inst, w_num, ms, sub_ms, event_num`) to every connected client except the Borehole, the Veto (16), and the GUI. The surface ESPs reply with the hits in their buffer near that time.
- Veto events get negative event numbers that count down.
- GPS time is converted to UTC with an 18 s leap-second offset (`GPS_UTC_OFFSET`).

## ServerControlGUI.py

```
python ServerControlGUI.py
```

Settings at the top of the file:
- `SERVER_SCRIPT`: path to `GPS_DAQ_Server.py` (placeholder `PATH/TO/...`). `GPS Data/` and `Survey Results/` are found relative to it.
- `OTA_SHELL`: shell used by the OTA terminal (`/bin/bash` on Linux).
- `DEVICE_IDS`: map from device name (BH, Veto, Det 1–10) to the ESP MAC byte. Update these to match your hardware.

Tabs:
- **Server Control**: start/stop the server process and view its output.
- **GPS Comms**: send control codes 201–211 to a chosen ESP over port 12347, e.g. survey-in, fixed coordinates, reset, measurement rate, MON-VER, TP1/PPS config, NAV-POSLLH.
- **GPS Slow Control** / **Data Slow Control**: live plots from the newest run's `gps_cpu_log.txt` and `stats_log.txt`.
- **Auto Survey**: runs survey-in across devices and saves `Survey Results/auto_survey_<timestamp>.txt`.
- **PPS Controls**: TP1/PPS settings.
- **OTA**: an interactive terminal for pushing code to the ESPs' OTA listener (HTTP port 8080).

## Known gaps

- The OTA tab refers to `execute_ota.py` and `version_probe.py`, which are not in this repo.
- `GPS_CONTROL_IDS.txt` names `esp_main.py` and `gps_ubx.py` as the ESP-side handlers for codes 201–211. They are not in this repo, and the current `Software/ESP-Files/Main/*_main.py` files do not handle those codes.
