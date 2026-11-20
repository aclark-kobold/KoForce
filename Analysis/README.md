# Analysis

Offline analysis of the data files written by the DAQ server (`GPS Data/Run_*/gps_daq_*.txt`). The script computes detector live time, PPS/clock period, time over threshold, coincidences between detectors, and DAQ health (packet integrity, rates, null counts), and saves the plots for each run.

## Files

- `gps_daq_airshower_analysis.py`: a single script with no command-line interface. Edit the settings in the `__main__` block, then run it.

## Requirements

```
pip install numpy matplotlib natsort
```

## Usage

1. In the `if __name__ == "__main__":` block, set:
   - `run_num`: run folder name, e.g. `"Run 75"`.
   - `save_folder_path`: where plots go. A `<run_num>` subfolder is created.
   - `data_folder_path`: folder that holds the run folders.
   - `DATA_FILE`: path to the joined data file (`joined_file.txt`).
   - `min` / `max`: percentile cuts used to trim outliers in the histograms.
2. Uncomment the analyses you want. By default only `gps_daq_health_variables(cycle_start, cycle_end, plotting)` runs.
3. Run `python gps_daq_airshower_analysis.py`.

## Main functions

| Function | Purpose |
|---|---|
| `folder_reader` / `file_joiner` | Collect the `gps_daq*` files in a run folder (natural sort) and concatenate them into `joined_file.txt` |
| `read_data` | Parse lines and split real data (`inst == 99`) into per-detector lists by device ID |
| `live_time_calculator` / `live_time_plotter` | Detector live time |
| `period_calculator` / `period_plotter` | Period between PPS/clock edges |
| `time_over_thres_calculator` / `tot_plotter` | Pulse width (time over threshold) from rising/falling edge pairs |
| `coincidence_finder` / `coincidence_plotter` / `..._per_file` | Time differences between detectors for the same event |
| `gps_daq_health_variables` | Feeds `integrity_plotter`, `rate_plotter`, and `real_null_counter_plotter` over a range of file cycles |

## Notes

- Device IDs are the last byte of each ESP's MAC address (e.g. 16 = Veto, 112 = Det 1, 180 = Det 2). This script uses **48** for the Borehole. The current server and GUI use **128**, so update `read_data` if your BH box is 128.
- Everything after about line 1224 is a commented-out (`"""`) older `__main__`, kept for reference (per-detector efficiency, error-log reader).
- File paths are placeholders (`PATH\\TO\\...`). Replace them with your local folders.
