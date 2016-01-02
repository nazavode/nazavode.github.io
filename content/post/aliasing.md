+++
categories = ["Lectures", "C"]
date = "2016-01-02T00:13:08+01:00"
description = ""
tags = [""]
title = "Aliasing explained"

draft = true

+++

Since my job involves mainly HPC stuff and *performance-obsessed code*, one of the first topics I had to dive into as
a developer at [Cineca](http://hpc.cineca.it/) was aliasing in the C language.

I've just [uploaded][speakerdeck] a lecture I held some time ago to recap what I had discovered about it

The most important thing I've learned is: when writing high performance and number crunching codes in C (and strongly
typed languages in general, like C++ or Fortran), **never, ever blame the compiler if your code turns out to be
deadly slow**.

You can find full presentation materials on [Speaker Deck][speakerdeck].

[speakerdeck]: https://speakerdeck.com/nazavode/aliasing
