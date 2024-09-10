## Python Overview

Python is an interpreted, dynamically typed (aka: untyped) language. It's strengths lie in it's ease of use and the immense library of state of the art packages.

## Data Representation

Because it is untyped, all values in Python are runtime-inspectable.

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
        global var
        var = 2
    c()
    def d():
        nonlocal var
        var = 3
    d()
    return var
print(a()) # prints 3
print(var) # prints 2
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

## Iterators

Iterators are a simple abstraction over a "stream" of data. They only have one relevant function: next, which gets the next element in the stream, and throws a `StopIteration` exception if there are no more elements.

## Iterable

An iterable is a val

### For loops

## Exceptions

## Data Types

## Context Managers

## Type Hints

## Parallelism

### Processes

### Async

### Threads

## Numpy
