+++
title = "Visitor pattern and Python: dynamic dispatch revisited"
description = "Implementing the Visitor pattern in Python augmenting the standard `singledispatch` decorator"
categories = ["Development"]
tags = ["python", "design patterns"]
date = "2015-12-19T12:59:50+01:00"

draft = true
+++

Well, everyone knows that *GOF design patterns* are a set of agnostic concepts, fairly independent from any actual
language. True, but they have been designed with a specific language in mind, trying to cope with some of its shortcomings.
When you think about patterns in Python, some of them make less sense (the [Factory][factory-pattern] is actually
idiomatic) while others turn out to be less straightforward: the *Visitor* is one of them.

The point of the [Visitor][visitor-pattern] is to cope with a hierarchical structure of types where you want all of them
to support some new behavior. This is especially valuable when the types have been defined by someone else in a piece of
code that you can't (or you're not supposed to) change. Moreover, even if you *could* change it, you *shouldn't* just
because *isolating data from algorithms is always a good thing* and prevents you ending up with *behemoth classes* with
a countless number of operations (also the *S* in [SOLID][SOLID] is a thing *you want* in your design).

Let's assume we have a trivial object hierarchy:

```python
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
```
This simple hierarchy leads to something like this when `pyreverse`-d:

![hierarchy](/images/visitor-ast-hierarchy.png)

*Note: since I'm currently developing a reactive port graph calculus framework, the hierarchy I'm working on is
obviously graph-oriented but, hey, `Node`, `Edge`, `Graph`...the concepts are the same.*

```python
def visit(car):
    if isinstance(car, Trabant):
        pass
    if isinstance(car, Trabant):
        pass
    if isinstance(car, Trabant):
        pass
```

Multiple dispatching.

Visitor in Python:

1. isinstance
2. dispatch table - Look at that dictionary and think about other languages: what we are doing here
  is trying to write *by hand* the [dynamic dispatch][dynamic-dispatch] mechanisms that some
  strongly typed languages know very well,
  **that `dict` is actually trying to become a [virtual table][cpp-vtable].**

```python
__dispatch_table__ = {
    Volga: '',
    Trabant: '',
    FiatPanda750: '',
    SeriousCar: '',
}

def reverse_engineer(car):

```

3. singledispatch
4. visitor class: singledispatch as external function

Overload.


While googling around, I stumbled upon [this great post by Chris Lamb][chris-lamb-dispatchon] and I decided that his
approach was brilliant and worth a try.


```python
class GraphVisitor(object):

  @argdispatch('element')
  def visit(self, element):
    pass

  @visit.register(Node)
  def _(self, element):
    pass
```

You can find the `argdispatch` decorator [on GitHub][overload-github].

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
[my extension][overload-github] a try keeping in mind to avoid evil (and useless) things.

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
[overload-github]: https://github.com/nazavode/overload
