+++
title = "Aliasing Explained (Part 2)"
description = "What is aliasing and why you should care - Continued"
date = "2017-03-12"
categories = ["Dev", "Languages", "C"]
tags = ["c", "c++", "fortran", "aliasing", "optimization"]
draft = true
+++

Second episode.

<!--more-->

## A more complex example

{{<gist nazavode 22326361646ba3cfd24cec3fd9594d49 "example-move-01.c">}}

`gcc -std=c11 –O3 -fstrict-aliasing` (3.4.1 for PowerPC)

{{<gist nazavode 22326361646ba3cfd24cec3fd9594d49 "example-move-01.s">}}

### Memory windows tracking

Let's modify our function adding some unit-scoped arrays:

{{<gist nazavode 22326361646ba3cfd24cec3fd9594d49 "example-move-02.c">}}

`gcc -std=c11 –O3 -fstrict-aliasing` (3.4.1 for PowerPC)

{{<gist nazavode 22326361646ba3cfd24cec3fd9594d49 "example-move-02.s">}}


Memory windows (stripes/data channels):

```
    velocity[0] ---> [1] ---> [2] ---> [N]
    position[0] ---> [1] ---> [2] ---> [N]
acceleration[0] ---> [1] ---> [2] ---> [N]
```

Let's modify our function adding some restricts:

{{<gist nazavode 22326361646ba3cfd24cec3fd9594d49 "example-move-03.c">}}

`gcc -std=c11 –O3 -fstrict-aliasing` (3.4.1 for PowerPC)

{{<gist nazavode 22326361646ba3cfd24cec3fd9594d49 "example-move-03.s">}}

### Overlapped memory windows

Prova

{{<gist nazavode 22326361646ba3cfd24cec3fd9594d49 "example-overlapped.c">}}

Are we breaking the rules here?

Il restrict non dice che l’oggetto puntato è completamente privo di alias, solo che gli indirizzi letti (load) e scritti (store) lo sono.

Non importa che le memory windows siano sovrapposte, l’importante è che gli accessi siano disgiunti (Regola delle Piastrelle©).

## Pointers hierarchy

E’ possibile copiare localmente (outer-to-inner) ed utilizzare il valore dei puntatori restrict: si crea (implictamente) una gerachia di puntatori.

E’ possibile accedere agli oggetti puntati solamente attraverso le foglie della gerarchia. Ogni accesso da un livello più alto è una violazione ed il risultato è indefinito.

{{<gist nazavode 22326361646ba3cfd24cec3fd9594d49 "example-memorywindows.c">}}

Hierarchy:

```
                |---> velocity_x
velocity -------|---> velocity_y
                |---> velocity_z

                |---> position_x
position -------|---> position_y
                |---> position_z

                |---> acceleration_x
acceleration ---|---> acceleration_y
                |---> acceleration_z
```

## What to take away from here

* Always switch on all the warnings your compiler can detect for you, you're
  gonna get some false positives but they are far better than some devilish
  aliasing bug;
* look into the assembly code generated for your hotspots, that is the best and
  quickest way to be sure whether the compiler is doing a god job or not with
  our `restrict` qualifiers and aliasing control flags;
* don't break the *outer-to-inner* rule, that would open the gates of hell;
* remember that *memory windows* can overlap;
* publish at the higher possible level (right in the public API) your assumptions about
  aliasing. If some API parameters must be `restrict`, go for it and highlight
  it in the documentation;
* if you need it, start using the `restrict` qualifier as soon as possible
  during the development. Adding it on an already completed code would need to
  deeply understand its memory access patterns;

The most important thing I've learned is: when writing high performance and
number crunching codes in C (and formal languages in general), **never, ever
blame the compiler if your code turns out to be deadly slow**.


----------

Resources:

* [ISO/IEC 9899:201x][iso-doc]

[iso-doc]: http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1548.pdf
