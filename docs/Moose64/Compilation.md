<!-- For license of this file, see LICENSE.md in the base dir. -->

Compilation
===========

A program's code is written in an `.m64` text file, then translated
by [moosec
](https://codeberg.org/Horse64/core.horse64.org/docs/Resources.md#moosec) 
to C, and then turned into a final binary by a C compiler like
`gcc` or `clang`.
The result is a mostly standalone executable.

Other tooling like [horp
](https://codeberg.org/Horse64/core.horse64.org/docs/Resources.md#horp)
helps with project management.


Stages
------

These are the current compilation stages of mooseec as of 2024-06-08:

All stages from `--stage token` up to `--stage transformed-code` (or
the equivalent `--stage transformed-ast`) are the same as found in
[horsec
](https://codeberg.org/Horse64/core.horse64.org/docs/Resources.md#horsec).

The final stages are different:

5. `--stage c` will take the fully resolved and transformed
   AST and generate the resulting C code.

   *(Upcoming maybe at some point, more optimizations here.)*

6. `--stage binary` will write out the final binary as produced
   by whatever C compiler was used.
