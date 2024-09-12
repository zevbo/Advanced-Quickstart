## Python Overview

Python is an interpreted, dynamically typed (aka: untyped) language. It's strengths lie in it's ease of use and the immense library of state of the art packages.

## Data Representation

All values in Python are runtime-inspectable.

## Variables

### Scopes

Before going over basic usage, let's quickly touch on scope. There is a global scope, and every function creates a new scope. Other constructs, such as ifs and for loops, generally do not create their own scope.

### Basic Usages

Variables are both defined and mutated with the `=` operator. Note that this can be confusing as it can be unclear which one is happening. Here are some examples:

```python
var1 = 0 # doesn't exist yet, so definition
def a():
    print(var1)
var1 = 1 # already exists in same scope, so it will set
a() # prints 1
def b():
    var1 = 2 # doesnt exist in same scope, so definition
b()
a() # still prints 1
```

You'll notice there's some odd scoping in `b`. Notably, if a variable name is ever set using `=` in a certain scope, any reference to that variable name in that scope will reference that one, even if there is another instance of that same name in an outer scope. If you want to explictly not define a new variable, and have the `=` set a variable in an outer scope, you use the keyword `nonlocal`. You can also use `global` to explictly set a global variable.

```python
var = 0
def a():
    var = 1
    def c():
        global var # refers to the var defined with "var = 0"
        var = 2
    c()
    def d():
        nonlocal var # refers to the var defined with "var = 1"
        var = 3
    d()
    return var
print(a()) # prints 3
print(var) # prints 2

new_var = 10
def bloop():
    print(new_var) # this fails! Because it's referencing the new_var declared on the next line
    new_var = 4
bloop()
```

## Type Hints

While Python is by default untyped, it supported "type hints." Type hints allow you to write in types for variables and functions, which another program (not the Python interpreter--most commonly Mypy) can use to type check your code. This gives you a significant amount of the safety that statically typed programming languages give you, though it isn't close to sound, so there is still a real disadvantage.

Another important note is that the external type checkers generally implement signifcant _type inference_. This means you mostly rarely need to type variables, and mostly just need to type functions. Here are some examples:

```python
var1: int = 1 # Syntax for typing a variable, but not necessary as type checker will iner
var2 = 2
a = "1" + var1 # Type checker will flag this line, can't add string an integer
a = "2" + var2 # Type checker will flag this line, because it has inerred var2 as an int

def sum_int_float(v1: int, v2: float) -> float:
    return v1 + v2 # this passes!

def print_sum(msg: str, v1: int, v2: float) -> None:
    print(f"{msg}: {sum_int_float(v1, v2)}") # This line is fine
    print(f"{msg}: {sum_int_float(v2, v1)}") # This line fails, because you can't cast a float to an int
```

Because you have to integerate with untyped code, Python allows you to use the `Any` keyword. All types are subtypes of `Any` (this makes sense), but in addition, `Any` is considred a subtype of every other subtype. This allows soundness to be broken pretty easily, but it is necessary so that you can tell the type checker to "trust you" about untyped code you are integrating with. This is where a Python's type hinting system loses a lot of robustness, but the convenience is a serious benefit.

You can both cast a type, or tell the type checker to ignore a line with a comment:

```python
from typing import cast
a = 0
b: str = a # Fails
c: str = cast(str, a) # Passes
d: str = a # type: ignore -- the "type: ignore" is read by the type checker, and so it won't flag this line
```

For generic types, you can use square brackets (note that the syntax is slightly different before 3.10). And you can use `...` to signify there can be any number of that type:

```python
def sum_plus_tuple_sum(l: list[float], tup: tuple[float, float]) -> tuple[float, float]:
    return (sum(l), tup[0] + tup[1])

def alternating_sum_tuple(tup: tuple[float, ...]) -> float:
    s = 0
    for i, v in enumerate(tup): # this syntax is described later on
        sign = 1 if i % 2 == 0 else -1 # ternary
        s += v * sign
    return s
```

Other examples of constructs that can be typed like this are `dict[KeyType, ValType]` and `set[Type]`. Python also allows "union" types, meaning a value could be of type A or of type B. Union types can be expressed using `Union` or more commonly with `|`:

```python
def first_if_not_empty(l: list[int]) -> int | None:
    if len(l) == 0:
        return None
    else:
        return l[0]
```

### TypeVar

TypeVars are how you do generic in Python. They are used for classes (as described later on), but can also be used for functions.

```python
T = TypeVar("T")
def last(l: list[T]) -> T:
    return l[-1]
```

## Functions

### First-class

In python, functions are first class values. They can be passed around and set like any other value. Using the `def` syntax is more or less equivalant to setting a variable to an anonymous function. If you do:

```python
def b():
    return 0
def b2():
    return 1
def c():
    print(b())
c() # prints 0
b = b2
c() # prints 1
```

### Arguments

All arguments in python may be called with keywords:

```python
def a(x):
    return x
a(10) # valid
a(x=10) # also valid
a(y=10) # throws exception
```

You may add a `*` before an argument to capture all remaining ordered arguments as a list. Any arguments after that `*` must be done as kewyord args. And you can capture all keyword arguments as a dictionary with with `**` (though that is usually lazy)

```python
def b(x1, *args, x2, **kwargs):
    return x1 + sum(args)
b(1, x2=4) # 5
b(1, 2, 3, x2=4) # 10
b(1) # not valid
b(1, 2, 3) # not valid
```

Arguments may be given default values. No non-default ordered arguments (ie: arguments before a \* argument) may come after an argument with a default.

```python
def c(a, b = 0): # valid
    return a + b
def d(a, b = 0, c = 1, *args, keyword_arg, keyword_default_arg = "hello", **kwargs): # valid
    return a + b + c
def e(a = 0, b): # invalid
    return a + b
```

Notably the default value is calculated once at runtime, so defualt values you should be immutable.

```python
def a(l = []):
    l.append(0)
    return len(l)
a() # 1
a() # 2 -- that's confusing!
```

### Anonymous functions

You can define an annonymous function like so:

```
lambda a, b: a + b
```

### Decorators

Decorators allow you to "transform" functions. You can use one like so:

```python

def double_decorator(function):
    return lambda x: function(x * 2)

@double_decorator
def f(x):
    return x + 4

f(10) # 24
```

We first define a decorator function, whose argument is a function, and returns a new function. Then we use the `@decorator` syntax before the function so that the function is defined as `decorator(f)` rather than `f`.

## Classes

### Basic syntax

The following is a basic class defintion and usage in Python:

```python
class A:

    def __init__(self, construction_arg1): # constructor
        self.construction_arg1 = construction_arg1 # setting field

    def some_method(self, arg1, arg2):
        return self.construction_arg1 * arg1 + arg2

a = A(10)
a.some_method(5, 2) # 52
```

A couple of basic things to note:

- all methods start with a "self" argument (which you can name whatever you want, but is bad style to name anything else). That self is the "object" that's being called.
- special methods are generally denoted with `__SOMETHING__`. The constructor is really just a special method

### Overview

At their core, objects in Python are a value that can have arbitrary fields. Take any object `o`, and you can add a field to it with `o.some_random_field = 10`. All of the class definitions and special methods are more or less syntactic sugar around that. Take our example in the "basic syntax" section, and consider the line `a = A(10)`. What's really going on here, is something similar to the following:

```python
a = object() # basic object with the ability to set fields
a.__class__ = A
a.__name__ = "A"
a.__init__ = lambda *args, **kwargs: A.__init__(a, *args, **kwargs)
a.some_method = lambda *args, **kwargs: A.some_method(a, *args, **kwargs)
a.__init__(10)
```

You can't actually run the code above, because `object()` is immutable to the user, but this is still more or less what is going on under the hood.

### Special Methods

There are a number of "sepcial methods" you can define on your class. All that really means is their is some built in function that looks for a method with that name. For example:

```python
class TwoLists:
    def __init__(self):
        self.l1 = []
        self.l2 = []

    def __len__(self):
        return len(self.l1) + len(self.l2)
a = TwoLists()
len(a) # returns 0
```

[Here](https://www.pythonlikeyoumeanit.com/Module4_OOP/Special_Methods.html) is a decent list of special methods. For my money, the most important ones are: `__len__, __str__, __repr__, __getitem__, __setitem__`.

### Default Fields

You can explicitly write default fields outside of methods. Note that these are not static fields shared across a class. Rather, the field is attached to an object at construction, with the given value as a default.

```python
class A:
    b = 0
    def __init__(self):
        print(b)
        self.c = 0

a1 = A()
a2 = A()
print(a1.b) # 0
a1.b = 1
print(a1.b) # 1
print(a2.b) # 0
```

### Inheritance

Below is the syntax for inheritance in Python:

```python
class A:
    def __init__(self, v):
        self.v = v

    def a_method(self):
        return self.v * 2

class B(A): # B inherits from A

    def __init__(self, v1, v2):
        super().__init__(v1)
        self.v2 = v2

    def b_method(self):
        return self.v2 / 2
```

### Abstract Classes & Class Decorators

There are a number of features that are clearly missing: for instance abstract and static methods. These are generally implemented using decorators. Let's start with abstract classes, with the following example:

```python
from abc import ABC, abstractmethod # standards for abstract class

class A(ABC):

    def __init__(self, name):
        self.name = name

    @abstractmethod
    def get_value(self):
        ...

    def print_value(self):
        print(f"value for {self.name}: {self.get_value()}")
```

You can also use the `@staticmethod` decorator to create a static method, and a `@classmethod`, if you want it abstracted over the class.

```python
class B:

    def __init__(self, v1, v2):
        self.v1 = v2
        self.v2 = v2
```

## Iterators

Iterators are a simple abstraction over a "stream" of data. They only have one relevant function: next, which gets the next element in the stream, and throws a `StopIteration` exception if there are no more elements. Note that this means that often iterators are lazy.

### Iterable

An iterable is an object with `__iter__` method that returns an iterator. All iterators are themselves iterables.

### For loops

In Python, for loops allow you iterate through an iterable. They are by far the most common type of loop.

```python
for item in iterable:
    # do something...
```

Almost all data structures are iterable.

### Common Iterator Constructs

The most common iterator constructors are: `range`, `reversed`, `enumerate` and `zip`. Here are explanations for all of them:

- `range`: `range(min, max, step)` returns an iterator that will step from `min` to `max` with step sizes the size of `step`. If only 2 arguments are provided, `step` defaults to `1`, and if only one argument is provided, `min` defaults to `0`. It is inclusive of `min` and exclusive of `max`
- `reversed`: takes in an iterator and returns an iterator that runs through the elements in opposite order
- `enumerate`: takes in an iterator, and returns an iterator of tuples, where the first element in the tuple is the index of the element in the original iterator, and the second element is the value. For example:

```python
l = [8, 2, 4]
for i, el in enumerate(l):
    print(f"{i = }, {el = }")
```

Will print: "i = 0, el = 8\ni = 1, el = 2\ni = 3, el = 4\n"

- `zip`: takes in n iterators, and combines them into a single iterator, the ith element of which is a tuple of the ith element of each input. So:

```python
l1 = [0, 1]
l2 = [2, 3]
l3 = [4, 5, 6]
for t in zip(l1, l2, l3):
    print(t)
```

Will print "(0,2,4)\n(1,3,5)". Note that it will keep going until any of the iterators is emptied. If you would like it to throw an exception when the iterators are of different sizes, use the keyword argument `strict=true`.

You can also convert an iterator to a `list` or `set` with the `list()` or `set()` functions:

```python
a = list(reversed([0, 1, 2])) # [2, 1, 0]
```

## Exceptions

To raise an exception, you simply use the `raise` keyword, followed by an instance from an exception object (any object that inherits from `BaseException`):

```python
raise Exception("Some message!")
```

If something raises an exception, you can catch it using a try/except:

```python
try:
    raise Exception()
except Exception as e:
    print(f"This is the exception object! {e}")
```

You can use the `raise` keyword by itself in order to re-raise an exception from an except clause:

```python
try:
    raise Exception()
except Exception as e:
    print(f"This is the exception object! {e}")
    raise # still raises the exception
```

You can also raise a new exception "from" the old exception, so that both exceptions get logged:

```python
try:
    raise Exception()
except Exception as e:
    print(f"This is the exception object! {e}")
    newException = Exception("Special message")
    raise newException from ex
```

You can also use multiple except clauses:

```python
try:
    raise Exception()
except RuntimeError as e: # RuntimeError is a subclass of Exception
    print("Twas a runtime error") # doesn't run
except Exception as e:
    print("Twasn't a runtime error") # does run
```

The `finally` keyword allows you to run a piece of code even if the portion in the `try` errors;

```python
try:
    raise Exception()
finally:
    print("A")
```

The exception is raised after the `finally` clause finishes.

## Data Types

Lists are the basic data type in Python (implemented as arraylists):

```python
a = [] # empty list
b = [0, 10, 20] # list with something
b.append(30) # b is now [0, 10, 20, 30]
b.exetend([4, 2]) # b is now [0, 10, 20, 30, 4, 2]
for v in b:
    print(v) # will print the elements of b in order
```

The other commonly used list methods are: `clear`, `copy`, `insert`, `pop`, `remove`, `sort`, and `reverse`. All of these (except `copy`) change the list in place.

Dictionaries (aka HashMaps) are used like so:

```python
a = {} # empty dictionary
b = {"key1": 0, "key2": "anything can have any type", 5: False}
c["key1"] # 0
c["fdsjkdfsj"] # throws an exception
"key1" in c # True
for key in b: # by default iterates through the keys
    ...
for value in b.values():
    ...
for key, value in b.items(): # allows you to iterate through keys + values
    ...
```

## Context Managers

## Parallelism

### Processes

### Async

### Threads

## Numpy

## Dataclasses

## Protocol (structural inheritance)
