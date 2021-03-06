A simple, naive implementation of Conway's Game of Life in
Standard ML, Python, Clojure, Haskell.

These are not optimal or well-formed solutions to this problem.
They are just the first thing I hacked up. I am just learning
Standard ML, and the Python implementation is pretty much
a direct port of the Standard ML code with minor changes.

Here are the benchmark results for 100000 iterations of life on
a small, 5x5 grid with a little glider flying alone:

Standard ML compiled with MLton:

0.716 seconds (compiled)

Python run with CPython 2.7.2:

26.808 seconds

Python run with PyPy 1.7:

6.415 seconds

Clojure 1.3:

8.561 seconds (not counting JVM startup / Clojure bootstrap)

Haskell with GHC 7.0.3:

28.117 seconds (compiled with -O2 using [Int])

11.935 seconds (compiled with -O2 using UArray Int Int and a single
strictness hint)

1.375 seconds Using Data.Vector (contributed)

Chicken Scheme 4.7.0

15.494 seconds (compiled with -O4)

Stalin Scheme

1.358 seconds (compiled with -On -copt -O2)

Racket v5.1.1 "racket" language (Scheme)

5.656 seconds (compiled "racket" lang)
6.260 seconds (compiled "typed/racket" lang)

SBCL 1.0.50 (Common Lisp)

7.070 seconds

Clozure CL 1.7 (Common Lisp)

13.192 seconds

node.js 0.4.9 (JavaScript)

14.615 seconds

GCC 4.6.1 (C)

0.322 seconds (compiled with -O2)

So the SML/MLton version is about 9 times faster than PyPy and 37
times faster than CPython. Clojure has a decent showing, just a bit
slower than PyPy. Haskell time is comparable with CPython, when compiled
with -O2 using a normal list. I switched out the list for an Unboxed
Array which shaved a few seconds off, and then with a bunch of reading
and experimentation found a single strictness hint that cut the time
roughly in half down to 11.935 seconds which is in the same order though
slower than Clojure and PyPy.

After implementing the solution in Chicken Scheme, utilizing srfi-1 and
srfi-4 to get some functions like "fold" and homogenous vectors, I found
a highly optimizing Scheme compiler called Stalin. It's R4RS compliant,
only, so I had to adapt the code, which just meant implementing "fold"
and "subvector" and using generic vectors. Stalin truly does optimize
brutally, eclipsed only by MLton (by a factor of 2) and being the fastest
dynamic language implementation.

Racket seems to a be a pretty interesting project right now. I had fun
adapting the code to the "typed/racket" variant which uses
static/inferred typing. There were no speed gains from the conversion,
but in my mind the usefulness of static type verification is in program
correctness and ease of development. The type errors emitted by the
compiler were extremely easy to understand and fix. Racket performed
admirably, too.

Common Lisp has some nice things like DESTRUCTURING-BIND which works a
bit like native destructuring in Clojure. LABELS replaced LETREC in
Scheme. SBCL and Clozure seem to handle the recursive loop fine. MAP
handles vectors as well as lists. Lisp-2 is annoying with sharp-quoting
function names and having to use APPLY. Common Lisp solution ended up
being short: around 50 lines, like Haskell, Python, Clojure. Common Lisp
seems like a much stabler target than all the various Scheme versions
and implementation quirks.

JavaScript is a terrible language for doing math. I had to write a
modulo function that imitates the way modulus works in every other
language (except C) so far and I had to do a weird hack to get integer
division working. It's unnerving doing integer math with real numbers.
The language is fairly verbose. I expected V8 to perform similarly to
PyPy but the results were disappointing. However, it's probably because
of the math hackery in the inner-most loops that's slowing it down.

Implementing this in C reminds me why I don't write go out of my way to
write code in C. Without support for lambdas and recursion, I
compromised the algorithm the most with the C implementation, using
plenty of for loops, separating nested functions into separate top-level
functions and passing around variables that otherwise were just captured
in the environment of the closures in all other implementations (so
far). This was the most verbose solution and I even "cheated" by relying
on Boehm GC for garbage collection instead of doing it manually or
writing it to run in constant-space on the stack. As usual, C
performance blows away everything else, beating even MLton by a factor of 2.
It may be the fastest, but I felt like the compiler had me doing most
of the optimization by hand.

Found a webpage with some similar benchmarks:
http://www.ffconsultancy.com/ocaml/ray_tracer/languages.html

TODO:

Ocaml
Numpy
