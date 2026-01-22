# Hindcast Mode

!!! warning
    The hands-on in this section is intended for internal BMKG users, non-BMKG users may still be able to follow the tutorial without access to the computational resources.

!!! info
    This hands-on tutorial is designed to run on the MDC Supercomputer using the `cawohdcst_ft2` user account.

    Please contact the team for further information at [produksi.maritim@bmkg.go.id](mailto:produksi.maritim@bmkg.go.id).

??? info "Contents ..."
    ### Contents

    - [Main differences from the forecast system](#main-differences-from-the-forecast-system)
        - [Preprocessing Workflow](#preprocessing-workflow)
        - [Physical Considerations](#physical-considerations)

    - [Dependencies](#dependencies)
        - [Conda Environment](#conda-environment)
        - [LiveOcean](#liveocean)

    - [Global Reanalysis Data Acquisition](#global-reanalysis-data-acquisition)
        - [ERA5 Atmospheric Data](#era5-atmospheric-data)
        - [ERA5 Waves Data](#era5-waves-data)
        - [Mercator GLORYS12V1 Data](#mercator-glorys12v1-data)

    - [Pre Processing](#preprocessing)
        - [WRF Forcing Files](#wrf-forcing-files)
        - [SWAN Forcing Files](#swan-forcing-files)
        - [ROMS Forcing Files](#roms-forcing-files)

    - [Running The Hindcast](#running-the-hindcast)
        - [Compiling COAWST](#compiling-coawst)
        - [Running Scheme](#running-scheme)
        - [Modifying Namelist Files](#modifying-namelist-files)
        - [Copying input files](#copying-input-files)
        - [Running a day cycle](#running-a-day-cycle)
        - [Running automation](#running-automation)

    - [Post Processing](#postprocessing)
        - [WRF2D](#wrf2d)
        - [WRF3D](#wrf3d)
        - [Ocean2D](#ocean2d)
        - [Ocean3D](#ocean3d)

## Main differences from the forecast system

### Preprocessing workflow

The forecast input data is available only near each forecast cycle time. Thus, the data must be downloaded and processed as soon as possible so that the forecast outputs are generated in a timely manner.

To automate this process and ensure the results are delivered within the required time frame, the forecast system relies heavily on ROME tools. This suite of tools consists of complex driver scripts and watchdogs that manage each step of the workflow.

Although the hindcast workflow is based on the same processes used in the operational forecasts, it is greatly simplified because all input data for the 30-year period is already available. The hindcast components can run independently: while the hindcast is advancing through a given day, the reanalysis downloads and forcing-file generation can proceed for future dates. Likewise, postprocessing and model–data comparisons can be performed for earlier periods once their hindcast cycles have finished.

### Physical Considerations

The model configurations remain essentially the same, with one major exception: the ROMS open boundary conditions (OBCs).

In the forecast system, the velocities use zero-gradient boundary conditions, meaning the model does not ingest external information and simply extrapolates interior values to the boundary. This approach is acceptable for forecasts because each cycle spans only 10 days. However, in a long-term simulation the internal velocity fields will gradually drift away from realistic conditions. To address this, the hindcast uses radiation + nudging OBCs for the velocities.

For long simulations, it is also recommended to apply a light nudging zone near the boundaries, consistent with the sponge-layer (Marchesiello et al., 2003). Accordingly, a climatology nudging was applied to the four grid points closest to each boundary. The nudging was applied to temperature, salinity, baroclinic and barotropic velocities, and sea surface height, all obtained from the Glorys Reanalysis.

Additionally, the boundary conditions for SSH and barotropic velocities were updated to the Flather–Chapman combination, which is known to be very stable when used together with the OBC options noted above.

## Dependencies

### Conda Environment
Downloading, preprocessing, and postprocessing scripts are based on python tools. To ensure reproducibility of the processes, conda environments should be created and activated previously.

In `cawohdcst_ft2` home directory, `conda`'s source code is available in `/home/cawohdcst_ft2/opt/miniforge3`.

`conda` is intentionally not activated by default (to avoid cluttering `.bashrc`) in order to keep working environment clean. Hence, you need to activate it by typing the following command.

```bash
source /home/cawohdcst_ft2/opt/miniforge3/etc/profile.d/conda.sh
conda activate
```

??? info "Validating conda"
    You should see the output like this if you type `conda info`
    ```bash
    (base) [cawohdcst_ft2@mdclogin1 ~]$ conda info

        active environment : base
        active env location : /home/cawohdcst_ft2/opt/miniforge3
                shell level : 1
        user config file : /home/cawohdcst_ft2/.condarc
    populated config files : /home/cawohdcst_ft2/opt/miniforge3/.condarc
            conda version : 25.11.0
        conda-build version : not installed
            python version : 3.12.12.final.0
                    solver : libmamba (default)
        virtual packages : __archspec=1=zen2
                            __conda=25.11.0=0
                            __cuda=0=0
                            __glibc=2.28=0
                            __linux=4.18.0=0
                            __unix=0=0
        base environment : /home/cawohdcst_ft2/opt/miniforge3  (writable)
        conda av data dir : /home/cawohdcst_ft2/opt/miniforge3/etc/conda
    conda av metadata url : None
            channel URLs : https://conda.anaconda.org/conda-forge/linux-64
                            https://conda.anaconda.org/conda-forge/noarch
            package cache : /home/cawohdcst_ft2/opt/miniforge3/pkgs
                            /home/cawohdcst_ft2/.conda/pkgs
        envs directories : /home/cawohdcst_ft2/opt/miniforge3/envs
                            /home/cawohdcst_ft2/.conda/envs
                platform : linux-64
                user-agent : conda/25.11.0 requests/2.32.5 CPython/3.12.12 Linux/4.18.0-348.23.1.el8_5.x86_64 rhel/8.5 glibc/2.28 solver/libmamba conda-libmamba-solver/25.11.0 libmambapy/2.4.0
                    UID:GID : 10064:10023
                netrc file : None
            offline mode : False
    ```

There are four environments created for preprocessing and postprocessing.

1. `cawo_post`: postprocessing
2. `cdsapi`: downloading ERA5 data
3. `copernicusmarine`: downloading MERCATOR/CMEMS data
4. `loenv`: preprocessing ROMS

The modules required for each environment are stored in a `yaml` file, located in `/home/cawohdcst_ft2/yaml`. These `yaml` files can be used for creating the intended environment if you want play around in your own computer.

??? info "Checking conda environments"
    ```bash
    (base) [cawohdcst_ft2@mdclogin1 ~]$ conda env list

    # conda environments:
    #
    # * -> active
    # + -> frozen
    base                 *   /home/cawohdcst_ft2/opt/miniforge3
    cawo_post                /home/cawohdcst_ft2/opt/miniforge3/envs/cawo_post
    cdsapi                   /home/cawohdcst_ft2/opt/miniforge3/envs/cdsapi
    copernicusmarine         /home/cawohdcst_ft2/opt/miniforge3/envs/copernicusmarine
    loenv                    /home/cawohdcst_ft2/opt/miniforge3/envs/loenv

    (base) [cawohdcst_ft2@mdclogin1 ~]$ 
    ```

### LiveOcean 

[LiveOcean(LO)](https://github.com/parkermac/LO){:target="_blank"} is used for ROMS preprocessing tool. It was customized by CLSBrasil team, so we will use the source code from `/scratch/cawohdcst_ft/home/cawohdcst/LO`. Luckily, you don't have to do that as it already exists in the `cawohdcst_ft2` home directory.

## Global Reanalysis Data Acquisition

To generate all the input files that are necessary to run the hindcast, global reanalysis data that are used as initial and boundary conditions need to be downloaded to cover the total period of simulation. This procedure doesn't need to be complete before the simulations start, but needs to be always ahead in time in respect from the simulation. 

The hindcast daily cycles start at 12h00 UTC each day. This is determined by the Glorys Reanalysis time which are daily means centered at 12h UTC. Thus, to facilitate the workflow for the generation of forcing files for each day, the global models data are also downloaded for each daily cycle starting at 12h00 UTC.

### ERA5 Atmospheric Data

The ECMWF ERA5 data can be downloaded using the ECMWF Climate Data Store API (CDSAPI). The `.cdsapirc` already configured in `cawohdcst_ft2` home directory, so you don't need to worry about this.

??? info "Click to see detailed parameters"

    #### 3D Data
    Pressure levels:
    1000, 950, 925, 900, 850, 800, 700, 600, 500, 400, 300, 250, 200, 150, 100, 70, 50, 30, 20, 10, 7, 5

    | ECMWF Parameter Code | GRIB Parameter Name | Name                          | Units |
    |----------------------|---------------------|-------------------------------|-------|
    | 129                  | GH                  | Geopotential Height           | m     |
    | 130                  | TT                  | Temperature                   | K     |
    | 131                  | UU                  | U component of Wind Velocity  | m/s   |
    | 132                  | VV                  | V component of Wind Velocity  | m/s   |
    | 157                  | RH                  | Relative Humidity             | %     |

    #### 2D Data

    | ECMWF GRIB Parameter Code | Parameter Name | Name                         | Units |
    |--------------------------|--------------|------------------------------|-------|
    | 165                      | 10U          | U component of the wind velocity | m/s |
    | 166                      | 10V          | V component of the wind velocity | m/s |
    | 167                      | 2T           | 2 m Temperature              | K     |
    | 168                      | 2D           | 2 m Dewpoint Temperature     | K     |
    | 172                      | LSM          | Land-Sea Mask                | 0/1 flag |
    | 134                      | SP           | Surface Pressure             | Pa    |
    | 151                      | MSL          | Mean Sea-Level Pressure      | Pa    |
    | 235                      | SKT          | Skin Temperature             | K     |
    | 31                       | CI           | Sea Ice Fraction             | fraction |
    | 34                       | SST          | Sea Surface Temperature      | K     |
    | 33                       | RSN          | Snow Density                 | kg/m³ |
    | 141                      | SD           | Snow Depth                   | m (water equivalent) |
    | 139                      | STL1         | Soil Temperature 0–7 cm      | K     |
    | 170                      | STL2         | Soil Temperature 7–28 cm     | K     |
    | 183                      | STL3         | Soil Temperature 28–100 cm   | K     |
    | 236                      | STL4         | Soil Temperature 100–255 cm  | K     |
    | 39                       | SWVL1        | Soil Moisture 0–7 cm         | m³/m³ |
    | 40                       | SWVL2        | Soil Moisture 7–28 cm        | m³/m³ |
    | 41                       | SWVL3        | Soil Moisture 28–100 cm      | m³/m³ |
    | 42                       | SWVL4        | Soil Moisture 100–255 cm     | m³/m³ |


The required variables from ERA5 are downloaded using the ECMWF CDS api and python sripts:

- 3D data download script: `/home/cawohdcst_ft2/get_era5/get_era5_pl_automate.py`
- 2D data download script: `/home/cawohdcst_ft2/get_era5/get_era5_surface_automate.py`

Adjust the dates in both `*.py` files.
<figure id="fig1" markdown="span">
    !["Adjusting the dates"](../img/era5-atm.png)
    <figcaption>Figure 1. Adjusting the dates.</figcaption>
</figure>

The outputs are placed on daily folders following the pattern: `/scratch/<your_user_name>/data/era5/era5_YYYYMMDD`, or in this case our username is `cawohdcst_ft2`.

The `*.grib` files are from the day represented in the folder names starting at 12h00 UTC until the next day 12h00 UTC.

??? info "If the process completes successfully ..."
    If the process completes successfully, your terminal should look similar to the image below. In this example, the terminal shows the output of `get_era5_surface_automate.py`. However, even if you are running `get_era5_pl_automate.py`, the terminal output should be very similar.

    <figure markdown="span">
        !["ERA5 ATM output"](../img/era5-atm-out.png)
        <figcaption>Figure 2. ERA5 ATM output.</figcaption>
    </figure>

### ERA5 Waves Data

From the ERA5, the following variables are used as SWAN boundary conditions and need to be downloaded:

- `significant_height_of_combined_wind_waves_and_swell`;
- `mean_wave_period`;
- `mean_wave_direction`;

The data from ERA5 are downloaded using the ECMWF CDS api and python script. The script is located in `/home/cawohdcst_ft2/get_era5/get_era5_waves.py`.

Similar with the previous script ([Figure 1](#fig1)), adjust the dates by replacing the value of variables `start_date` and `end_date` to specify the downloading period. 

The outputs are placed on daily folders following the pattern: `/scratch/<your_user_name>/data/era5_waves/era5_YYYYMMDD`, or in this case our username is `cawohdcst_ft2`. 

The `*.grib` files are from the day represented in the folder names starting at 12h00 UTC until the next day 12h00 UTC.

### Mercator GLORYS12V1 Data

From the Mercator Reanalysis GLORYS12V1 the following variables are used as ROMS initial, boundary conditions and climatology files and need to be downloaded:

- `uo` – Eastward current velocities;
- `vo` – Northward current valocities;
- `thetao` – Sea water potential temperature;
- `so` – Sea water salinity;
- `zos` – Sea surface height above geoid;

Data from GLORYS12V1 are downloaded using copernicusmarine api and python sripts.
Data download scripts: 

- `/home/cawohdcst_ft2/get_glorys/ get_glorys_reanalysis.py` (1995 - 2021)
- `/home/cawohdcst_ft2/get_glorys/ get_glorys_reanalysis_interim.py` (2021 - 2024)

Similar with the previous script ([Figure 1](#fig1)), adjust the dates by replacing the value of variables `start_date` and `end_date` to specify the downloading period. 

The daily GLORYS12V1 files are placed on: `/scratch/<your_user_name>/mercator/GLORYS_Reanalysis_LO_YYYY-MM-DDT00:00:00.nc`, or in this case our username is `cawohdcst_ft2`.

*work in progress*