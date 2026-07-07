# Dissemination

InaCAWO produces atmospheric, ocean, and wave outputs in NetCDF format. Products are delivered to FISP components for marine weather analysis, archived in MANDALA, and displayed on the internal monitoring website.

## Output families

Two output families are generated:

| Family | Purpose | Consumers |
| :--- | :--- | :--- |
| **ESSENTIAL** | Reduced variable set to limit file size and processing time | FISP forecasting components |
| **FULL** | All model variables, all layers | MANDALA archive, API access |

## Output by forecast cycle

### 00 and 12 UTC (10-day forecasts)

Both FULL and ESSENTIAL products are produced.

| Product ID | File pattern | Content |
| :--- | :--- | :--- |
| WRF3D | `T_HCAW33_C_BMKG_*_WRF3D_R*_T*.nc` | Atmospheric 3D (FULL) |
| WRF2D | `T_HCAW22_C_BMKG_*_WRF2D_R2*_T*.nc` | Atmospheric 2D (FULL) |
| OCEAN3D | `T_HCAW11_C_BMKG_*_OCEAN3D_R*_T*.nc` | Ocean 3D + waves (FULL) |
| OCEAN2D | `T_HCAW00_C_BMKG_*_OCEAN2D_R*_T*.nc` | Ocean 2D + waves (FULL) |
| WRF3DESS | `T_HCAW34_C_BMKG_*_WRF3DESS_R*_T*.nc` | Atmospheric 3D (ESSENTIAL) |
| WRF2DESS | `T_HCAW23_C_BMKG_*_WRF2DESS_R2*_T*.nc` | Atmospheric 2D (ESSENTIAL) |
| OCEAN3DESS | `T_HCAW12_C_BMKG_*_OCEAN3DESS_R*_T*.nc` | Ocean 3D (ESSENTIAL) |
| OCEAN2DESS | `T_HCAW01_C_BMKG_*_OCEAN2DESS_R*_T*.nc` | Ocean 2D (ESSENTIAL) |

### 06 and 18 UTC (90-hour forecasts)

Only FULL products are generated (WRF3D, WRF2D, OCEAN3D, OCEAN2D).

## Key output variables

### Atmospheric (WRF)

**ESSENTIAL 2D:** 2 m temperature, dewpoint, relative humidity, surface pressure, MSLP, 10 m U/V wind and gusts, precipitation, sensible heat flux, shortwave radiation, sea surface temperature, equivalent reflectivity.

**ESSENTIAL 3D:** Air temperature, dewpoint, relative humidity, U/V wind, potential temperature, geopotential height, vertical velocity, vorticity.

### Ocean and waves (ROMS + SWAN)

SWAN wave results are embedded in ROMS output files via the MCT coupler.

**ESSENTIAL 2D:** Sea surface height, shortwave radiation, significant wave height, mean/peak wave period and direction, swell and wind-sea partitioning, primary swell parameters.

**ESSENTIAL 3D:** U/V/W velocity, salinity, temperature, density.

## Delivery pipeline

``` mermaid
flowchart LR
    A[InaCAWO model output] --> B[Post-processing P1/P3/P5/P7]
    B --> C[CF-1.8 NetCDF reformatting]
    C --> D[ESSENTIAL products]
    C --> E[FULL products]
    D --> F[FTP to FISP]
    E --> G[MANDALA archive]
    B --> H[Plot generation P9]
    H --> I[Internal website]
```

Post-processing jobs (see [Production](production.md)) perform:

1. **Native format conversion** — WRF and ROMS HDF5/NetCDF-4 outputs to Baron I/O API CF-1.8 compliant NetCDF
2. **Reprojection** — data regridded to an agreed lat/lon display grid
3. **Naming convention** — files renamed per external provider requirements
4. **FTP delivery** — ESSENTIAL and FULL files sent to external destinations (jobs P2, P4, P6, P8)
5. **Plotting** — Python scripts generate graphical products for the internal website on `mdc-cawo1`

Outputs are processed hourly during forecast runs, with only a few minutes delay from raw model write time.

## References

- `InaCAWO-Outputs-Description_M1_842_V1.0.pdf` — complete variable lists and file naming
- `OM_InaCAWO_Software-Manual.pdf` — Section 2.2.4 (post-processing jobs)
- `D1_MMS1-ICD_M1_033_V4.0.pdf` — MMS interface control document
