# Python Cheat Sheet — Basics & Syntax

Official docs: https://docs.python.org/3/tutorial/index.html
Style guide (naming conventions, formatting): https://peps.python.org/pep-0008/

## Running Python

```bash
python3 script.py            # run a script directly
python3                       # open the standard interactive REPL
ipython                       # open IPython (nicer REPL: tab-complete, history, magics)
```

Inside IPython:
```python
%run program.py    # run a script from inside an active IPython session
%timeit some_func()  # benchmark a line of code
%who                 # list variables defined in the session
```
IPython "magic commands" (prefixed with `%`) are IPython-specific, not part
of core Python. Docs: https://ipython.readthedocs.io/en/stable/interactive/magics.html

## Variables & Assignment

```python
x = 5                  # variables are dynamically typed — no declaration needed
name = "Adam"
is_active = True

a, b = 1, 2             # multiple assignment
a, b = b, a              # swap values without a temp variable

count = count + 1 if 'count' in dir() else 0
count += 1               # augmented assignment (also -=, *=, /=, //=, %=, **=)
```
Naming convention: `snake_case` for variables and functions (PEP 8).

## Core Data Types

```python
type(x)          # check a value's type

int              # whole numbers: 5, -3, 0
float            # decimals: 3.14, -0.5
str               # text: "hello"
bool              # True / False
complex           # complex numbers: 2+3j
NoneType          # the absence of a value: None
```

## Numbers & Operators

```python
7 + 3     # 10   addition
7 - 3     # 4    subtraction
7 * 3     # 21   multiplication
7 / 3     # 2.333...  true division (always returns float)
7 // 3    # 2    floor division (rounds down to nearest int)
7 % 3     # 1    modulo (remainder)
7 ** 2    # 49   exponent

# Comparison operators: ==, !=, <, >, <=, >=
# Logical operators: and, or, not
```

## Strings

```python
s = "Hello, World"

s.lower()              # "hello, world"
s.upper()               # "HELLO, WORLD"
s.strip()               # remove leading/trailing whitespace
s.split(",")             # ["Hello", " World"] — split into a list
",".join(["a", "b"])      # "a,b" — join a list into a string
s.replace("Hello", "Hi")  # "Hi, World"
len(s)                    # 12 — length of the string

s[0]        # "H" — indexing (0-based)
s[-1]       # "d" — negative indexing counts from the end
s[0:5]      # "Hello" — slicing [start:stop] (stop is exclusive)
s[::-1]     # reverse the string
```

### f-strings (modern, preferred formatting)
```python
name = "Adam"
age = 30
print(f"{name} is {age} years old")     # "Adam is 30 years old"
print(f"{3.14159:.2f}")                  # "3.14" — format spec: 2 decimal places
```
f-strings (PEP 498, Python 3.6+) are the current best-practice string
formatting method — prefer them over the legacy `%` formatting or
`str.format()`. Docs: https://docs.python.org/3/reference/lexical_analysis.html#f-strings

## Type Conversion

```python
int("5")        # 5      string -> int
float("3.14")    # 3.14   string -> float
str(5)           # "5"    int -> string
bool(0)          # False  0/empty values are "falsy"
bool(1)          # True   non-zero/non-empty values are "truthy"
list("abc")      # ['a', 'b', 'c']
```

## Comments & Docstrings

```python
# single-line comment

"""
Multi-line string, commonly used as a docstring when it's the
first statement inside a function, class, or module.
"""

def greet(name):
    """Return a greeting string for the given name."""
    return f"Hello, {name}"
```
Docstring convention reference: https://peps.print.python.org/pep-0257/ (PEP 257)

## Getting Help

```python
help(str)          # show documentation for an object/type
dir(str)            # list all attributes/methods available on an object
str.upper.__doc__   # view a specific method's docstring
```