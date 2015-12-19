+++
categories = ["Development"]
tags = []
date = "2015-12-19T12:59:50+01:00"
description = "Implementing the Visitor pattern in Python augmenting the standard singledispatch decorator"
keywords = ["python", "visitor", "singledispatch", "type dispatch"]
title = "Visitor pattern and Python: dynamic dispatch revisited"
draft = true
+++

Visitor pattern.

Multiple dispatching.

Visitor in Python:

1. isinstance
2. dispatch table - Look at that dictionary and think about other languages: what we are doing here
  is trying to write *by hand* the [dynamic dispatch][dynamic-dispatch] mechanisms that some
  strongly typed languages know very well,
  **that `dict` is actually playing as a [C++ virtual table][cpp-vtable].**
3. singledispatch
4. visitor class: singledispatch as external function

Overload.

Inspiration from [Chris Lamb][chris-lamb-dispatchon]

I know that, as discussed quite extensively [here][abc-support],
the whole thing smells a lot of anti-pattern:

> OO and generic functions are different development paradigms,
> and there are limitations on mixing them. Generic functions are for
> stateless algorithms, which expect to receive all required input
> through their arguments. By contrast, class and instance methods
> expect to receive some state implicitly - in many respects, they
> *already are* generic functions.

Moreover, the real point is that if you consider `self` as an implicit
first dispatch argument, our own argument triggers a **double dispatch**:

> Thus, this is really a request for dual dispatch in disguise: you want
> to first dispatch on the class or instance (through method dispatch)
> and *then* dispatch on the second argument (through generic function
> dispatch).

**Disclaimer**: I want to stress again the fact that doing *method dispatching* must
be considered a Python bad practice since it's actually an attempt to negate
*duck typing*, the soul of the language. If you're considering this kind
of stuff, think about it two, three, *ten times*: in the vast majority
of situations it's just a clear evidence of bad design.

Despite that, the `singledispatch` decorator is now standard and it looks
like the *neat* way to cope with some specific situations
([AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree) anyone?).

[cpp-vtable]: https://en.wikipedia.org/wiki/Virtual_method_table
[dynamic-dispatch]: https://en.wikipedia.org/wiki/Dynamic_dispatch
[visitor-pattern]: https://sourcemaking.com/design_patterns/visitor
[multiple-dispatch]: https://en.wikipedia.org/wiki/Multiple_dispatch
[visitor-python-eckel]: http://python-3-patterns-idioms-test.readthedocs.org/en/latest/Visitor.html
[abc-support]: http://code.activestate.com/lists/python-dev/122554/
[dict-dispatch]: http://codereview.stackexchange.com/questions/7433/dictionary-based-dispatch-in-python-with-multiple-parameters
[chris-lamb-dispatchon]: https://chris-lamb.co.uk/posts/visitor-pattern-in-python

[overload-github]: https://github.com/nazavode/overload
