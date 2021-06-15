---
title: "Decorators: A neat way to modify functions"
date: 2012-04-03T19:28:08Z
draft: false
aliases:
    - /articles/decorators-a-neat-way-to-modify-functions/
---

In Python, decorators are a construction that help reduce boilerplate code, enhance the maintainability of our code and address the separation of concerns. A decorator is a statement just above the function it decorates. Here's a simple example so you can understand what I mean:<!--more-->

```python
def heading1(func):
    """
    Define the decorating function - this wraps the return value of the 
    string in html heading tags. Don't actually write html like this - it is a
    silly idea.
    """
    def wrapper(*args, **kwargs):
        return "<h1>" + func(*args, **kwargs) + "</h1>"
    return wrapper

# Use the decorator here, before the function definition.
@heading1
def subject():
    return "Decorators: A neat way to modify functions"
    
print subject()
# Output is "<h1>Decorators: A neat way to modify functions</h1>"
```

That seems very neat, but this is a simple way of writing something complicated, so there are some kinks to iron out. The first is that in a generally applicable decorator, the decorating function must be able to accept an arbitrary argument list (meaning any arrangement of arguments and keyword arguments). The issue with this is that the resulting decorated function takes on the signature of the decorating function, so introspection of the function tells you that it accepts all positional and keyword arguments (`*args, **kwargs`).

The second problem is that the decorating function does not have the same attributes as the decorated function. E.g. `__name__`, `__doc__`, `__module__`, `__dict__` and in Python 3, `__annotations__`.

The solution to these problems is provided in the form of the [decorator](http://pypi.python.org/pypi/decorator/) package. This does all the leg-work of copying attributes as well as making signature-preserving decorators. As an added bonus, it removes the need to nest the wrapper function.

Here is the same trivial example, written using the `decorator` module:

```python
import decorator

@decorator.decorator
def heading1(func, *args, **kwargs):
    """This was the wrapper function for the decorator. It now takes the 
    caller function as the first argument. The defining of the decorator 
    itself is handled by... the decorator!
    """
    return "<h1>" + func(*args, **kwargs) + "</h1>"


# Use the decorator here, before the function definition.
@heading1
def subject():
    return "Decorators: A neat way to modify functions"
    
print subject()
# Output is "<h1>Decorators: A neat way to modify functions</h1>"
```

There are plenty more resources for managing and learning about decorators. If you want to avoid installing the decorators package, you can do some of the hard work with [functools.wraps](http://docs.python.org/py3k/library/functools.html?highlight=functools#functools.wraps). There's more about the decorators package and some good examples in the [package documentation](https://micheles.googlecode.com/hg/decorator/documentation3.html). [StackOverflow](http://stackoverflow.com/search?q=[python]+decorators) is always a good place to look for examples (good and bad). The Python docs have a page full of [decorator examples](http://wiki.python.org/moin/PythonDecoratorLibrary).

Thanks to saurik over on [Hacker News](https://news.ycombinator.com/item?id=3794443) for pointing out that I should really be using `decorator.decorator` as a decorator.
