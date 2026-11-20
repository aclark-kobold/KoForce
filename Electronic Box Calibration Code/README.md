# Electronic Box Calibration Code

Bench calibration for each detector electronics box. A pulse generator with a known pulse width and period drives both GPS input channels. The box timestamps every edge, and the analysis checks the measured period, the pulse width, and the channel 0 vs channel 1 time difference against the set values.

This folder is a self-contained snapshot of an **older** version of the client and server (9-int packets). It is separate from the live code in `Software/`.

## Files

| File | Runs on | Purpose |
|---|---|---|
| `Calibration_Code/(Calibration)KoForce_Client.py` | ESP32 (upload as `main.py`) | Reads GPS timestamps over UART and sends them to the server as 9 unsigned ints (`"!IIIIIIIII"`). Includes its own Wi-Fi connect and a 5 s watchdog. |
| `Calibration_Code/(Calibration)KoForce_GPS_DAQ_Server.py` | PC | TCP server on `0.0.0.0:12345`. Writes `gps_daq_<date>_run<N>_cycle<M>.txt`, rotates at 1 MB, gzips old cycles, and logs `connection_log.txt` and `error_log.txt`. |
| `Calibration_Code/(Calibration)Data_File_Analysis.py` | PC | Parses a calibration data file and plots clock-period, pulse-width, and ch0/ch1 coincidence histograms. |
| `Calibration Data/` | — | Reference calibration runs for boxes 4, 5, 8, and 9 (`.txt`, with `.gz` copies). |

## Procedure

1. Connect the pulse generator to both channels of the box under test (e.g. 100 ns width, 5 ms period).
2. In the client, set `ssid`, `password`, and `HOST` (the IP of the PC running the server). Then upload it to the ESP32 as `main.py`.
3. On the PC, run `python "(Calibration)KoForce_GPS_DAQ_Server.py"` and let it collect data.
4. In `(Calibration)Data_File_Analysis.py`, set the `__main__` values:
   - `DATA`: the data file name.
   - `box_num`: the box under test.
   - `set_pulse_width`: in ns, e.g. `100`.
   - `set_pulse_period`: e.g. `5000000`.
   - `min` / `max`: percentile cuts.
5. Run `python "(Calibration)Data_File_Analysis.py"` (needs `numpy`, `matplotlib`).

## Data format

```
Req Code; ID; RF; Cal; Ch, W#; t_ow mil; t_ow submil; Event
99; 112; 0; 1; 1; 0; 1267747; 468807; 0
```

There are 9 columns: instruction, device ID, rise(0)/fall(1), cal flag, channel, GPS week, time of week (ms), sub-ms remainder (ns), and event number.
