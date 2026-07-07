# Production

Production forecasts are launched automatically from the ROME crontab on account `cawofcst_prod` (`mdclogin0`). The pipeline below summarizes the major job groups; each job writes logs to `$RT_LOGSNOW` (`$RT_LOGS/cycles/yyyymmddcc`).

## 1. ROME housekeeping

| Job | Schedule | Script | Purpose |
| :--- | :--- | :--- | :--- |
| M1 | Every 5 min | `launch_monitor_oper4.csh` | Scan logs for errors, prepare alert messages |
| M2 | 00, 06, 12, 18 UTC | `launch_update_logsnow.csh` | Create cycle log directory |
| M3 | 02:55, 14:55 UTC | `launch_clean_dir_lists.csh` | Purge expired data to manage disk usage |

## 2. Global model data downloads

| Job | Schedule | Source | Purpose |
| :--- | :--- | :--- | :--- |
| D1 | 03:31, 09:31, 15:31, 21:31 UTC | GFS | Filtered atmospheric GRIB (240 h) |
| D2 | 03:40, 09:40, 15:40, 21:40 UTC | GFSWAVE | Wave GRIB data |
| D3 | 21:10 UTC | HYCOM (ESPC) | Ocean analysis (backup) |
| D4 | 06:05 UTC | CMEMS Mercator | Ocean analysis (primary) |

Tracking jobs (D5–D10) monitor download completeness and report status to Zabbix. File size and data-integrity checks ensure only complete datasets proceed to pre-processing.

## 3. Pre-processing

Pre-processing converts raw global model data into model-ready IC/BC files.

### ROMS (ocean)

| Job | Schedule | Input | Output |
| :--- | :--- | :--- | :--- |
| R1 | 09:00 UTC | Mercator | ROMS IC/BC (3D interpolation to 70-layer grid) |
| R2 | 00:40, 06:40, 13:40, 19:40 UTC | HYCOM | ROMS IC/BC (backup) |
| R3 | 04:00, 10:00, 16:00, 22:00 UTC | GFS analysis | ROMS atmospheric forcing for spin-up |

### SWAN (waves)

| Job | Schedule | Input | Output |
| :--- | :--- | :--- | :--- |
| E-S1 | 05:45, 17:45 UTC | ECMWF-Wave | SWAN boundary conditions |
| S1 | 03:45, 09:45, 15:45, 21:45 UTC | GFSWAVE (60 h) | SWAN BC (early chunk for analysis) |
| S2 | 04:15, 10:15, 16:15, 22:15 UTC | GFSWAVE (full) | SWAN BC (complete set) |
| S3 | 04:05, 10:05, 16:05, 22:05 UTC | GFS analysis | SWAN forcing for spin-up |

### WRF (atmosphere)

| Job | Schedule | Input | Output |
| :--- | :--- | :--- | :--- |
| E-W1 | 05:30, 17:30 UTC | ECMWF-IFS + ECMWF-Wave | Linked and converted GRIB for WPS |
| E-W2 | 05:35, 17:35 UTC | ECMWF | 10-day WRF IC/BC (chunked: 24, 48, 96, 144, 192, 240 h) |
| W1 | 03:45, 15:45 UTC | GFS | 10-day WRF IC/BC (backup for 00/12 UTC) |
| W2 | 09:35, 21:35 UTC | GFS | 90-hour WRF IC/BC (06/18 UTC) |
| F1 | 03:50, 09:50, 15:50, 21:50 UTC | GFS analysis | 6-hourly atmospheric forcing for ocean spin-up |

## 4. Model spin-up

Before each forecast, standalone spin-up runs produce balanced initial conditions for ROMS and SWAN.

| Job | Schedule | Duration | Description |
| :--- | :--- | :--- | :--- |
| E-S2 | 05:50, 17:50 UTC | 72 h | SWAN spin-up with ECMWF-Wave BCs (00/12 UTC) |
| S4 | 04:30, 10:30, 16:30, 22:30 UTC | 72 h | SWAN spin-up with GFSWAVE BCs (all cycles) |
| R4 | 15:55 UTC | 72 h | ROMS spin-up with Mercator IC/BC (daily) |
| R5 | 04:40, 10:40, 16:40, 22:40 UTC | 72 h | ROMS spin-up with HYCOM IC/BC (backup, 4× daily) |

## 5. Analysis cycling

Six-hour coupled-model runs (T−6 to T0) equilibrate the ocean and wave fields before forecast launch.

| Job | Schedule | Priority | Description |
| :--- | :--- | :--- | :--- |
| CA1 | 05:30, 11:30, 17:30, 23:30 UTC | Primary | 3-way coupled cycle; prioritizes Mercator/GFSWAVE spun-up ICs |
| CA2 | 05:30, 11:30, 17:30, 23:30 UTC | Backup | Uses HYCOM IC/BC every 6 hours without requiring spun-up ROMS |

## 6. Forecast runs

| Job | Schedule | Length | Atmospheric source | Ocean IC |
| :--- | :--- | :--- | :--- | :--- |
| CF1 | 05:57, 17:57 UTC | 240 h (10 days) | ECMWF (primary), GFS (fallback) | Hot-start from CA1 (backup CA2) |
| CF2 | 10:40, 22:40 UTC | 90 h | GFS | Hot-start from CA1 (backup CA2) |

Forecast runs execute in chunks (e.g. 0–24, 24–48, 48–96 h for ECMWF) so early timesteps are available before all IC/BC data arrive. SWAN outputs are passed through MCT to ROMS output files.

## 7. Post-processing

Post-processing jobs launch concurrently with forecast runs and begin as soon as the first output timestep is written.

| Job | Schedule | Component | Purpose |
| :--- | :--- | :--- | :--- |
| P1, P3 | 05:57, 17:57 UTC | WRF, Ocean | Process 10-day outputs (plots + NetCDF) |
| P2, P4 | 05:59, 17:59 UTC | WRF, Ocean | FTP delivery of 10-day products |
| P5, P7 | 11:30, 23:30 UTC | WRF, Ocean | Process 90-hour outputs |
| P6, P8 | 11:40, 23:40 UTC | WRF, Ocean | FTP delivery of 90-hour products |
| P9 | 04:55, 10:55, 16:55, 22:55 UTC | All | Watchdog — trigger plot generation for internal website |

## Monitoring

Operators can trace any job through its log tree starting from `$RT_LOGSCRON` (cron logs) down to `$RT_LOGSNOW` (cycle logs). ROME's monitor job (M1) scans logs every 5 minutes and flags errors in `$RT_LOGSNOW/verify/`.

!!! warning
    Installing or modifying the crontab requires care. Use `$RT_SCRIPTS/cron/install_cron.csh` and install jobs incrementally when testing changes. See the [Troubleshooting](troubleshooting.md) page for log analysis procedures.

## References

- `OM_InaCAWO_Software-Manual.pdf` — Sections 2.2.1–2.2.4 (complete job listing)
- `20230615_InaCAWO_SAT_Integration_Test_M1_622_v1.0.pdf` — integration test criteria
