# Troubleshooting

Under normal operation, InaCAWO is designed as an automated, self-correcting forecast system with built-in backup options in each job's decision tree. This page covers common failure modes and how to diagnose them using ROME logs and Zabbix status messages.

## Log analysis procedure

When ROME's monitor job (M1) flags an error, follow this procedure:

1. **Check verify directory** — inspect `$RT_LOGSNOW/verify/` for flagged log file names
2. **Open the cycle log** — the actual log is in `$RT_LOGSNOW` (e.g. `$RT_LOGS/cycles/2024052812/`)
3. **Trace to cron log** — identify the cron job schedule from `$RT_SCRIPTS/cron/cawofcst_prod@mdclogin0_utc.tab`, then review `$RT_LOGSCRON/`
4. **Follow the job tree** — top-level worker scripts may reference lower-level logs; follow references down the tree

Example: a missing `hycom_download.success` file means HYCOM pre-processing (Job R2) will not run. Trace back to Job D3's cron log in `$RT_LOGSCRON/` and the ingest log in `$RT_LOGS/ingest/`.

## Typical errors

### Global ocean dataset problems

Affects MERCATOR (Job D4/R1) and HYCOM (Job D3/R2) downloads and pre-processing.

| Cause | Symptom | Impact |
| :--- | :--- | :--- |
| Provider API change | Download script failures | Scripts revised when MERCATOR/CMEMS protocols update |
| Network interruption | Partial downloads, retry loops | Jobs retry until timeout; may switch to backup source |
| Bad data (NaNs) | MATLAB pre-processing issues | Filtered by nearest-neighbor fill; rare downstream ROMS errors |

The system is resilient — losing one day of HYCOM files typically does not disrupt service because Mercator is prioritized. Zabbix workflow files track which global ocean source is active per cycle.

### ECMWF-IFS timeliness (00/12 UTC only)

ECMWF is the preferred atmospheric source for on-cycle runs. If ECMWF files are not available within the wait window, the system falls back to GFS for both ICs and BCs.

| Scenario | Behavior |
| :--- | :--- |
| ECMWF 24 h files arrive on time | ECMWF used for entire 10-day forecast |
| ECMWF delayed beyond wait period | GFS fallback activated; flag recorded in Zabbix |
| ECMWF arrives in irregular batches | Pre-processing backlog may delay model chunks; forecast marked "slow" |

The forecast runs in six chunks (0–24, 24–48, 48–96, 96–144, 144–192, 192–240 h). Each chunk waits up to 30 minutes for ECMWF-based files before switching to GFS.

### ROMS model blow-ups

ROMS is sensitive to CFL violations when ocean velocities exceed the stability limit (transport distance > grid spacing in one timestep).

| Cause | Example |
| :--- | :--- |
| Pre-processing memory contention on login node | ROMS CFL error on off-cycle run (documented June 2023 SAT) |
| Bad boundary condition data | Silent MATLAB pre-processing failure propagating NaNs |
| Missing spun-up initial conditions | Using uninterpolated global model ICs |

When a ROMS blow-up occurs, the affected cycle's forecast may fail. Subsequent cycles typically recover if the root cause (e.g. login node contention) is resolved.

### HPC interruptions

| Issue | Effect | Recovery |
| :--- | :--- | :--- |
| Compute node failure | Single cycle may be missed | Next cycle uses previous restart files |
| Mellanox Infiniband issues | Batch job failures | Resubmit; ROME monitor flags the error |
| Disk space low | ROME cleaning alerts (Job M3) | Automatic purge per `mdclogin0_disks.cfg` thresholds |

## Zabbix status monitoring

Zabbix consumes text status files produced by tracking jobs (D5–D10) and forecast scripts. Key status categories:

| Category | What it tracks |
| :--- | :--- |
| Disk space warnings | Available space below configured thresholds |
| Pre-processing / analysis | Six-hourly analysis cycle completion |
| 10-day forecast runs | On-cycle (00/12 UTC) timeliness and fallback flags |
| Off-cycle forecast runs | 90-hour (06/18 UTC) completion |
| Post-processing | FTP transfer and plot generation status |

## Failure file

InaCAWO maintains a failure file that records cycle-level failures. Operators should review this file alongside ROME verify logs when investigating missed forecast cycles.

## When to intervene manually

Manual intervention is rarely needed. The system is designed to:

- Switch between primary and backup global data sources automatically
- Continue forecast production even when one ocean source is unavailable
- Alert operators via ROME monitor and Zabbix without stopping the pipeline

Intervention may be warranted when:

- Multiple consecutive cycles fail (possible HPC outage)
- Crontab changes are being deployed (install incrementally)
- Persistent ECMWF delivery issues degrade forecast quality (contact data provider)
- Disk space cleaning thresholds need adjustment

!!! info
    For operational support, contact [produksi.maritim@bmkg.go.id](mailto:produksi.maritim@bmkg.go.id).

## References

- `OM_InaCAWO_Related-Software-Manual.pdf` — Sections 3.2–3.3 (SOPs, error handling, Zabbix)
- `20230615_InaCAWO_SAT_Operation_Test_M1_623_v1.0.pdf` — operational acceptance test results
