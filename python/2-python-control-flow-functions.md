# Python Cheat Sheet — Control Flow & Functions

Docs: https://docs.python.org/3/tutorial/controlflow.html

## Conditionals

```python
x = 10

if x > 5:
    print("big")
elif x == 5:
    print("medium")
else:
    print("small")

# Ternary (conditional expression) — compact one-line if/else
label = "big" if x > 5 else "small"
```

## Loops

```python
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 10, 2):    # start=2, stop=10 (exclusive), step=2 -> 2,4,6,8
    print(i)

for fruit in ["apple", "banana"]:
    print(fruit)

for index, fruit in enumerate(["apple", "banana"]):   # get index + value together
    print(index, fruit)

i = 0
while i < 5:
    print(i)
    i += 1
```

### Loop control
```python
for i in range(10):
    if i == 3:
        continue     # skip this iteration, go to next
    if i == 7:
        break         # exit the loop entirely
    print(i)
else:
    print("loop finished without break")   # 'else' on a loop runs only if no 'break' occurred
```

## Functions

```python
def greet(name):
    """Docstring: return a greeting for the given name."""
    return f"Hello, {name}"

greet("Adam")     # "Hello, Adam"
```

### Default & keyword arguments
```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}"

greet("Adam")                       # "Hello, Adam" — uses default
greet("Adam", greeting="Hi")         # "Hi, Adam" — keyword argument, order-independent
greet(name="Adam", greeting="Hi")     # equivalent, fully keyword-based
```

### *args and **kwargs
```python
def add_all(*args):            # collects any number of positional args into a tuple
    return sum(args)

add_all(1, 2, 3)     # 6

def show_info(**kwargs):        # collects any number of keyword args into a dict
    for key, value in kwargs.items():
        print(f"{key}: {value}")

show_info(name="Adam", role="Scientist")
```

### Type hints (current best practice)
```python
def add(a: int, b: int) -> int:
    return a + b
```
Type hints (PEP 484) don't enforce types at runtime, but improve readability,
enable IDE autocomplete, and let static checkers like `mypy` catch bugs
before running the code. Standard practice on any non-trivial modern Python
project. Docs: https://docs.python.org/3/library/typing.html

### Lambda (anonymous functions)
```python
square = lambda x: x ** 2
square(5)     # 25

sorted([3, 1, 2], key=lambda x: -x)     # [3, 2, 1] — custom sort logic inline
```
Use lambdas for short, throwaway functions (e.g. a `key=` argument) — for
anything more than one line of logic, a named `def` function is clearer.

## Error Handling

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except (TypeError, ValueError) as e:    # catch multiple exception types
    print(f"Error: {e}")
else:
    print("No error occurred")           # runs only if no exception was raised
finally:
    print("Always runs")                  # runs whether or not an exception occurred
```

### Raising your own exceptions
```python
def set_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    return age
```
Docs: https://docs.python.org/3/tutorial/errors.html

## Context Managers (`with`)

```python
with open("file.txt", "r") as f:
    contents = f.read()
# file is automatically closed here, even if an error occurred inside the block
```
`with` is the current best-practice way to handle any resource that needs
cleanup (files, database connections, locks) — prefer it over manual
`open()`/`close()` calls, since it guarantees cleanup runs.