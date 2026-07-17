# Reference Notes

Personal reference library — setup guides, cheat sheets, and notes collected
while working through coding, data science, Linux administration, and various
libraries/tools. Organized by topic so entries are easy to find later.

## How this repo is organized

Each topic lives in its own folder with markdown files inside. Two common
file types per topic:
- **`*-setup.md`** — install/config walkthroughs (what was broken, why, how it
  was fixed, with command explanations and doc links)
- **`*-cheatsheet.md`** — quick command/syntax reference for day-to-day use

```
/
├── README.md                      <- you are here
├── linux/
│   └── ...                        <- general Ubuntu/Linux admin notes
├── python/
│   └── ...                        <- Python environments, packaging, libraries
├── data-science/
│   └── ...                        <- pandas, numpy, ML libraries, stats notes
├── protein-science/
│   ├── pymol-mamba-setup.md
│   └── pymol-cheatsheet.md
├── databases/
│   └── ...                        <- PostgreSQL, SQLAlchemy, etc.
└── devops/
    └── ...                        <- Docker, git, CI/CD, servers
```

## Index

### Linux
_(add entries as they're created)_

### Python / Environments
_(add entries as they're created)_

### Data Science / ML
_(add entries as they're created)_

### Protein Science
- [PyMOL + Mamba setup](protein-science/pymol-mamba-setup.md) — installing
  PyMOL via conda-forge on Ubuntu, fixing the broken apt package
- [PyMOL cheat sheet](protein-science/pymol-cheatsheet.md) — commands for
  selections, rendering, alignment, and scripting

### Databases
_(add entries as they're created)_

### DevOps / Tooling
_(add entries as they're created)_

## Conventions

- Every setup doc explains *why* a step is needed, not just the command —
  these are meant to be readable months later without re-deriving context.
- Every command block includes a link to the official documentation.
- Legacy vs. current best-practice patterns are called out explicitly when a
  library has multiple approaches (e.g. SQLAlchemy 2.0 `DeclarativeBase` vs.
  the legacy `declarative_base()`).
- File names use `kebab-case` and end in `-setup.md` or `-cheatsheet.md` where
  applicable, so they're easy to scan in a directory listing.