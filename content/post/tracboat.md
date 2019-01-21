+++
title = "Salvaging 10 years of Trac while moving to GitLab: tooling for a better life"
description = "Tracboat, a trac-to-gitlab lifeboat"
categories = ["Dev", "Tooling"]
tags = ["python", "gitlab", "trac", "migration", "devops"]
date = "2017-04-11"
+++

Some time ago at the office we began considering a complete makeover of our devops infrastructure.
The platform in use was based on **[Trac][trac-home]**, a tool that ten years ago proved itself packed
with features, looked great and came in the form of an open, clear and easily extensible Python code
base. Almost **a mandatory choice back then**. It served us *very* well (thanks Trac guys!), even when
it had to run on underpowered hardware, without maintenance and under heavy loads. Despite its stability
and practicality, the need for a proper, *modern* devops platform seamlessly supporting continuous
integration and deployment pipelines, as well as code review, suddenly came out. **The choice went
for [GitLab][gitlab-home]**, a solution that would have given us the same degree of freedom
(free community edition, open source, easily manageable on premise) along all the shiny new
features we were looking for.

At that point, **the issue of how to migrate all of the content came out**.

<!--more-->

At the time every team used the same Trac instance as a *knowledge base*, with thousands
of wiki pages, GBs of attachments and **ten years of bug hunting chronicles**.

The boss was *really scared*. To be honest, we were all scared at the idea of loosing
something in the migration process.

Even the remote chance of loosing a single bit of our history was not acceptable and a
risk we weren't keen facing at the point that leaving everything as it was untouched just
out of fear was *real*. In the face of the risk of giving up on the makeover, I decided I
had to do something. The prospect of such a bump in life quality for the whole team was
enough for me to invest one weekend in crafting some tool that would have left no
objections on the table and hopefully all of us with a shiny, new devops platform.

**I came up with [Tracboat][tracboat-home]**, a **command line tool** written in Python aimed at
**migrating an entire Trac instance to GitLab**. It's obviously not perfect: it is has been
written in a rush, it's slow
(no [XML-RPC multicalls](https://docs.python.org/3/library/xmlrpc.client.html#multicall-objects)),
with little unit testing and the code base definitely needs some love and refactoring,
but *it works*. And **it worked extremely well for our own migration**.

You can do essentially two of things with the tool:

1. **export a Trac instance as a whole in a single `json` file** (maybe piping it into some
   compressor, `json` can grow very fast in size);
2. **import a previously exported instance into a fresh GitLab instance**.

It will take care of **attachments** by uploading them and updating links, **translate wiki
pages** from Trac format to markdown and **map Trac users to new GitLab accounts**
(creating them from scratch when needed).
By combining those basic operations you can, for example, migrate a Trac instance
directly into GitLab without having to rely on a intermediate local file.

Unfortunately the lack of spare time forced me to set back any further development
despite the lots of people coming out asking for fixes and new features (yes, a lot of
folks are still using Trac in 2017), so I decided to move the
[original repository](https://github.com/nazavode) to a
[brand new organization](https://github.com/tracboat) and accept the help offered by
the community.

## Credits

The originating idea was taken from [trac-to-gitlab](https://github.com/moimael/trac-to-gitlab)
by [Maël Lavault](https://github.com/moimael).

A lot of work has beek taken over by [Elan Ruusamäe](https://github.com/glensc) that is now
the most active maintainer (thank you!).

[gitlab-home]: https://gitlab.com/
[trac-home]: https://trac.edgewall.org/
[tracboat-home]: https://github.com/tracboat/tracboat