# Python Reference — `os` Module & `.env` Files

Docs: https://docs.python.org/3/library/os.html | https://pypi.org/project/python-dotenv/

## Why `.env` Files Exist

Hardcoding secrets (API keys, database passwords, connection strings) directly
in your code is a security risk, especially if the code goes into version
control. The standard pattern instead:

1. Store secrets/config as key-value pairs in a `.env` file
2. Load them into environment variables at runtime
3. Read them in code via the `os` module
4. Add `.env` to `.gitignore` so it never gets committed

## The `os` Module — Environment Variables

```python
import os

os.environ["MY_VAR"]                 # get a value — raises KeyError if not set
os.environ.get("MY_VAR")               # get a value — returns None if not set (safer)
os.environ.get("MY_VAR", "default")      # get with a fallback default value

os.environ["MY_VAR"] = "value"             # set a variable for this process only
                                              # (does not persist outside the running script)

"MY_VAR" in os.environ                        # check if a variable exists
os.getenv("MY_VAR")                             # equivalent shortcut for os.environ.get()
```
`os.environ.get()` / `os.getenv()` are generally preferred over
`os.environ[...]` directly, since a missing variable returns `None` instead
of crashing the program with a `KeyError`.

## The `os` Module — Other Common Uses

```python
os.getcwd()               # get current working directory
os.listdir(".")             # list files/folders in a directory
os.path.exists("file.txt")    # check if a path exists
os.makedirs("a/b/c", exist_ok=True)   # create nested directories, no error if they exist
os.remove("file.txt")           # delete a file
os.rename("old.txt", "new.txt")  # rename/move a file
```
Note: for path manipulation specifically (joining, checking extensions,
etc.), `pathlib.Path` is the current best-practice choice over `os.path` —
see `python-file-io-modules.md`. The `os` module remains the right tool for
environment variables and process/OS-level operations.

## Creating a `.env` File

`.env` (plain text, no quotes needed around values, no spaces around `=`):
```
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
API_KEY=sk-abc123xyz
DEBUG=True
```

`.gitignore` — add this line so secrets never get committed:
```
.env
```

## Loading `.env` with `python-dotenv`

Python doesn't read `.env` files natively — the `python-dotenv` package
bridges the gap by loading them into `os.environ`.

```bash
pip install python-dotenv
```

```python
from dotenv import load_dotenv
import os

load_dotenv()      # looks for a .env file in the current directory and loads it into os.environ

db_url = os.getenv("DATABASE_URL")
api_key = os.getenv("API_KEY")
debug = os.getenv("DEBUG") == "True"   # env vars are always strings — cast/compare explicitly
```
Docs: https://pypi.org/project/python-dotenv/

### Loading from a specific path
```python
from pathlib import Path
from dotenv import load_dotenv

env_path = Path(__file__).parent / ".env"
load_dotenv(dotenv_path=env_path)
```
Useful when your script isn't run from the same directory the `.env` file
lives in — `load_dotenv()` with no arguments only searches the current
working directory and its parents by default.

## Type Casting Gotcha

All environment variables — whether set in `.env`, your shell, or Docker —
are strings, even if they "look like" numbers or booleans.

```python
port = os.getenv("PORT")            # "5000" (a string, not an int!)
port = int(os.getenv("PORT", 5000))  # cast explicitly, with a numeric fallback

debug = os.getenv("DEBUG", "False").lower() == "true"   # common boolean pattern
```

## Common `.env` Usage Patterns by Context

**Flask** — often auto-loaded if `python-dotenv` is installed and
`FLASK_ENV`/`FLASK_APP` are set, or load explicitly with `load_dotenv()` at
the top of your entry-point file.

**Docker Compose** — Compose reads a `.env` file in the same directory as
`docker-compose.yml` automatically, separate from anything `python-dotenv`
does inside the container. See `docker-compose-guide.md` for the
`environment:` block pattern.

**pydantic Settings** (modern alternative for larger projects) — instead of
scattering `os.getenv()` calls throughout your code, `pydantic-settings`
lets you define a typed settings class that reads from `.env` automatically:
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    debug: bool = False

    class Config:
        env_file = ".env"

settings = Settings()
settings.database_url   # already typed, validated, and cast correctly
```
Docs: https://docs.pydantic.dev/latest/concepts/pydantic_settings/
Worth adopting once a project's config grows beyond a handful of variables —
gives you validation and type safety that raw `os.getenv()` calls don't.