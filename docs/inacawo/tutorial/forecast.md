# Forecast Mode

!!! warning
    The hands-on in this section is intended for internal BMKG users. Non-BMKG users may still be able to follow the tutorial without access to the computational resources.

!!! info
    Please contact the team for further information at [produksi.maritim@bmkg.go.id](mailto:produksi.maritim@bmkg.go.id).

The operational InaCAWO forecast system runs automatically on the BMKG HPC under account `cawofcst_prod`, managed by ROME. This page describes how the forecast mode works; for hands-on hindcast exercises see [Hindcast Mode](hindcast.md).

## Forecast schedule

InaCAWO produces **four forecast cycles per day**:

| Cycle (UTC) | Forecast length | Atmospheric IC/BC | Output products |
| :--- | :--- | :--- | :--- |
| **00, 12** | 10 days (240 h) | ECMWF-IFS (primary), GFS (backup) | FULL + ESSENTIAL |
| **06, 18** | 90 hours | GFS only | FULL only |

## End-to-end workflow

``` mermaid
flowchart TD
    A[Download global model data] --> B[Pre-process IC/BC]
    B --> C[72-hour ROMS/SWAN spin-up]
    C --> D[6-hour coupled analysis cycle]
    D --> E[Launch forecast run]
    E --> F[Post-process and disseminate]
```

### 1. Data ingest and pre-processing

Global model data are downloaded and converted into model-ready files:

- **WRF** — ECMWF-IFS or GFS atmospheric IC/BC via WPS (`ungrib`, `metgrid`, `real`)
- **ROMS** — CMEMS Mercator or HYCOM ocean IC/BC via MATLAB interpolation to the 70-layer grid
- **SWAN** — ECMWF-Wave or GFSWAVE boundary conditions via MATLAB

See [Configurations](../overview/configuration.md) for data source details.

### 2. Spin-up and analysis cycling

Before each forecast, the system runs:

1. **Standalone spin-up** (72 h) — ROMS and SWAN are driven with GFS analysis forcing to produce balanced hot-start files
2. **Coupled analysis cycle** (6 h) — the full 3-way coupled model runs from T−6 to T0, producing final ocean and wave initial conditions

### 3. Forecast execution

The coupled COAWST model runs in time chunks to minimize wall-clock delay:

**10-day runs (00/12 UTC):** 0–24, 24–48, 48–96, 96–144, 144–192, 192–240 h

**90-hour runs (06/18 UTC):** shorter chunked sequence

Each chunk waits for its IC/BC data, with ECMWF preferred and GFS as automatic fallback.

### 4. Post-processing and delivery

As forecast output timesteps become available:

- WRF and ROMS/SWAN outputs are converted to CF-1.8 NetCDF (FULL and ESSENTIAL)
- Products are FTP-delivered to downstream systems
- Plot scripts generate graphical products for the internal monitoring website

See [Dissemination](../operational/dissemination.md) for output variable lists and file naming.

## Key differences from hindcast mode

| Aspect | Forecast (operational) | Hindcast |
| :--- | :--- | :--- |
| **Trigger** | Automatic via ROME crontab | Manual execution |
| **Input data** | Near-real-time global models | Pre-acquired reanalysis (ERA5, GLORYS12V1) |
| **ROMS OBCs** | Zero-gradient velocities | Radiation + nudging with climatology |
| **Time pressure** | Must complete within service window | No real-time constraint |
| **Workflow** | ROME job tree with backups | Simplified independent components |

## Accounts and access

| Account | Node | Use |
| :--- | :--- | :--- |
| `cawofcst_prod` | `mdclogin0` | Production — do not modify without authorization |
| `cawofcst_dev` | `mdclogin1` | R&D testing and script development |

Software deployment repositories are available at `https://mms.bmkg.go.id/gitlab/mms/inacawo-deployment/`.

## Monitoring a forecast cycle

To check the status of a running or recent cycle:

```bash
# Set cycle (example: 12 UTC on 2024-05-30)
export CYCLE=2024053012

# Check cycle log directory
ls $RT_LOGS/cycles/$CYCLE/

# Check for flagged errors
ls $RT_LOGS/cycles/$CYCLE/verify/

# Review cron launch logs
ls $RT_LOGSCRON/ | grep $(date -u +%Y%m%d)
```

For detailed job listings and error recovery procedures, see [Production](../operational/production.md) and [Troubleshooting](../operational/troubleshooting.md).

## References

- `OM_InaCAWO_Software-Manual.pdf` — forecast job architecture (Jobs CF1, CF2)
- `Local Training InaCAWO August 2023.pdf` — operational training materials
- `20230615_InaCAWO_SAT_Operation_Test_M1_623_v1.0.pdf` — operational acceptance criteria
