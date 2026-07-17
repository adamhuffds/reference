# Python Cheat Sheet — Data Structures

Docs: https://docs.python.org/3/tutorial/datastructures.html

## Lists — ordered, mutable

```python
fruits = ["apple", "banana", "cherry"]

fruits[0]                # "apple" — indexing
fruits[-1]                 # "cherry" — negative indexing
fruits[0:2]                 # ["apple", "banana"] — slicing

fruits.append("date")        # add to the end
fruits.insert(1, "kiwi")      # insert at a specific index
fruits.remove("banana")        # remove by value (first match)
fruits.pop()                    # remove and return the last item
fruits.pop(0)                    # remove and return item at index 0
fruits.sort()                     # sort in place (alphabetical/numeric)
fruits.sort(reverse=True)          # sort descending
fruits.reverse()                    # reverse in place
len(fruits)                          # number of items
"apple" in fruits                     # True — membership test

sorted(fruits)                         # return a NEW sorted list, doesn't modify original
```

### List comprehensions
```python
squares = [x**2 for x in range(10)]                # [0, 1, 4, 9, ...]
evens = [x for x in range(20) if x % 2 == 0]         # filter with a condition
pairs = [(x, y) for x in range(3) for y in range(2)]  # nested loops
```
Comprehensions are the idiomatic, current best-practice way to build lists
from an iterable — prefer them over manually appending in a `for` loop when
the logic fits on one readable line.

## Tuples — ordered, immutable

```python
point = (3, 4)
x, y = point            # unpacking
point[0]                  # 3 — indexed like a list, but cannot be modified

single = (5,)               # trailing comma required for a one-element tuple
```
Use tuples for fixed collections that shouldn't change (e.g. coordinates,
function returns of multiple values) — immutability signals intent and is
slightly more memory-efficient than a list.

## Dictionaries — key/value pairs, mutable

```python
person = {"name": "Adam", "age": 30}

person["name"]                    # "Adam" — access by key
person.get("email")                 # None — safe access, no KeyError if missing
person.get("email", "N/A")           # "N/A" — with a default fallback

person["email"] = "a@example.com"      # add/update a key
del person["age"]                        # remove a key
person.pop("email")                       # remove and return a value

person.keys()          # view of all keys
person.values()          # view of all values
person.items()            # view of (key, value) pairs — common in for loops

for key, value in person.items():
    print(f"{key}: {value}")
```

### Dict comprehensions
```python
squares = {x: x**2 for x in range(5)}     # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

## Sets — unordered, unique values, mutable

```python
a = {1, 2, 3}
b = {2, 3, 4}

a | b       # {1, 2, 3, 4} — union
a & b       # {2, 3} — intersection
a - b       # {1} — difference
a ^ b       # {1, 4} — symmetric difference

a.add(5)
a.remove(1)     # raises KeyError if not present
a.discard(1)     # silent no-op if not present

3 in a       # True — fast membership testing (sets are hash-based)
```
Sets are the right tool when you need uniqueness or fast membership checks —
much faster than `in` on a list for large collections.

## Choosing the Right Structure

| Need | Use |
|---|---|
| Ordered, allow duplicates, changeable | `list` |
| Ordered, allow duplicates, fixed | `tuple` |
| Key-value lookup | `dict` |
| Unique values, fast membership tests | `set` |

## Common Built-in Functions Across Structures

```python
len(x)              # number of items
max(x) / min(x)       # largest/smallest value
sum(x)                  # sum of numeric values
sorted(x)                # new sorted list from any iterable
list(zip(a, b))            # pair up two iterables element-wise
enumerate(x)                 # get (index, value) pairs while iterating
list(reversed(x))              # reversed iterable as a list
```

Docs: https://docs.python.org/3/library/functions.html