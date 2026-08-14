# Python `rich` Library Cheatsheet

`rich` is a library for rendering formatted text, tables, progress bars, syntax-highlighted
code, tracebacks, and more in the terminal. It's a great fit for CLI tools like your
`biofetch` project or your trade tracker's `shell.py`.

- Docs: https://rich.readthedocs.io/en/stable/
- GitHub: https://github.com/Textualize/rich
- API reference: https://rich.readthedocs.io/en/stable/reference.html

---

## Install

```bash
pip install rich
```
No arguments needed here — plain `pip install <package>` pulls the latest stable
release from PyPI into whatever environment (venv/conda) is currently active.

---

## The `Console` object

`Console` is the central object in `rich`. Almost everything renders through it,
rather than through the built-in `print()`.

```python
from rich.console import Console

# Docs: https://rich.readthedocs.io/en/stable/console.html
console = Console()

# console.print() understands rich's markup and styling, unlike print()
console.print("Hello, [bold magenta]World[/bold magenta]!")
```

- `Console()` — creates a console instance. You typically create **one per
  application** and reuse it (e.g. a module-level `console` in your `shell.py`),
  rather than instantiating a new one every call.
- `console.print(*objects, style=None, ...)` — the `style` kwarg lets you apply
  a style to the whole call without inline markup tags.

---

## Inline markup (styling text)

Rich has its own lightweight markup language, similar to BBCode.

```python
console.print("[bold red]Error:[/bold red] Connection failed")
console.print("[italic yellow]Warning[/italic yellow]: low disk space")
console.print("[underline]Underlined[/underline] and [strike]struck through[/strike]")
```

- Tags open with `[style]` and close with `[/style]`. If you leave the closing
  tag empty, e.g. `[/]`, it closes the most recently opened style.
- Multiple styles can be combined in one tag: `[bold red on white]`.
- Docs: https://rich.readthedocs.io/en/stable/markup.html

### Common style keywords
| Category  | Examples                                              |
|-----------|--------------------------------------------------------|
| Weight    | `bold`, `dim`                                          |
| Slant     | `italic`                                                |
| Decoration| `underline`, `strike`, `blink`                          |
| Color     | `red`, `green`, `blue`, `magenta`, `cyan`, `#ff8700`    |
| Background| `on red`, `on white`, `on #222222`                      |

Full color list: https://rich.readthedocs.io/en/stable/appendix/colors.html

---

## `Style` objects (reusable styling)

For styles you'll reuse across a project (e.g. all your trade tracker's "profit"
vs "loss" colors), define a `Style` object instead of retyping markup strings.

```python
from rich.style import Style

# Docs: https://rich.readthedocs.io/en/stable/style.html
profit_style = Style(color="green", bold=True)
loss_style = Style(color="red", bold=True)

console.print("Trade P&L: +$412.50", style=profit_style)
console.print("Trade P&L: -$88.10", style=loss_style)
```

---

## Tables

Useful for your trade tracker dashboard output or any tabular CLI report.

```python
from rich.table import Table

# Docs: https://rich.readthedocs.io/en/stable/tables.html
table = Table(title="Open Positions")

# add_column defines a column header and its default styling/justification
table.add_column("Symbol", style="cyan", no_wrap=True)
table.add_column("Qty", justify="right", style="magenta")
table.add_column("Avg Price", justify="right", style="green")

# add_row adds one row of data; args map positionally to the columns above
table.add_row("AAPL", "100", "$185.32")
table.add_row("MSFT", "50", "$412.10")

console.print(table)
```

- `Table(title=..., show_header=True, header_style="bold", box=None, ...)` —
  `box` controls the border style; set `box=None` for a borderless table, or
  pass a style from `rich.box` (e.g. `box.SIMPLE`, `box.MINIMAL`).
- `justify` on `add_column` accepts `"left"`, `"center"`, `"right"`.

---

## Progress bars

Good for long-running ETL steps (e.g. your `cmd_weather_etl` pipeline) or model
training loops.

```python
from rich.progress import track
import time

# Docs: https://rich.readthedocs.io/en/stable/progress.html
# track() wraps any iterable and displays a progress bar as you loop over it
for item in track(range(20), description="Processing..."):
    time.sleep(0.05)  # simulate work
```

For finer control (multiple concurrent tasks, custom columns):

```python
from rich.progress import Progress

with Progress() as progress:
    # add_task returns a task ID used to update that specific bar's progress
    task = progress.add_task("[green]Downloading...", total=100)

    while not progress.finished:
        progress.update(task, advance=5)  # advance=amount to increment by
        time.sleep(0.1)
```

---

## Syntax highlighting

Handy for a CLI tool that prints code snippets or config files.

```python
from rich.syntax import Syntax

# Docs: https://rich.readthedocs.io/en/stable/syntax.html
code = '''
def add(a, b):
    return a + b
'''

# lexer="python" tells rich which grammar to use for highlighting
# theme picks a color scheme (e.g. "monokai", "ansi_dark", "dracula")
# line_numbers=True prepends line numbers to the output
syntax = Syntax(code, lexer="python", theme="monokai", line_numbers=True)
console.print(syntax)
```

---

## Panels (boxed content)

```python
from rich.panel import Panel

# Docs: https://rich.readthedocs.io/en/stable/reference.html#rich.panel.Panel
console.print(Panel("Trade executed successfully.", title="Status", border_style="green"))
```

---

## Pretty-printing data structures

Replaces `pprint` for dicts, lists, dataclasses, etc., with syntax-aware coloring.

```python
from rich.pretty import pprint

# Docs: https://rich.readthedocs.io/en/stable/pretty.html
data = {"symbol": "AAPL", "qty": 100, "tags": ["long", "swing"]}
pprint(data)
```

---

## Logging integration

Rich can format Python's standard `logging` output automatically.

```python
import logging
from rich.logging import RichHandler

# Docs: https://rich.readthedocs.io/en/stable/logging.html
logging.basicConfig(
    level="INFO",
    format="%(message)s",   # let RichHandler own the formatting, not logging's default
    handlers=[RichHandler()]
)

log = logging.getLogger("trade_tracker")
log.info("Fetched 42 trades from database")
log.warning("Missing close price for TSLA on 2026-08-10")
```

---

## Pretty tracebacks

Drop this near the top of your entry-point script (`main.py`) to get colorized,
more readable tracebacks for uncaught exceptions.

```python
from rich.traceback import install

# Docs: https://rich.readthedocs.io/en/stable/traceback.html
# show_locals=True prints local variable values at each frame — very useful
# while debugging, but consider turning it off for production/shared tools
# since it can leak sensitive data (e.g. API keys) into logs.
install(show_locals=True)
```

---

## Prompts (interactive input)

Useful for a tkinter-free quick entry form in the terminal.

```python
from rich.prompt import Prompt, Confirm, IntPrompt

# Docs: https://rich.readthedocs.io/en/stable/prompt.html
symbol = Prompt.ask("Enter ticker symbol")
qty = IntPrompt.ask("Enter quantity", default=100)
confirmed = Confirm.ask("Save this trade?")
```

---

## Live-updating displays

For dashboards that refresh in place (e.g. a live P&L view).

```python
from rich.live import Live
import time

# Docs: https://rich.readthedocs.io/en/stable/live.html
with Live(table, refresh_per_second=4) as live:
    for _ in range(10):
        time.sleep(0.5)
        # mutate `table` here, then call live.update(table) to redraw
        live.update(table)
```

---

## Quick reference: common imports

```python
from rich.console import Console
from rich.table import Table
from rich.progress import Progress, track
from rich.syntax import Syntax
from rich.panel import Panel
from rich.pretty import pprint
from rich.logging import RichHandler
from rich.traceback import install
from rich.prompt import Prompt, Confirm, IntPrompt
from rich.live import Live
from rich.style import Style
```

---

## Notes on best practice

- Prefer `console.print()` over bare `print()` once `rich` is in a project —
  mixing the two is fine, but `console.print()` is what respects your markup,
  themes, and terminal width detection.
- Instantiate a single `Console()` per application (module-level or passed via
  dependency injection) rather than creating new instances scattered through
  the codebase — this keeps output configuration (width, color system, force
  settings) consistent.
- `rich.traceback.install()` is a one-time call at startup, not something to
  call repeatedly.

Full API reference (all modules): https://rich.readthedocs.io/en/stable/reference.html