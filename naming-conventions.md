# Naming Conventions Reference

Personal reference for consistent naming across files, directories, code, and environment variables. Platform: Linux (Ubuntu 24.04).

## Quick Reference Table

| Item | Convention | Example |
|---|---|---|
| Directories | `kebab-case` | `construct-assistant/`, `data-pipelines/` |
| Generic filenames | `kebab-case` | `naming-conventions.md`, `project-notes.md` |
| Python modules/packages | `snake_case.py` | `liability_screening.py`, `stability_scoring.py` |
| Python classes | `PascalCase` (identifier, not filename) | `class ProteinConstruct:` inside `protein_construct.py` |
| Python variables/functions | `snake_case` | `database_url`, `def parse_fasta():` |
| JS/TS variables/functions | `camelCase` (standard) | `const databaseUrl = ...` |
| React components (file) | `PascalCase.tsx` (ecosystem norm) | `UserProfile.tsx` |
| Conventional root files | Exact case tooling expects | `README.md`, `LICENSE`, `Dockerfile`, `Makefile` |
| Dotfiles / config filenames | lowercase | `.env`, `.env.local`, `.gitignore`, `.bashrc` |
| Environment variables | `UPPER_SNAKE_CASE` | `DATABASE_URL`, `API_KEY`, `DEBUG` |
| SQL tables/columns | `snake_case` | `qsar_results`, `pic50_value` |

> **Note on JS/TS:** `camelCase` is the ecosystem-standard convention for variables and functions (per [Airbnb JS Style Guide](https://github.com/airbnb/javascript#naming-conventions) and most linters/formatters like ESLint's default configs). This can be overridden to `snake_case` on a per-project basis if preferred for consistency with a Python-heavy codebase or personal taste — just apply it consistently within that project and note the deviation for anyone else touching the code.

## The Core Rule

**Directories and generic filenames → lowercase, hyphens, no spaces.**
**Language-specific source files → follow that language's own convention.**

### Why lowercase-kebab-case is the default for files/dirs

1. **Case sensitivity is a landmine.** Linux filesystems (ext4, etc.) are case-sensitive; macOS/Windows often aren't by default. Inconsistent casing (`MyFile.py` vs `myfile.py`) causes bugs that only show up cross-platform — broken imports, git seeing "no changes" on a rename, CI failures that don't reproduce locally.
2. **Hyphens over underscores** for files/dirs — the de facto convention for repo names, doc slugs, package folders, URLs. Easier to type (no Shift key).
3. **No spaces, ever** — spaces force quoting in every shell command (`cd "my folder"` vs `cd my-folder`), a constant avoidable friction in bash.

### Where this breaks (deliberately)

| Case | Convention | Why |
|---|---|---|
| Python module/package files | `snake_case.py` | [PEP 8](https://peps.python.org/pep-0008/#package-and-module-names) — matches the code inside |
| Conventional root files | `README.md`, `LICENSE`, `Dockerfile`, `Makefile` | Tooling (GitHub, Docker, Make) expects these exact names |
| React components | `PascalCase.tsx` | Common convention, matches the exported component name |
| Directories | Always `kebab-case` | No language enforces a rule on directory names — consistency wins |

## `.env` Files: Two Conventions in One File

The filename and the variables inside it follow **different** rules.

### The filename: stays lowercase

`.env`, `.env.local`, `.env.production` follow the standard dotfile convention (leading dot = hidden config file, lowercase like `.bashrc`, `.gitignore`).

### The variables inside: `UPPER_SNAKE_CASE`

```bash
# .env
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
API_KEY=sk-xxxxxxxx
DEBUG=true
```

This is a **shell/POSIX convention for environment variables**, not a config-file convention — it predates `.env` files entirely (same reason `$PATH`, `$HOME`, `$JAVA_HOME` are all-caps).

**Why the distinction exists:**

1. **Shell convention signals scope.** `UPPER_SNAKE_CASE` = environment/exported variable; `lower_snake_case` = local shell script variable. At a glance you can tell what's "external" vs script-local:
   ```bash
   #!/bin/bash
   local_counter=0              # script-local, lowercase
   echo "$DATABASE_URL"         # environment var, uppercase — signals "external"
   ```
2. **Collision/namespace separation.** Uppercase for exported vars is a long-standing informal namespace split from shell internals.
3. **`.env` files are loaded as shell environment variables** (via `export $(cat .env | xargs)`, `python-dotenv`, `docker --env-file`, etc.), so they inherit the shell convention.

### The handoff point: env var → Python variable

Uppercase applies **only while the value lives as an environment variable.** Once pulled into code and bound to a name, it drops to normal language convention.

```python
import os
from dotenv import load_dotenv  # https://pypi.org/project/python-dotenv/

load_dotenv()  # reads .env into the process environment

# Still UPPER_SNAKE_CASE here — it's external data:
database_url = os.environ["DATABASE_URL"]  # but the Python variable itself
                                             # is lowercase snake_case (PEP 8)
```

## Reference Documentation

- [PEP 8 — Package and Module Names](https://peps.python.org/pep-0008/#package-and-module-names)
- [python-dotenv on PyPI](https://pypi.org/project/python-dotenv/)
- [The Twelve-Factor App — Config](https://12factor.net/config) — source of the "config in env vars" pattern most `.env` tooling implements
- [Google Shell Style Guide — Naming Conventions](https://google.github.io/styleguide/shellguide.html#s7.3-constants-and-environment-variable-names)