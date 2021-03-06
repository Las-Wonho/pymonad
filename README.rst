=======
PyMonad
=======

PyMonad is a small library implementing monads and related data abstractions -- functors, applicative functors, and monoids -- for use in implementing functional style programs.
For those familiar with monads in Haskell, PyMonad aims to implement many of the features you're used to so you can use monads in python quickly and easily.
For those who have never used monads but are interested, PyMonad is an easy way to learn about them in, perhaps, a slightly more forgiving environment without needing to learn Haskell.

Features
========

* Easily define curried functions with the ``@`` ``curry`` decorator.
* Straight-forward partial application: just pass a curried function the number of arguments you want.
* Composition of curried functions using ``<<``.
* Functor, Applicative Functor, and Monad operators: ``<<``, ``&``, and ``>>``.
* Monoids - int, float, str, list, List, Maybe, First, and Last: ``+``.
* Six predefined monad types.

1. Maybe - For when a calculation might fail.
2. Either - Similar to Maybe but with additional error reporting.
3. List - For non-deterministic calculations.
4. Reader - For sequencing calculations which all access the same data.
5. Writer - For keeping logs of program execution.
6. State - Simulating mutable state in a purely functional way.

Getting Started
===============

Installation
------------

Using pip::

    pip install PyMonad

Or download the package or clone the `git repository from bitbucket <https://bitbucket.org/jason_delaat/pymonad>` and run::

    python setup.py install

from the project directory.

Imports
-------

Import the entire package::

    from pymonad import *

Or just a single monad type::

    from pymonad.Maybe import *

If you're not importing everything but want to use curried functions::

    from pymonad.Reader import curry

Curried Functions and Partial Application
-----------------------------------------

To define a curried function use the ``@curry`` decorator::

    @curry
    def add(x, y):
        return x + y

    @curry
    def func(x, y, z):
        # Do something with x, y and z.
        ...

The above functions can be partially applied by passing them less than their full set of arguments::

    add(7, 8)       # Calling 'add' normally returns 15 as expected.
    add7 = add(7)   # Partial application: 'add7' is a function taking one argument.
    add7(8)         # Applying the final argument retruns 15...
    add7(400)       # ... or 407, or whatever.

    # 'func' can be applied in any of the following ways.
    func(1, 2, 3)   # Call func normally.
    func(1, 2)(3)   # Partially applying two, then applying the last argument.
    func(1)(2, 3)   # Partially applying one, then applying the last two arguments.
    func(1)(2)(3)   # Partially applying one, partially applying again, then applying the last argument.

Function Composition
--------------------

Curried functions can be composed with the ``<<`` operator.
Functions are applied from right to left::

    # Returns the first element of a list.
    @curry
    def head(aList):
        return aList[0]

    # Returns everything except the first element of the list.
    @curry
    def tail(aList):
        return aList[1:]

    second = head << tail   # 'tail' will be applied first, then its result passed to 'head'
    second([1, 2, 3, 4])    # returns 2

You can also compose partially applied functions::

    @curry
    def add(x, y):
        return x + y

    @curry
    def mul(x, y):
        return x * y

    comp = add(7) << mul(2)   # 'mul(2)' is evaluated first, and it's result passed to 'add(7)'
    comp(4)                   # returns 15

    # Composition order matters!
    comp = mul(2) << add(7)
    comp(4)                   # returns 22

Functors, Applicative Functors, and Monads
------------------------------------------

All Monads are also Applicative Functors, and all Applicative Functors are also Functors, though the same is not necessarily true in reverse.
All the types included with PyMonad are defined as all three but you can define new types however you want.

All of these types ultimately derive from ``Container`` which simply holds a value and provides some basic value equality checking.
The method ``getValue()`` will allow you to "extract" the value of monadic computations if/when necessary.

Functors
--------

All functors define the ``fmap`` method which can be invoked via the fmap operator ``<<``.
``fmap`` takes functions which operate on simple types -- integers, strings, etc. -- and allows them to operate of functor types::

    from pymonad.Maybe import *
    from pymonad.List import *

    # 'neg' knows nothing about functor types...
    def neg(x):
        return -x

    # ... but that doesn't stop us from using it anyway.
    neg << Just(9)                 # returns Just(-9)
    neg << Nothing                 # returns Nothing
    neg << List(1, 2, 3, 4)        # returns List(-1, -2, -3, -4)

Notice that the function is on the left and the functor type is on the right.
If you think of ``<<`` as a sort of fancy opening parenthesis, then normal calls and ``fmap`` calls have basically the same structure:

========  ========  ======  ===========  =====  ===========  ======
Type      Function  Opener  Argument     Sep    Argument     Closer
========  ========  ======  ===========  =====  ===========  ======
Normal    ``neg``   ``(``   ``9``                            ``)``
Normal    ``add``   ``(``   ``7``        ``,``  ``8``        ``)``
``fmap``  ``neg``   ``<<``  ``Just(9)``
``amap``  ``add``   ``<<``  ``Just(7)``  ``&``  ``Just(8)``
========  ========  ======  ===========  =====  ===========  ======


Notice that ``<<`` is also the function composition operator.
In fact, curried functions are instances of the ``Reader`` monad, and ``fmap`` -ing a function over another function is the same thing as function composition.
As shown next, functions that take multiple arguments can be called by chaining ("anding", ``amap`` -ing) the functors.

Applicative Functors
--------------------

Functors allow you to use normal functions of a single argument -- like ``neg`` above -- with functor types.
Applicative Functors extend that capability -- via ``amap`` and its operator ``&`` -- allowing you to use normal functions of multiple arguments with functor types::

    # 'add' operates on simple types, not functors or applicatives...
    def add(x, y):
        return x + y

    # ... but we're going to use it on those types anyway.
    # Note that we're still using '<<' but now in conjunction with '&'
    add << Just(7) & Just(8)                    # returns Just(15)
    add << Nothing & Just(8)                    # returns Nothing
    add << Just(7) & Nothing                    # returns Nothing
    add << List(1, 2, 3) & List(4, 5, 6)        # returns List(5, 6, 7, 6, 7, 8, 7, 8, 9)

If ``<<`` is a fancy parenthesis, ``&`` is the fancy comma used to separate arguments.

Monads
------

Monads allow you to sequence a series of calculations within than monad using the ``bind`` operator ``>>``.

The first argument to ``>>`` is a monad type.
The second argument is a function which takes a single, non-monad argument and returns an instance of the same monad::

    from pymonad.List import *
    from pymonad.Reader import curry

    # Takes a simple number type and returns a 'List' containing that value and it's negative.
    def positive_and_negative(x):
        return List(x, -x)

    # You can call 'positive_and_negative' normally.
    positive_and_negative(9)        # returns List(9, -9)

    # Or you can create a List...
    x = List(9)

    # ... and then use '>>' to apply positive_and_negative'
    x >> positive_and_negative        # also returns List(9, -9)

    # But 'x' could also have more than one value...
    x = List(1, 2)
    x >> positive_and_negative        # returns List(1, -1, 2, -2)

    # And of course you can sequence partially applied functions.
    @curry
    def add_and_sub(x, y):
        return List(y + x, y - x)

    List(2) >> positive_and_negative >> add_and_sub(3)        # creates List(2)
                                                              # applies positive_and_negative: List(2, -2)
                                                              # then add_and_sub(3): List(5, -1, 1, -5)
                                                              # final result: List(5, -1, 1, -5)

Variable assignment in monadic code
-----------------------------------

The second argument to ``>>`` is a function which takes a single, non-monad argument.
Because of that, you can use ``lambda`` to assign values to a variable withing monadic code, like this::

    from pymonad.Maybe import *

    Just(9) >> (lambda x:                 # Here, 'x' takes the value '9'
    Just(8) >> (lambda y:                 # And 'y' takes the value '8'
    Just(x + y)))                         # The final returned value is 'Just(9 + 8)', or 'Just(17)'

You can also simply ignore values if you wish::

    Just(9) >> Just(8)                    # The '9' is thrown away and the result of this computation is 'Just(8)'

Implementing Monads
-------------------

Implementing other functors, applicatives, or monads is fairly straight-forward.
There are three classes, serving as interfaces::

    Monad --> Applicative --> Functor

To implement a new functor, create a new class which derives from ``Functor`` and override the ``fmap`` method.

To implement a new applicative functor, create a new class which derives from ``Applicative`` and override the ``amap`` and ``fmap`` methods.

To implement a new monad, create a new class which derives from ``Monad`` and override at least the ``bind`` method, and preferably the ``amap`` and ``fmap`` methods as well.

The operators, ``<<``, ``&``, and ``>>`` are pre-defined to call the above methods so you shouldn't need to touch them directly.

unit (aka return)
-----------------

The previous version of PyMonad didn't include the method ``unit`` (called ``return`` in Haskell).  ``unit`` takes a bare value, such as ``8``, and places it in a default context for that monad.
Haskell allows polymorphism on return types as well as supporting type inference, so you (mostly) don't have to tell ``return`` what types to expect, it just figures it out.
We can't do that in Python, so you *always* need to tell ``unit`` what type you're expecting.

The ``unit`` method is implemented as a class method in Functor.py, so it can be used with any functor, applicative or monad.
There is also a ``unit`` *function* which expects a functor type (though you can also give it an instance) and a value and invokes the corresponding ``unit`` method.
It is provided to give a more "functional look" to code, but use whichever method you prefer.
With the Maybe monad for example:

1. Maybe.unit(8)         # returns Just(8)
2. unit(Maybe, 8)        # also returns Just(8)

In either case all functors (and applicatives and monads) should implement the ``unit`` class method.

Monoids
=======

Monoids are a data type which consists of some operation for combining values of that type, and an identity value for that operation.
The operation is called ``mplus`` and the identity value is called ``mzero``.
Despite the names, they are not necessarily addition and zero.
They *can* be addition and zero though, numbers are sort of the typical monoid.

In the case of numbers, zero is the identity element and addition is the operation.
Monoids adhere to the following laws:

1. Left and right identity: ``x + 0 = 0 + x = x``
2. Associativity: ``(x + y) + z = x + (y + z) = x + y + z``

Stings are also monoids with the identity element ``mzero`` equal to the empty string, and the operation ``mplus`` concatenation.

Creating New Monoids
--------------------

To create a new monoids type create a class deriving from ``Monoid`` and override the ``mzero`` static method which takes no arguments and should return an instance of the class containing the identity value for the monoid.
Also, override the ``mplus`` method.
For instance, numbers can be a monoid in two ways, one way with zero and addition as discussed above and the other way with one and multiplication.
We could implement that like this::

    class ProductMonoid(Monoid):
        @staticmethod
        def mzero():
            return ProductMonoid(1)

        def mplus(self, other):
            return ProductMonoid(self.getValue() * other.getValue())

The ``+`` operator (aka ``__add__()``) is defined to call ``mplus`` on monoid instances, so you can simply "add" monoid values together rather than having to call ``mplus`` directly.

"Natural" Monoids
-----------------

Similar to ``unit`` for monads, there is an ``mzero`` function which expects a type and can be used instead of the ``mzero`` method.
Unlike ``unit`` however, the ``mzero`` function serves another purpose.
Numbers, strings and lists can all be used as monoids and all already have an appropriate definition for ``+``.
What they don't have is an ``mzero`` method.
To allow numbers, strings and lists to be used as monoids without any extra work, the ``mzero`` *function* will return the appropriate value for these types and will attempt to call the ``mzero`` method on anything else.
For instance::

    mzero(int)                    # returns 0, also works with float
    mzero(str)                    # returns ""
    mzero(list)                   # returns []
    mzero(ProductMonoid)          # return ProductMonoid(1)
                                  # etc...

If you write code involving monoids, and you're not sure what type of monoid you might be handed, you should use the ``mzero`` *function* and *not* the ``mzero`` method.

Monoids and the Writer Monad
============================

The Writer monad performs calculations and keeps a log.
The log can be any monoid type -- strings being a typical example.

The ``Writer`` class doesn't have a default log type, so to use Writer you need to inherit from it.
It is extremely simple as the only thing you need to do is define the log type.
For instance::

    class StringWriter(Writer):
        logType = str

That's it.
Everything else is already defined by ``Writer``.
``StringWriter``, ``NumberWriter``, and ``ListWriter`` are already defined in the ``Writer`` module for you to use.

Calling ``unit`` with a ``Writer`` class packages whatever value you give it with the ``mzero`` of the log type::

    unit(StringWriter, 8)        # Returns Writer(8, "")

``Writer`` constructors take two values, the first being the result of whatever calculation you've just performed, the second being the log message -- or value, or whatever -- to add to the log.

### Other Methods ###

* ``getValue()``: Returns the result and log as a two-tuple.
* ``getResult()``: Returns only the result.
* ``getLog()``: Returns only the log.

A quick example::

    @curry
    def add(x, y):
        return StringWriter(x + y, "Adding " + str(x) + " and " + str(y) + ". ")

    x = unit(StringWriter, 8) >> add(4) >> add(5)
    print(x.getResult())     # prints 17
    print(x.getLog())        # prints "Adding 8 and 4. Adding 12 and 5. "

In the definition of ``add`, ``StringWriter`` could have also been just ``Writer``.
It's really only necessary to use subclasses when using ``unit``, because ``unit`` checks for the ``logType`` variable.
Otherwise simply giving plain old ``Writer`` a string -- or other monoid type argument -- accomplishes the same thing.
Both ``unit`` and ``bind`` (or ``>>``) convert ``*Writer`` types to plain ``Writer`` but using ``StringWriter`` -- or whatever -- makes your intentions more clear.

State Monad
===========

Unlike most of the other monad types, the state monad doesn't wrap values it wraps functions.
Specifically, it wraps functions which accept a single 'state' argument and produce a result and a new 'state' as a 2-tuple.
The 'state' can be anything: simple types like integers, lists, dictionaries, custom objects/data types, whatever.
The important thing is that any given chain of stateful computations all use the same type of state.

The ``State`` constructor should only be used to create stateful computations.
Trying to use ``State`` to inject values, or even non-stateful functions, into the monad will cause it to function incorrectly.
To inject values, use the ``unit`` function.

Here's an example of using ``State``.
We'll create a little system which can perform addition and subtraction.
Our total will never be allowed to drop below zero.
The state that we'll be keeping track of is a simple count of the total number of operations performed.
Every time we perform an addition or subtraction the count will go up by one::

	@curry
	def add(x, y):
		return State(lambda old_state: (x + y, old_state + 1))

	@curry
	def subtract(y, x):
		@State
		def state_computation(old_state):
			if x - y < 0:
				return (0, old_state + 1)
			else:
				return (x - y, old_state + 1)
		return state_computation

As mentioned, the ``State`` constructor takes a function which accepts a 'state', in this case simply an integer, and produces a result and a new state as a tuple.
Although we could have done ``subtract`` as a one-liner, I wanted to show that, if your computation is more complex than can easily be contained in a ``lambda`` expression, you can use ``State`` as a decorator to define the stateful computation.

Using these functions is now simple::

	x = unit(State, 1) >> add(2) >> add(3) >> subtract(40) >> add(5)

``x`` now contains a stateful computation but that computation hasn't been executed yet.
Since ``State`` values contain functions, you can call them like functions by supplying an initial state value::

	y = x(0)        # Since we're counting the total number of operations, we start at zero.
	print(y)        # Prints (5, 4), '5' is the result and '4' is the total number of operations performed.

Calling a ``State`` function in this way will always return the (result, state) tuple.
If you're only interested in the result::

	x.getResult(0)        # Returns the value 5, the result of the computataion.

Or if you only care about the final state::

	x.getState(0)         # Returns the value 4, the final state of the computation.
