# docs-metmar

Technical documentation for marine meteorology systems maintained by the **Center for Marine Meteorology** (DMM), BMKG.

Published site: [devmetmar.github.io/docs-metmar](https://devmetmar.github.io/docs-metmar/)

Repository: [github.com/devmetmar/docs-metmar](https://github.com/devmetmar/docs-metmar)

## Contents

| Section | Description |
| :--- | :--- |
| [InaCAWO](docs/inacawo/) | Indonesia Coupled Atmosphere–Wave–Ocean — high-resolution coupled modelling for marine weather services across the Indonesian archipelago |
| [SWAN](docs/swan/) | Simulating WAves Nearshore — third-generation wave modelling for coastal and nearshore areas |

## Requirements

- Python 3.12+
- [Zensical](https://zensical.org/) (static site generator)

## Local development

```bash
# Install dependencies (pinned in requirements.txt)
pip install -r requirements.txt

# Preview locally (default: http://127.0.0.1:8000)
zensical serve

# Build static site to ./site
zensical build --clean
```

## Deployment

The site is built and published to [GitHub Pages](https://devmetmar.github.io/docs-metmar/) by GitHub Actions on every push to `main` (see [`.github/workflows/docs.yml`](.github/workflows/docs.yml)).

One-time setup in the GitHub repo: **Settings → Pages → Build and deployment → Source: GitHub Actions**.

A custom domain such as `docs.pusmar.org` can be added later under the same Pages settings (update `site_url` in `zensical.toml` to match).

`run-app.sh` is a legacy helper for the old tmux/`zensical serve` hosting on the server; it is not used for GitHub Pages.

## Contributing

### 1. Get the repository

```bash
git clone https://github.com/devmetmar/docs-metmar.git
cd docs-metmar
pip install -r requirements.txt
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

### 5. Push to GitHub

```bash
git checkout -b your-branch
git add docs/ zensical.toml
git commit -m "Add documentation for ..."
git push -u origin your-branch
```

Open a pull request on GitHub. Once merged to `main`, GitHub Actions rebuilds and deploys the site automatically.

## Project structure

```
docs-metmar/
├── .github/workflows/  # CI/CD (GitHub Pages deploy)
├── docs/               # Markdown source files
│   ├── index.md        # Site homepage
│   ├── inacawo/        # InaCAWO documentation
│   └── swan/           # SWAN documentation
├── zensical.toml       # Site configuration and navigation
├── requirements.txt    # Pinned build dependencies
├── run-app.sh          # Legacy server launcher (unused for Pages)
└── site/               # Build output (generated; gitignored)
```

## Contact

For questions about this documentation, contact [produksi.maritim@bmkg.go.id](mailto:produksi.maritim@bmkg.go.id).
