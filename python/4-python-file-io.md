# Python Cheat Sheet — File I/O & Modules

Docs: https://docs.python.org/3/tutorial/inputoutput.html

## Reading & Writing Files

```python
with open("data.txt", "r") as f:      # "r" = read mode (default)
    contents = f.read()                 # read entire file as one string

with open("data.txt", "r") as f:
    lines = f.readlines()                # list of lines, each ending in \n

with open("data.txt", "r") as f:
    for line in f:                         # memory-efficient — reads one line at a time
        print(line.strip())                  # .strip() removes the trailing newline

with open("output.txt", "w") as f:        # "w" = write mode, OVERWRITES existing content
    f.write("Hello\n")

with open("output.txt", "a") as f:        # "a" = append mode, adds to end without overwriting
    f.write("More text\n")
```
Always use `with` for file operations — it guarantees the file is closed
properly even if an exception occurs mid-read/write.

### File modes
| Mode | Meaning |
|---|---|
| `"r"` | read (error if file doesn't exist) |
| `"w"` | write (creates file, overwrites if it exists) |
| `"a"` | append (creates file if it doesn't exist) |
| `"r+"` | read and write |
| `"rb"` / `"wb"` | binary mode (images, non-text data) |

## Working with Paths

```python
from pathlib import Path

p = Path("data/file.txt")

p.exists()            # True/False
p.name                  # "file.txt"
p.stem                    # "file" (no extension)
p.suffix                   # ".txt"
p.parent                    # Path("data")

p.parent.mkdir(parents=True, exist_ok=True)   # create directories as needed

for file in Path("data").glob("*.csv"):        # iterate over matching files
    print(file)
```
`pathlib` (Python 3.4+) is the current best-practice way to handle
filesystem paths — prefer it over the older `os.path` string-based approach,
since `Path` objects are more readable and less error-prone across OSes.
Docs: https://docs.python.org/3/library/pathlib.html

## JSON

```python
import json

data = {"name": "Adam", "age": 30}

with open("data.json", "w") as f:
    json.dump(data, f, indent=2)     # write dict to a JSON file, pretty-printed

with open("data.json", "r") as f:
    loaded = json.load(f)              # read JSON file back into a dict

json_string = json.dumps(data)          # dict -> JSON string (in memory, no file)
parsed = json.loads(json_string)          # JSON string -> dict
```
Docs: https://docs.python.org/3/library/json.html

## CSV

```python
import csv

with open("data.csv", "r") as f:
    reader = csv.reader(f)
    for row in reader:                # each row is a list of strings
        print(row)

with open("data.csv", "r") as f:
    reader = csv.DictReader(f)          # each row is a dict, keyed by header names
    for row in reader:
        print(row["column_name"])

with open("out.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["name", "age"])
    writer.writerow(["Adam", 30])
```
For anything beyond quick scripts, `pandas.read_csv()` /
`DataFrame.to_csv()` is generally preferred for tabular data work — see the
data-science reference docs for that workflow.

## Imports & Modules

```python
import math                      # import a whole module
math.sqrt(16)                      # access via module.function

from math import sqrt               # import a specific function directly
sqrt(16)                              # call it without the module prefix

import numpy as np                   # import with an alias (very common convention)

from mypackage.submodule import my_function   # import from within your own package
```
Module naming convention: lowercase, short, `snake_case` if multi-word (e.g.
`my_module.py`, not `MyModule.py`).

## Environment & Package Management

```bash
python3 -m venv venv          # create a virtual environment named 'venv'
source venv/bin/activate        # activate it (Linux/macOS)
deactivate                        # exit the virtual environment

pip install package_name           # install a package
pip install -r requirements.txt      # install everything listed in a requirements file
pip freeze > requirements.txt          # snapshot currently installed packages to a file
pip list                                  # list installed packages
```
Docs: https://docs.python.org/3/library/venv.html

## Common Standard Library Modules Worth Knowing

```python
import os          # operating system interfaces (env vars, process info)
import sys          # interpreter internals, command-line args (sys.argv)
import datetime      # dates and times
import re              # regular expressions
import random           # random number generation
import itertools          # advanced iteration tools (combinations, permutations, etc.)
import collections          # specialized containers (Counter, defaultdict, namedtuple)
```
Full standard library index: https://docs.python.org/3/library/index.html