+++
title = "Tooling for a better life"
description = "A trac-to-gitlab migration toolbox"
categories = ["Dev", "Tooling"]
tags = ["python", "gitlab", "trac", "migration", "devops"]
date = "2014-02-20"
draft = true
+++

Some time ago we were considering a complete makeover of our devops infrastructure. It
was based on Trac, a tool that ten years ago was packed with features, looked great and
came in the form of an open, clear and easily extensible Python code base. In short,
almost a mandatory choice back then. It served us *very* well, even when it had to run on
underpowered hardware, without maintenance and under heavy loads.

Everyone used the same Trac instance as a *knowledge base*, with thousands
of wiki pages, GBs of attachments and *ten years of bug hunting chronicles*.


The boss was really scared. To be fair, we were all really scared by the idea of loosing
something in the migraiton process.

Even the remote chance of was not acceptable and a risk we weren't keen in running at the
point that leaving everything untouched just out of fear was *real*.

In the face of the risk of giving up on the makeover, I decided I had to do something.
The prospect of such a bump in life quality for the whole team was enough for me to
invest one weekend in crafting some tool that would have left the boss with no objections
and all of us with a shiny devops platform.

I came up with Tracboat. It is slow (no XMLRpc remote chaining), little unit tests and
the code base definitely needs some love and refactoring, but it mostly works. And it
worked extremely well for our migration.

You can do essentially two of things with it:

1. export a Trac instance as a whole in a single `json` file (maybe piping it into some
   compressor, `json` can grow so fast in size);
2. import a previously exported instance into a fresh GitLab instance;

By combining those basic operations you can, for example, migrate a Trac instance
directly into GitLab without having to rely on a intermediate local file.

Unfortunately the lack of spare time forced me to set back any further development
despite the lots of people coming out asking for fixes and new features (yes, a lot of
folks are still using Trac in 2017).
