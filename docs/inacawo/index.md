# InaCAWO

**Indonesia Coupled Atmosphere–Wave–Ocean** is a state-of-the-art high-resolution numerical modelling system that supports marine weather services across the Indonesian archipelago. It replaces BMKG's previously independent atmospheric, wave, and ocean models with a three-way coupled system running on the BMKG HPC.

## Sections

- [Overview](overview/index.md) — system background, capabilities, and validation
- [Configurations](overview/configuration.md) — model components, domain, input data, and HPC resources
- [Operational Flow](operational/index.md) — automated production pipeline
    - [Production](operational/production.md) — job scripts, spin-up, and forecast execution
    - [Dissemination](operational/dissemination.md) — output products and delivery
    - [Troubleshooting](operational/troubleshooting.md) — error handling and log analysis
- [Tutorial](tutorial/index.md) — hands-on guides
    - [Installation](tutorial/install.md) — HPC deployment
    - [Forecast Mode](tutorial/forecast.md) — operational forecast workflow
    - [Hindcast Mode](tutorial/hindcast.md) — long-term reanalysis simulations