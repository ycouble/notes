
# So you want to be a Python expert ?
Talk by James Powell: https://www.youtube.com/watch?v=cKPlPJyQrt4

## Topics
- Python data-model
- Decorators
- Generators
- Meta classes (not retranscripted here, he basically presented the mechanisms behind the ABC `@abstractclass` decorator)
- Context Managers

## Python data-model
Python is entirely inspectable and has a very linear execution pattern (everything is read from top to bottom).
The language makes it very easy to define operations, and to define different behaviours for any object.


```python
class Summer:
    def __init__(self, *args):
        self.coefs = list(args)
    
    # It is possible to define a __call__ function so that the object is callable
    def __call__(self):
        return sum(self.coefs)
    
    # To be able to use built-in operators you may define these operation 
    # with the corresponding "dunder" (double under) method
    def __add__(self, other):
        return Summer(*(self.coefs + other.coefs))
    
    def __repr__(self):
        return (f"Summer({self.coefs})")
```


```python
a = Summer(1,4,5,3,10)
print(a, a(), a+a, (a+a)())
```

    Summer([1, 4, 5, 3, 10]) 23 Summer([1, 4, 5, 3, 10, 1, 4, 5, 3, 10]) 46


Therefore, there is very limited difference between an empty object with a `__call__` method and a function.


```python
# In Object defined function
class Adder:
    def __call__(self, x, y):
        return x + y
add_obj = Adder()

# Function
def add(x, y):
    return x + y
```


```python
print(add_obj, add)
print(add_obj(10, 20), add(10, 20))
```

    <__main__.Adder object at 0x104a6f5f8> <function add at 0x104bf12f0>
    30 30


## Decorators
Decorators are a syntaxic sugar of Python to define a function to wrap around any kind of function. Their main goal is to factor code and make it easier to maintain wrapping functions


```python
def add(x, y):
    return x + y
def sub(x, y):
    return x - y
def mult(x, y):
    return x * y

print(add(10, 20))
print(sub(10, 20))
print(mult(10, 20))
```

    30
    -10
    200


For example if you want to debug this simple code, by printing the inputs and their types before executing the function, you could put the code in each function like this:


```python
def add(x, y):
    print(x, y)
    return x + y
def sub(x, y):
    print(x, y)
    return x - y
def mult(x, y):
    print(x, y)
    return x * y

print(add(10, 20))
print(sub(10, 20))
print(mult(10, 20))
```

    10 20
    30
    10 20
    -10
    10 20
    200


Or, you can define a function that, given a function, prints the inputs before returning the result of the function


```python
def printer(func):
    def wrapper(*args):
        print(*args)
        return func(*args)
    return wrapper

def add(x, y):
    return x + y
add = printer(add)
def sub(x, y):
    return x - y
sub = printer(sub)
def mult(x, y):
    return x * y
mult = printer(mult)

print(add(10, 20))
print(sub(10, 20))
print(mult(10, 20))
```

    10 20
    30
    10 20
    -10
    10 20
    200


And then, Python provides a syntactic sugar that does exactly these `add = printer(add)`, but more beautifully:


```python
def printer(func):
    def wrapper(*args):
        print(*args)
        return func(*args)
    return wrapper

@printer
def add(x, y):
    return x + y

@printer
def sub(x, y):
    return x - y

@printer
def mult(x, y):
    return x * y

print(add(10, 20))
print(sub(10, 20))
print(mult(10, 20))
```

    10 20
    30
    10 20
    -10
    10 20
    200


## Generators
Generators are essentially functions that can give the hand back to the caller in the middle of their execution. It is a way to introduce lib-level to user-level interaction.


```python
def one_then_two():
    print("first step")
    yield 1
    print("second step")
    yield 2
    print("third step: None")

print(one_then_two)
```

    <function one_then_two at 0x104bfd378>



```python
a = one_then_two()
b = next(a)
print(b)
b = next(a)
print(b)
next(a)
```

    first step
    1
    second step
    2
    third step: None



    ---------------------------------------------------------------------------

    StopIteration                             Traceback (most recent call last)

    <ipython-input-10-1e40d6f632f3> in <module>()
          4 b = next(a)
          5 print(b)
    ----> 6 next(a)
    

    StopIteration: 


When there is no more yield statement, the generator raises a `StopIteration` exception to indicate it.

Generators enables bidirectional interaction with the `send` built-in function


```python
a.send?
```


    [0;31mDocstring:[0m
    send(arg) -> send 'arg' into generator,
    return next yielded value or raise StopIteration.
    [0;31mType:[0m      builtin_function_or_method




```python
def lib(word):
    message = ""
    for _ in range(4):
        message = yield (message + " " + word)
        print("lib", message)
        
def user(lib, word):
    message = None
    try:
        for _ in range(10):
            message = lib.send(message) + " " + word
            print("user", message)
    except StopIteration:
        print("the end")
```


```python
ping = lib("ping")
pong = user(ping, "pong")

```

    user  ping pong
    lib  ping pong
    user  ping pong ping pong
    lib  ping pong ping pong
    user  ping pong ping pong ping pong
    lib  ping pong ping pong ping pong
    user  ping pong ping pong ping pong ping pong
    lib  ping pong ping pong ping pong ping pong
    the end


Generators are also used for lazy computing: Iterating over a list of items without computing all of their values at once. It is especially useful when you may not need all of the values.


```python
def integers():
    i = 0
    while True:
        yield i
        i += 1
        
for i in integers():
    print(i)
    if i >= 5:
        break
```

    0
    1
    2
    3
    4
    5


At all points in the above code, only one integer was kept in memory, compared with a huge (inifite) list of all integers.

## Context Managers
Context managers are functions that help manage before and after steps, like opening a connection to a db, opening a file, creating a table and dropping it after etc.


```python
def prepare(d):
    d["list"] = []
    d["dict"] = {}
    d["description"] = ""

def destroy(d):
    d.pop("list")
    d.pop("dict")
    d.pop("description")
    
def some_actions(d):
    d["list"].append(1)
    d["list"].append(2)
    d["dict"]["hello"] = 100
    d["description"] = "arbitrarily filled dict"
    
d = dict(a=19)
prepare(d)
print(d)
some_actions(d)
print(d)
destroy(d)
print(d)
```

    {'a': 19, 'list': [], 'dict': {}, 'description': ''}
    {'a': 19, 'list': [1, 2], 'dict': {'hello': 100}, 'description': 'arbitrarily filled dict'}
    {'a': 19}


To avoid doing the prepare/destroy each time you want to use the function `some_action`, it is possible to use a context manager, which are designed exactly for that.

A context manager is essentially an object with an `__enter__` and a `__exit__` method, it is called with the `with`statement.


```python
class DictManager:
    def __init__(self, d):
        self._dict = d
        
    def __enter__(self):
        self._dict["list"] = []
        self._dict["dict"] = {}
        self._dict["description"] = ""
    
    def __exit__(self, *args):
        self._dict.pop("list")
        self._dict.pop("dict")
        self._dict.pop("description")

d = dict(a=19)
print(d)
with DictManager(d):
    print(d)
    some_actions(d)
    print(d)
print(d)
```

    {'a': 19}
    {'a': 19, 'list': [], 'dict': {}, 'description': ''}
    {'a': 19, 'list': [1, 2], 'dict': {'hello': 100}, 'description': 'arbitrarily filled dict'}
    {'a': 19}


And since `__enter__` and `__exit__` are called in order, there can be a generator that creates the sequence:


```python
class DictManager:
    def __init__(self, gen):
        self.gen = gen
    
    def __call__(self, *args, **kwargs):
        self.args, self.kwargs = args, kwargs
        return self
        
    def __enter__(self):
        self.gen_inst = self.gen(*self.args, **self.kwargs)
        next(self.gen_inst)
    
    def __exit__(self, *args):
        next(self.gen_inst, None)

def tempdict(d):
    d["list"] = []
    d["dict"] = {}
    d["description"] = ""
    print("init d")
    yield
    d.pop("list")
    d.pop("dict")
    d.pop("description")
    print("restored d")
tempdict = DictManager(tempdict)
        
d = dict(a=19)
print(d)
with tempdict(d):
    print(d)
    some_actions(d)
    print(d)
print(d)
```

    {'a': 19}
    init d
    {'a': 19, 'list': [], 'dict': {}, 'description': ''}
    {'a': 19, 'list': [1, 2], 'dict': {'hello': 100}, 'description': 'arbitrarily filled dict'}
    restored d
    {'a': 19}


And that is basically what a context manager is. There is a predefined `contextmanager` decorator which does this for us:


```python
from contextlib import contextmanager
@contextmanager
def tempdict(d):
    d["list"] = []
    d["dict"] = {}
    d["description"] = ""
    print("init d")
    try:
        yield
    finally:
        d.pop("list")
        d.pop("dict")
        d.pop("description")
        print("restored d")
        
d = dict(a=19)
print(d)
with tempdict(d):
    print(d)
    some_actions(d)
    print(d)
print(d)
```

    {'a': 19}
    init d
    {'a': 19, 'list': [], 'dict': {}, 'description': ''}
    {'a': 19, 'list': [1, 2], 'dict': {'hello': 100}, 'description': 'arbitrarily filled dict'}
    restored d
    {'a': 19}


So, in the end, all you have to do to create a proper context manager is to import the `contextmanager` decorator from contextlib, and put it on a function that calls yield once, surrounded preferably by a `try`/`finally` statement to always perfome the closing instructions
