# Operational Flow

The InaCAWO operational forecast system is a fully automated pipeline managed by **ROME** (Baron Real-Time Operational Modeling Environment) on the BMKG HPC. Cron-scheduled job scripts orchestrate data ingest, pre-processing, model spin-up, analysis cycling, forecast runs, and post-processing.

## High-level workflow

``` mermaid
flowchart LR
    A[Global model data ingest] --> B[Pre-processing IC/BC]
    B --> C[Model spin-up]
    C --> D[6-hourly analysis cycling]
    D --> E[Forecast run]
    E --> F[Post-processing]
    F --> G[Dissemination]
```

### Processing stages

| Stage | Description | Frequency |
| :--- | :--- | :--- |
| **Data ingest** | Download ECMWF, CMEMS Mercator, GFS, GFSWAVE, HYCOM | Per cycle / daily |
| **Pre-processing** | Generate WRF, ROMS, SWAN IC/BC and forcing files | Per cycle |
| **Spin-up** | 72-hour standalone ROMS and SWAN spin-up runs | 1–4× daily |
| **Analysis cycling** | 6-hour coupled-model spin-up (T−6 to T0) | Every 6 hours |
| **Forecast** | 10-day (00/12 UTC) or 90-hour (06/18 UTC) coupled run | 4× daily |
| **Post-processing** | NetCDF reformatting, plotting, FTP delivery | Concurrent with forecast |

## Forecast cycles

| Cycle (UTC) | Forecast length | Atmospheric IC/BC | Output products |
| :--- | :--- | :--- | :--- |
| 00, 12 | 10 days (240 h) | ECMWF (primary), GFS (backup) | FULL + ESSENTIAL |
| 06, 18 | 90 hours | GFS only | FULL only |

## ROME accounts

| Account | Node | Role |
| :--- | :--- | :--- |
| `cawofcst_prod` | `mdclogin0` | Production forecasts |
| `cawofcst_dev` | `mdclogin1` | Research and development |
| `web_prod` / `monitor` | `mdc-cawo1` | Internal website and monitoring |

## Sections

- [Production](production.md) — job scripts, spin-up, and forecast execution
- [Dissemination](dissemination.md) — output products, formats, and delivery
- [Troubleshooting](troubleshooting.md) — error handling, log analysis, and recovery

## References

- `OM_InaCAWO_Software-Manual.pdf` — Sections 2.2–2.2.6 (job architecture and timelines)
- `OM_InaCAWO_Related-Software-Manual.pdf` — ROME supervision and SOPs
- `20230615_InaCAWO_SAT_Operation_Test_M1_623_v1.0.pdf` — operational acceptance criteria
