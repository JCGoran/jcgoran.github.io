---
layout: post
title: Dynamically listing writable properties in Python
tags: programming, python
mathjax: true
---
{% include mathjax.html %}

I've recently taken the monumental undertaking of refactoring my cosmology code, [COFFE][coffe-github], and building a semi-decent, user-friendly Python wrapper for it in [Cython][cython-wiki].
In the first iteration of it, I've decided to just shove all of the parameters (which the C version reads from a parameter file) into a single class (_cue Pylint complaining about too many attributes_), and make them Python [properties][property], so we could do something like this:

```cython
cdef class Coffe:
    cdef double _omega_cdm
    def __init__(self):
        # stuff here
        self._omega_cdm = 0.25
        # more stuff here

    @property
    def omega_cdm(self):
        return self._omega_cdm

    @omega_cdm.setter
    def omega_cdm(self, value):
        # type and value checking for `value`
        self._omega_cdm = value

# cythonize the module

from coffe import Coffe

cosmo = Coffe()
print(cosmo.omega_cdm) # prints 0.25
cosmo.omega_cdm = 0.27 # setting omega_cdm, the safe way
```

We also want a method which is similar to classy's (the Python wrapper to [CLASS][class-github]) `set` method, say, `set_parameters`, which takes a dictionary, and performs the bulk setting of the parameters (while doing the type and value checking of course).
This also comes in handy if we want output that's convertible to a [Pandas][pandas-wiki] DataFrame object.
Now, assuming we have a bunch of properties like above, how would we get a dictionary, with keys as the properties of the class, and values as, well, their values?
The first idea that came to mind was to just try `getattr` and manually specify the parameters:

```python
def get_properties(self):
    return {
        key : getattr(self, key) \
        for key in ('omega_cdm', 'omega_baryon', ...)
    }
```
Unfortunately, this doesn't really work well for at least two reasons:

1. we need to make sure we don't leave out any of the parameters in our list
2. what happens if we decide to change the names of the properties? Then we would have to rewrite the list of names every time, and we all know that naming things is [one of the hardest things in computer science][naming-things].

Additionally, we're dealing with Cython, and not Python, for which [this solution][python-get-properties] sadly doesn't work.

As a start, we will need the `dir` builtin, which more or less provides us with a list of attributes and methods our class has:

```python
dir(cosmo)
```

which outputs something like this:

```python
['__class__',
 '__delattr__',
 '__dict__',
 '__dir__',
 '__doc__',
# more stuff here
 '_omega_cdm',
 'omega_cdm']
```

Now, if we look at the docs of the `property` builtin, we'll see that it defines, among others, the `__set__` method, so we may have some luck with `hasattr`:

```python
hasattr(cosmo.omega_cdm, '__set__')
```

which, to my surprise, outputs `False`!
Upon closer inspection though, this makes sense, since `cosmo.omega_cdm` actually returns a `float`, which clearly doesn't have the `__set__` method.
On the other hand, `type(Coffe.omega_cdm)` returns `getset_descriptor`, so we may have better luck with it:

```python
hasattr(Coffe.omega_cdm, '__set__')
```

which returns `True`, so we're on the right track.
The question now is though, how do we access the attribute, without knowing the actual name of the attribute (in this case, `omega_cdm`)?
Why, with `getattr` of course!

```python
hasattr(getattr(Coffe, 'omega_cdm'), '__set__')
```

Now we just need to go through the contents of the output of `dir` and we're good to go.
Going back to the implementation of `get_properties`, our refined version looks like this:

```python
def get_properties(self):
    return {
        key : getattr(self, key) \
        for key in dir(self) \
        if hasattr(getattr(Coffe, key), '__set__')
    }
```

That `Coffe` is sticking out a bit like a sore thumb though, since it's not passed to the function, and we want to make our function as generic as possible.
Fortunately, Python has a solution for that as well: each object has a `__class__` attribute which returns the class itself, so instead of `Coffe`, we may as well write `self.__class__`.

There's a small problem in the above implementation: it will also return read-only properties, since the `property` decorator doesn't discriminate and just defines the `__set__` method for everything it's applied to, and we want to be able to do something like the below, without raising any errors[^prototype]:

```python
p1 = Coffe()
p2 = Coffe()
# change parameters in p1
...
# we want something like this to work
p2.set_parameters(p1.get_properties())
```

Since setting a read-only property always raises an `AttributeError`, we may as well attempt to set it to its current value, and if we're successful, the property is writable, otherwise it's read-only, and we don't do anything.
This brings us to the final implementation:

```python
def get_properties(self):
    properties = {
        key : getattr(self, key) \
        for key in dir(self.__class__) \
        if hasattr(getattr(self.__class__, key), '__set__')
    }
    writable = {}
    for key in properties:
        try:
            # try to set the attribute;
            # if it doesn't raise an error,
            # we add it to the new dictionary
            setattr(self, key, getattr(self, key))
            writable[key] = getattr(self, key)
        except AttributeError:
            pass
    return writable
```

And there we have it, a method that returns all of the writable attributes of a given class, in both Cython and pure Python (well, as long as the class in question isn't terribly complex).
Of course, to get the read-only attributes, we can just invert the `try`-`except` block in the above.

One thing to note (as pointed out in [this SO answer][dir-stackoverflow]) is that we are looping through the class (`self.__class__`) in the above, but checking the object (`self`);
this is intentional, since it's possible for an object to have additional attributes, as can be observed with this simple code:

```python
cosmo = Coffe()
set(dir(cosmo)) - set(dir(Coffe))
```

which outputs `{'_omega_cdm'}`, since this attribute doesn't technically exist until we create an object.

Note that if we want `get_property` to be a property itself, we need to change our conditional to `if key != 'get_properties' and hasattr(...)` in the final implementation (sadly, getting the name of a function is [a bit convoluted][self-name-stackoverflow], so we opt for specifying it as a string literal), otherwise the code would get stuck in a loop since `get_properties` would have a `__set__` method, and `getattr` would call `get_properties`, which would have a `__set__` method, and `getattr` would call `get_properties`...well, you probably get the picture :)


### Footnotes

[coffe-github]: https://github.com/JCGoran/coffe
[cython-wiki]: https://en.wikipedia.org/wiki/Cython
[property]: https://docs.python.org/3/library/functions.html#property
[class-github]: https://github.com/lesgourg/class_public
[python-get-properties]: https://stackoverflow.com/questions/17735520/determine-if-given-class-attribute-is-a-property-or-not-python-object
[naming-things]: https://martinfowler.com/bliki/TwoHardThings.html
[dir-stackoverflow]: https://stackoverflow.com/a/9693182
[^prototype]: at this point, we could just copy the whole object using the builtin `copy` module (possibly defining our custom `__copy__` and `__deepcopy__` methods along the way), but then we wouldn't have had so much fun with `getattr` and `hasattr`
[pandas-wiki]: https://en.wikipedia.org/wiki/Pandas_(software)
[self-name-stackoverflow]: https://stackoverflow.com/questions/5067604/determine-function-name-from-within-that-function-without-using-traceback
