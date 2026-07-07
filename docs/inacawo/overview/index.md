# Overview

BMKG previously operated independent atmospheric, wave, and ocean models, which limited the representation of cross-domain interactions in forecast outputs. To address this limitation, BMKG has implemented **InaCAWO** (Indonesia Coupled Atmosphere–Wave–Ocean), a three-way coupled modelling system built on the [COAWST](https://www.myroms.org/projects/coawst) (Coupled-Ocean-Atmosphere-Wave-Sediment Transport) framework.

InaCAWO seamlessly integrates three established models:

| Component | Model | Role |
| :--- | :--- | :--- |
| Atmosphere | [WRF](https://www.mmm.ucar.edu/weather-research-and-forecasting-model) | Atmospheric dynamics and surface forcing |
| Ocean | [ROMS](https://www.myroms.org/) | Three-dimensional ocean circulation |
| Waves | [SWAN](../swan/index.md) | Wind-wave generation and propagation |

The three-way coupling approach, pioneered by [Warner et al. (2010)](https://doi.org/10.1016/j.ocemod.2010.03.003), allows direct exchange of critical metocean surface data between models through the [Model Coupling Toolkit (MCT)](https://web.cels.anl.gov/projects/climate/mct/). Each component is aware of and responds to the others, much as these processes occur in nature.

## Key capabilities

- **High resolution** — uniform 3 km grid covering the Indonesian archipelago (90–145°E, 15°S–15°N)
- **Improved coupling** — representation of atmosphere–wave–ocean interactions in a single integrated system
- **Operational forecasting** — four cycles per day (00, 06, 12, 18 UTC) on BMKG HPC
- **Extended lead times** — 10-day forecasts at 00/12 UTC; 90-hour forecasts at 06/18 UTC

Compared to its predecessors, InaCAWO provides significantly higher spatial resolution, enabling finer-grained simulation of complex air–sea processes and more localized forecasts for severe weather and marine hazards — particularly tropical cyclones affecting southern Indonesian waters.

## System context

InaCAWO is the coupled forecast component of the BMKG **Maritime Meteorological System (MMS1)**. It receives global model data as initial and boundary conditions, runs on the BMKG HPC under the Baron **ROME** (Real-Time Operational Modeling Environment) workflow manager, and delivers products to downstream analysis, archiving (MANDALA), and dissemination systems.

Go to the [Configurations](configuration.md) page for model setup details, or the [Operational Flow](../operational/index.md) for the production workflow.

## References

Source documents in `refs/inacawo/`:

- `AGU_InaCAWO_Final_Draft-print.pdf` — scientific introduction
- `CAWO_ValidationStudyReport.pdf` — one-year validation study (Baron Weather, 2023)
- `OM_InaCAWO_Software-Manual.pdf` — system architecture and software manual (Baron Weather, 2024)
