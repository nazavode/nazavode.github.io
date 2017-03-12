+++
title = "Visitor pattern and Python: dynamic dispatch revisited"
description = "Implementing the Visitor pattern in Python augmenting the standard `singledispatch` decorator"
categories = ["Development", "Python"]
tags = ["python", "design patterns"]
date = "2015-12-19T12:59:50+01:00"

draft = true
+++

Well, everyone knows that *GOF design patterns* are a set of agnostic concepts, fairly independent from any actual
language. True, but they have been designed with a specific language in mind, trying to cope with some of its
shortcomings.
When you think about patterns in Python, some of them make less sense (the [Factory][factory-pattern] is actually
idiomatic) while others turn out to be less straightforward: the *Visitor* is one of them.

## The Visitor pattern

The point of the [Visitor][visitor-pattern] is to cope with a hierarchical structure of types where you want all of them
to support some new behavior. This is especially valuable when the types have been defined by someone else in a piece of
code that you can't (or you're not supposed to) change. Moreover, even if you *could* change it, you *shouldn't* just
because *isolating data from algorithms is always a good thing* and prevents you ending up with *behemoth classes* with
a countless number of operations (also the *S* in [SOLID][SOLID] is a thing *you want* in your design).

Let's assume we have a trivial object hierarchy:

{{<gist nazavode a7fd359bee09cf6d1078955cfef9d80b "ast.py">}}

This simple hierarchy leads to something like this when `pyreverse`-d:

![hierarchy](/images/visitor-ast-hierarchy.png)

*Note: since I'm currently spending my spare time on graph isomorphisms, the hierarchy I'm working on is obviously
graph-oriented but, hey, `Node`, `Edge`, `Graph`...the concepts are the same.*

## The Visitor in Python

### Explicit

{{<gist nazavode a7fd359bee09cf6d1078955cfef9d80b "visit-explicit.py">}}

### Lexical

A more somehow *refined* approach is the one adopted in the [Python `ast` module][visitor-python-ast] relying
on lexical lookup based on actual *names* of types to be visited:

{{<gist nazavode a7fd359bee09cf6d1078955cfef9d80b "visit-lexical.py">}}

This solution works pretty well but it's far from perfect since it prevents any kind of subclassing of node types.

### Dispatch table

Another widespread variant, [even adopted by Bruce Eckel][visitor-python-eckel] in the Python flavour of his famous
*Thinking in* series, is based on a dictionary acting like a *dispatch table*, associating types and visitor functions
directly:

{{<gist nazavode a7fd359bee09cf6d1078955cfef9d80b "visit-dispatchtable.py">}}

Look at that dictionary and think about other languages: what we are doing here is trying to write *by hand*
the [dynamic dispatch][dynamic-dispatch] mechanisms that some strongly typed languages know very well,
 *that `dict` is actually trying to become a [virtual table][cpp-vtable]*.


### The `singledispatch` decorator

{{<gist nazavode a7fd359bee09cf6d1078955cfef9d80b "visit-singledispatch.py">}}

**Nope.** What we are getting here is type dispatching on the first argument, and for bound methods the first argument
is `self`. No way out, we are going to dispatch on the type of `self` and I really can't figure out any usage that makes
sense to me.

## Improving the `singledispatch` decorator

Apart from methods and functions, I came up with this idea that **having a dispatch decorator that allows to specify a
single custom argument among formal parameters would have been nice**.

While googling around, I stumbled upon [this great post by Chris Lamb][chris-lamb-dispatchon] where he presents an
interesting approach (without providing actual code) that matches exactly what I was thinking about. Yep, that looked
brilliant, so I decided to write something similar by my own.

Let's assume we want to be able to write something like this:

{{<gist nazavode a7fd359bee09cf6d1078955cfef9d80b "visit-argdispatch.py">}}

Apart from looking pretty nifty, a thing like that could be a tool available for a lot of other cases too, even for
abstract base classes:

```python

# ABC

```

The implementation is fairly straightforward: while the actual type dispatching is still carried out by the underlying
standard `singledispatch`, the `argdispatch` decorator produces a closure that picks the custom argument instead of the
first one found in the callable signature. This is quite powerful and its main value is that allows to ignore the
`this` argument on bound methods, enabling its adoption in class methods, instance methods and abstract base classes as
well.

You can find the `argdispatch` implementation [on GitHub][argdispatch-github].

## Disclaimer

I know that, as discussed quite extensively [here][abc-support], the whole thing smells a lot of anti-pattern:

> OO and generic functions are different development paradigms,
> and there are limitations on mixing them. Generic functions are for
> stateless algorithms, which expect to receive all required input
> through their arguments. By contrast, class and instance methods
> expect to receive some state implicitly - in many respects, they
> *already are* generic functions.

...instance methods *are* single dispatch, actually. In fact, considering `self` as the first dispatch argument, our own
argument triggers a **double dispatch**.

> Thus, this is really a request for dual dispatch in disguise: you want
> to first dispatch on the class or instance (through method dispatch)
> and *then* dispatch on the second argument (through generic function
> dispatch).

I want to stress again the point that doing *method dispatching* must be considered a Python bad practice since it's
actually an attempt to negate *duck typing*. If you're considering this kind of approach, think about it two, three,
*ten times*: in the vast majority of situations it's just a clear evidence of bad design.

Despite that, the `singledispatch` decorator is now [standard][singledispatch] and it looks like a neat way to cope with
some specific situations ([trees](https://en.wikipedia.org/wiki/Abstract_syntax_tree) anyone?) therefore I'm giving
[my extension][argdispatch-github] a try keeping in mind to avoid evil (and useless) things.

[factory-pattern]: https://sourcemaking.com/design_patterns/factory_method
[visitor-pattern]: https://sourcemaking.com/design_patterns/visitor
[SOLID]: https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)
[cpp-vtable]: https://en.wikipedia.org/wiki/Virtual_method_table
[dynamic-dispatch]: https://en.wikipedia.org/wiki/Dynamic_dispatch
[multiple-dispatch]: https://en.wikipedia.org/wiki/Multiple_dispatch
[visitor-python-eckel]: http://python-3-patterns-idioms-test.readthedocs.org/en/latest/Visitor.html
[visitor-python-ast]: https://docs.python.org/3.5/library/ast.html#ast.NodeVisitor
[abc-support]: http://code.activestate.com/lists/python-dev/122554/
[dict-dispatch]: http://codereview.stackexchange.com/questions/7433/dictionary-based-dispatch-in-python-with-multiple-parameters
[chris-lamb-dispatchon]: https://chris-lamb.co.uk/posts/visitor-pattern-in-python
[singledispatch]: https://docs.python.org/3/library/functools.html#functools.singledispatch
[argdispatch-github]: https://github.com/nazavode/argdispatch
