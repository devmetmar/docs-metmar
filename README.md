# docs-metmar

Technical documentation for marine meteorology systems maintained by the **Center for Marine Meteorology** (DMM), BMKG.

Published site: [docs.pusmar.org](https://docs.pusmar.org)

Repository: [gitlab.pusmar.org/tyo/docs](http://gitlab.pusmar.org/tyo/docs)

## Contents

| Section | Description |
| :--- | :--- |
| [InaCAWO](docs/inacawo/) | Indonesia Coupled Atmosphere–Wave–Ocean — high-resolution coupled modelling for marine weather services across the Indonesian archipelago |
| [SWAN](docs/swan/) | Simulating WAves Nearshore — third-generation wave modelling for coastal and nearshore areas |

## Requirements

- Python 3.x
- [Zensical](https://zensical.org/) (static site generator)

## Local development

```bash
# Install Zensical
pip install zensical

# Preview locally (default: http://127.0.0.1:8000)
zensical serve

# Build static site to ./site (optional; not used in production)
zensical build --clean
```

## Deployment

The live site at [docs.pusmar.org](https://docs.pusmar.org) is served directly from this repository using Zensical's development server — there is no static build step or GitLab CI pipeline.

On the server, `run-app.sh` starts the server and keeps it running in a tmux session:

```bash
# Start (or attach to) the docs tmux session
tmux new -s docs -d ./run-app.sh

# Attach to an existing session
tmux attach -t docs
```

`run-app.sh` runs `zensical serve -a 0.0.0.0:7799`, which is proxied to `docs.pusmar.org`. Zensical watches the `docs/` directory, so after pulling new changes on the server the site updates automatically without restarting the process.

## Contributing

### 1. Get the repository

```bash
git clone http://gitlab.pusmar.org/tyo/docs.git
cd docs
```

### 2. Write or edit content

All documentation pages are Markdown files under `docs/`. Place new pages in the appropriate section:

```
docs/
├── index.md              # Homepage
├── inacawo/              # InaCAWO documentation
│   ├── overview/
│   ├── operational/
│   └── tutorial/
└── swan/                 # SWAN documentation
    ├── overview/
    ├── introduction/
    └── tutorial/
```

Use existing pages as a reference for style. Zensical supports [Material for MkDocs extensions](https://zensical.org/docs/authoring/markdown/) such as admonitions, code blocks, tabs, and Mermaid diagrams:

```markdown
!!! warning
    Important note for the reader.

!!! info
    Supplementary information.
```

### 3. Register new pages in navigation

If you add a new page, list it in the `nav` section of `zensical.toml` so it appears in the sidebar:

```toml
{ "Tutorial" = [
    "inacawo/tutorial/index.md",
    "inacawo/tutorial/forecast.md",
    "inacawo/tutorial/your-new-page.md",   # add here
] },
```

Also add a link on the section index page (e.g. `docs/inacawo/index.md`) if the page should be discoverable from there.

### 4. Preview your changes

```bash
zensical serve
```

Open [http://127.0.0.1:8000](http://127.0.0.1:8000) and check that the page renders correctly and appears in the navigation.

### 5. Push to GitLab

```bash
git checkout -b your-branch
git add docs/ zensical.toml
git commit -m "Add documentation for ..."
git push -u origin your-branch
```

Open a merge request on GitLab. Once merged, pull the changes on the server for them to appear on [docs.pusmar.org](https://docs.pusmar.org):

```bash
# On the server
cd /home/opn/apps/docs
git pull
```

## Project structure

```
docs-metmar/
├── docs/              # Markdown source files
│   ├── index.md       # Site homepage
│   ├── inacawo/       # InaCAWO documentation
│   └── swan/          # SWAN documentation
├── zensical.toml      # Site configuration and navigation
├── run-app.sh         # Production server launcher
└── site/              # Build output (generated; not used in production)
```

## Contact

For questions about this documentation, contact [produksi.maritim@bmkg.go.id](mailto:produksi.maritim@bmkg.go.id).
