# Python Cheat Sheet — Object-Oriented Programming

Docs: https://docs.python.org/3/tutorial/classes.html

## Defining a Class

```python
class Dog:
    # Class attribute — shared by all instances unless overridden
    species = "Canis familiaris"

    def __init__(self, name, age):
        # __init__ is the constructor, called automatically when creating an instance
        self.name = name    # instance attribute — unique to each object
        self.age = age

    def bark(self):
        return f"{self.name} says woof!"

    def __str__(self):
        # controls what print(instance) or str(instance) displays
        return f"Dog(name={self.name}, age={self.age})"


my_dog = Dog("Rex", 3)     # create an instance
my_dog.bark()               # "Rex says woof!"
print(my_dog)                 # "Dog(name=Rex, age=3)" — uses __str__
```
Class naming convention: `PascalCase` (e.g. `Dog`, `ProteinStructure`).
`self` is the conventional name for the first parameter of instance methods —
it refers to the specific instance the method is called on.

## Inheritance

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return f"{self.name} makes a sound"


class Cat(Animal):              # Cat inherits from Animal
    def speak(self):              # method override — replaces the parent's version
        return f"{self.name} says meow"


class Puppy(Animal):
    def __init__(self, name, breed):
        super().__init__(name)     # call the parent class's __init__
        self.breed = breed

    def speak(self):
        parent_sound = super().speak()    # call the parent's version, then extend it
        return f"{parent_sound} (a puppy bark)"
```
`super()` gives access to the parent class's methods — the current
best-practice way to extend rather than fully override behavior, and
required for correctly chaining `__init__` in multi-level inheritance.

## Class Methods, Static Methods, Properties

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        # lets you access .area like an attribute, not area() like a method,
        # while still computing it dynamically
        return 3.14159 * self.radius ** 2

    @classmethod
    def from_diameter(cls, diameter):
        # alternate constructor — receives the class itself (cls), not an instance
        return cls(diameter / 2)

    @staticmethod
    def describe():
        # doesn't need access to the instance or class — just lives in the class's namespace
        return "A circle is a round shape."


c = Circle(5)
c.area                      # 78.53975 — no parentheses needed, thanks to @property
Circle.from_diameter(10)      # equivalent to Circle(5)
Circle.describe()               # "A circle is a round shape."
```
Docs: https://docs.python.org/3/library/functions.html#property

## Common Dunder (Double-Underscore) Methods

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):
        # unambiguous representation, ideally valid Python to recreate the object
        return f"Point({self.x}, {self.y})"

    def __eq__(self, other):
        # controls behavior of ==
        return self.x == other.x and self.y == other.y

    def __add__(self, other):
        # controls behavior of +
        return Point(self.x + other.x, self.y + other.y)

    def __len__(self):
        # controls behavior of len()
        return 2
```
Full dunder method reference: https://docs.python.org/3/reference/datamodel.html#special-method-names

## Modern Alternative: Dataclasses

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

p = Point(3, 4)     # __init__, __repr__, and __eq__ are auto-generated
```
For simple classes that mostly just hold data, `@dataclass` (Python 3.7+) is
the current best-practice pattern — it eliminates boilerplate `__init__`,
`__repr__`, and `__eq__` code you'd otherwise write by hand.
Docs: https://docs.python.org/3/library/dataclasses.html