# Installation

!!! warning
    InaCAWO deployment is restricted to authorized BMKG HPC accounts. This page summarizes the installation procedure; contact the system administrator for repository access.

## Prerequisites

- Access to BMKG HPC login nodes (`mdclogin0` for production, `mdclogin1` for R&D)
- Authorized account (`cawofcst_prod`, `cawofcst_dev`, or `cawofcst_test`)
- Compiled dependencies: Baron I/O API, m3tools, COAWST (WRF + ROMS + SWAN + MCT)

## Software repository

Deployment packages are hosted on the BMKG GitLab instance:

```
https://mms.bmkg.go.id/gitlab/mms/inacawo-deployment/
```

Gzipped tar archives are also available on the HPC:

| Account | Location |
| :--- | :--- |
| `cawofcst_prod` | `/scratch/cawofcst_prod/gzips/mdclogin0` |
| `cawofcst_dev` | `/scratch/cawofcst_prod/gzips/mdclogin1` |

A redundant copy of final deployment files is at `/scratch/backup_initial_testing/final`.

## Deployment steps

### 1. Extract repositories

Each component is distributed as a `.tar.gz` archive. Extract without modifying the directory structure:

```bash
cd $HOME
gunzip < /scratch/cawofcst_prod/gzips/mdclogin1/<repository>.tar.gz | tar xvf -
```

Inspect archive contents before extracting:

```bash
tar -tvf <repository>.tar | head
```

Each repository includes README files with component-specific instructions.

### 2. Build from source

Follow `README.master` in the deployment package to build:

1. Baron I/O API
2. Baron m3tools
3. COAWST coupled model (WRF, ROMS, SWAN, MCT)

### 3. Configure ROME

Connect the account to ROME by editing `$HOME/RC/my.cshrc`:

```csh
setenv RT_MODE prod        # or "test" for development
setenv RT_HOME ${HOME}/rome
set _setup_rome = ${RT_HOME}/oper/scripts/setup_rome_host.csh
source ${_setup_rome}
```

Link the login profile:

```bash
ln -sf RC/my.cshrc $HOME/.cshrc
```

Customize the ROME directory configuration file at `$RT_SCRIPTS/config/cawofcst_dirs_<machine>.dat` for data paths, model directories, and global model ingest locations.

### 4. Install crontab

!!! warning
    The system does not run operationally until a crontab is installed. Install jobs incrementally and verify each before adding the next.

```bash
# Review current crontab
crontab -l

# Install production crontab
$RT_SCRIPTS/cron/install_cron.csh cawofcst_prod@mdclogin0_utc.tab
```

For the R&D account on `mdclogin1`:

```bash
$RT_SCRIPTS/cron/install_cron.csh cawofcst_dev@mdclogin1_utc.tab
```

### 5. Verify deployment

After installation, confirm:

- ROME environment variables are set (`echo $RT_SCRIPTS`, `echo $RT_LOGS`)
- Log directories are writable
- A test pre-processing job completes without error
- Monitor job (M1) is scanning logs

## Web server components

The internal monitoring website runs on VM `mdc-cawo1` (IP `10.202.26.25`), not on the HPC:

| Account | Role |
| :--- | :--- |
| `web_prod` | Primary web server account |
| `monitor` | ROME monitoring messaging |

Web server repositories are in `/scratch/cawofcst_prod/gzips/mdc-cawo1/`. Deployment requires Apache configuration on a dedicated `/web` partition.

## References

- `OM_InaCAWO_Software-Manual.pdf` — Sections 4.1–4.5 (repository and deployment)
- `OM_InaCAWO_Related-Software-Manual.pdf` — Section 3.1 (ROME configuration)
- `M1-Lot-B2-Final_Acceptance_Report.pdf` — system acceptance documentation
