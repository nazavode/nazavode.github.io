+++
title = "Visitor pattern and Python: dynamic dispatch revisited"
description = "Implementing the Visitor pattern in Python augmenting the standard `singledispatch` decorator"
categories = ["Dev", "Languages"]
tags = ["python", "design patterns", "visitor", "type dispatching", "singledispatch", "multimethod"]
date = "2015-12-19"

draft = true
+++

Well, everyone knows that *GOF design patterns* are a set of agnostic concepts,
fairly independent from any actual language. True, but they have been designed
with a specific language in mind, trying to cope with some of its shortcomings.
When you think about patterns in Python, some of them make less sense (the
[Factory][factory-pattern] is actually idiomatic) while others turn out to be
less straightforward: the *Visitor* is one of them.

<!--more-->

## The Visitor pattern

The point of the [Visitor][visitor-pattern] is to cope with a hierarchical
structure of types where you want all of them to support some new behavior. This
is especially valuable when the types have been defined by someone else in a
piece of code that you can't (or you're not supposed to) change. Moreover, even
if you *could* change it, you *shouldn't* just because *isolating data from
algorithms is always a good thing* and prevents you ending up with *behemoth
classes* with a countless number of operations (also the *S* in [SOLID][SOLID]
is a thing *you want* in your design).

Let's assume we have a trivial object hierarchy:

{{< highlight python >}}
class AstNode(object):
    pass

class Expression(AstNode):
    pass

class BinOp(AstNode):
    pass

class Add(BinOp):
    pass

class Name(AstNode):
    pass

class Num(AstNode):
    pass
{{< / highlight >}}

This simple hierarchy leads to something like this when feed through `pyreverse`:

![hierarchy](/img/visitor-ast-hierarchy.png)

*Note: since I'm currently spending my spare time on graph isomorphisms, the
*hierarchy I'm working on is obviously graph-oriented but, hey, `Node`, `Edge`,
*`Graph`...the concepts are the same.*

## The Visitor in Python

### Explicit

{{< highlight python >}}
def visit(node):
    if isinstance(node, Num):
        return '[Num: {}]'.format(node)
    elif isinstance(node, Name):
        return '[Name: {}]'.format(node)
    if isinstance(node, BinOp):
        return '[BinOp: {} {} {}]'.format(
            visit(node.lhs),
            visit(node.op),
            visit(node.rhs)
        )
    # continue...
{{< / highlight >}}

### Lexical

A more somehow *refined* approach is the one adopted in the [Python `ast`
module][visitor-python-ast] relying on lexical lookup based on actual *names* of
types to be visited:

{{< highlight python >}}
class OpVisitor(object):

    def visit(self, node):
        fun_name = 'visit_' + node.__class__.__name__
        fun = getattr(self, fun_name, self.visit_default)
        return fun(node)

    def visit_default(self, node):
        return '[{}]'.format(node)

    def visit_Num(self, node):
        return '[Num: {}]'.format(node)

    def visit_Name(self, node):
        return '[Name: {}]'.format(node)

    def visit_BinOp(self, node):
        return '[BinOp: {} {} {}]'.format(
            self.visit(node.lhs),
            self.visit(node.op),
            self.visit(node.rhs)
        )
    # continue...
{{< / highlight >}}

This solution works pretty well but it's far from perfect since it prevents any
kind of subclassing of node types.

### Dispatch table

Another widespread variant, [even adopted by Bruce Eckel][visitor-python-eckel]
in the Python flavour of his famous *Thinking in* series, is based on a
dictionary acting like a *dispatch table*, associating types and visitor
functions directly:

{{< highlight python >}}
class OpVisitor(object):

    def __init__(self):
        self._dispatch_table = {
            Name: self.visitName,
            Num: self.visitNum,
            BinOp: self.visit_BinOp,
        }

    def visit(self, node):
        visit_fun = \
            self._dispatch_table.get(node.__class__, self.visit_default)
        return visit_fun(node)

    # the same as in the previous example...
{{< / highlight >}}

Look at that dictionary and think about other languages: what we are doing here
is trying to write *by hand* the [dynamic dispatch][dynamic-dispatch] mechanisms
that some strongly typed languages know very well, *that `dict` is actually
trying to become a [virtual table][cpp-vtable]*.


### The `singledispatch` decorator

{{< highlight python >}}
class OpVisitor(object):

    @singledispatch('node')
    def visit(self, node):
        return '[{}]'.format(node)

    # ...
{{< / highlight >}}

**Nope.** What we are getting here is type dispatching on the first argument,
and for bound methods the first argument is `self`. No way out, we are going
to dispatch on the type of `self` and I really can't figure out any usage that
makes sense to me.

## Improving the `singledispatch` decorator

Apart from methods and functions, I came up with this idea that **having a
dispatch decorator that allows to specify a single custom argument among formal
parameters would have been nice**.

While googling around, I stumbled upon [this great post by Chris
Lamb][chris-lamb-dispatchon] where he presents an interesting approach (without
providing actual code) that matches exactly what I was thinking about. Yep, that
looked brilliant, so I decided to write something similar by my own.

Let's assume we want to be able to write something like this:

{{< highlight python >}}
class OpVisitor(object):

    @argdispatch('node')
    def visit(self, node):
        return '[{}]'.format(node)

    @visit.register(Num)
    def _(self, node):
        return '[Num: {}]'.format(node)

    @visit.register(Name)
    def _(self, node):
        return '[Name: {}]'.format(node)

    @visit.register(BinOp)
    def _(self, node):
        return '[BinOp: {} {} {}]'.format(
            self.visit(node.lhs),
            self.visit(node.op),
            self.visit(node.rhs)
        )
      # continue...
{{< / highlight >}}

Apart from looking pretty nifty, a thing like that could be a tool available for
a lot of other cases too, even for abstract base classes:

```python

# ABC

```

The implementation is fairly straightforward: while the actual type dispatching
is still carried out by the underlying standard `singledispatch`, the
`argdispatch` decorator produces a closure that picks the custom argument
instead of the first one found in the callable signature. This is quite powerful
and its main value is that allows to ignore the `this` argument on bound
methods, enabling its adoption in class methods, instance methods and abstract
base classes as well.

You can find the `argdispatch` implementation [on GitHub][argdispatch-github].

## Disclaimer

I know that, as discussed quite extensively [here][abc-support], the whole thing
smells a lot of anti-pattern:

> OO and generic functions are different development paradigms,
> and there are limitations on mixing them. Generic functions are for
> stateless algorithms, which expect to receive all required input
> through their arguments. By contrast, class and instance methods
> expect to receive some state implicitly - in many respects, they
> *already are* generic functions.

...instance methods *are* single dispatch, actually. In fact, considering `self`
as the first dispatch argument, our own argument triggers a **double dispatch**.

> Thus, this is really a request for dual dispatch in disguise: you want
> to first dispatch on the class or instance (through method dispatch)
> and *then* dispatch on the second argument (through generic function
> dispatch).

I want to stress again the point that doing *method dispatching* must be
considered a Python bad practice since it's actually an attempt to negate *duck
typing*. If you're considering this kind of approach, think about it two, three,
*ten times*: in the vast majority of situations it's just a clear evidence of
bad design.

Despite that, the `singledispatch` decorator is now [standard][singledispatch]
and it looks like a neat way to cope with some specific situations
([trees](https://en.wikipedia.org/wiki/Abstract_syntax_tree) anyone?) therefore
I'm giving [my extension][argdispatch-github] a try keeping in mind to avoid
evil (and useless) things.

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
