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
    print(f"{var1 = }")
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
```

## Functions

### First-class

In python, functions are first class values. They can be passed around and set like any other value. Using the `def` syntax is more or less equivalant to setting a variable to an anonymous function. If you do:

```python
def b():
    return 0
def c():
    print(f"{b}")
b = 10
c()
```

The call to `c()` will print 10.

### Arguments

All arguments in python may be called with keywords:

```python
def a(x):
    return x
a(10) # valid
a(x=10) # also valid
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

- all methods start with a "self" argument (which you can name whatever you want, but is bad style to name anything else). That self is the "object" that's being
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

You can't actually run the code above, because `object()` is weridly immutable to the user, but this is still more or less what is going on under the hood.

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

## Data Types

## Context Managers

## Type Hints

## Parallelism

### Processes

### Async

### Threads

## Numpy

## Dataclasses
