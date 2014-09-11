Beautiful C++ Code
==================

Stop using macros -- we have inline functions, and compilers are smart enough
to inline things when it might be a smart thing to do.

Don't write walls of code -- separate into different functions. Combine
related items into a class.

Use `lambda`s like inline functors, or to encapsulate code that uses
exceptions.

Take advantage of destructors, RAII, smart pointers for cleanup. Doing it by
hand is always dangerous, especially with exceptions.

Don't write useless comments. Don't write comments that repeat exactly what
the code tells you.

Unit Tests
==========

Unfortunately, not that exciting :( Mostly a marketing pitch for Cevelop +
CUTE.

Probably should just use [Catch](https://github.com/philsquared/Catch). Mainly
I like it because it looks very similar to Hadley's `testthat`.

Parallel
========

Scientific compution now drives most hardware innovations. Currently --
parallel architectures.

Performance vs. Expressiveness tradeoffs -- it becomes difficult to be
expressive in performance-constrained scenarios.

Solutions that accommodate parallel programming should be non-disruptive (ie,
appear relatively familiar to copmetent programmers)

Design libraries as **Domain Specific Embedded Languages**.

Doing parallel programming is not easy. Use a model (framework?) that
pre-solves parallel programming problems (ie, write code to fit a model that
is known to solve certain facets of parallel programming)

### Data Parallel Skeletons

- map: Apply an `n-ary` function in SIMD mode over data
- fold: Perform reduction over data
- scan: Perform prefix reduction over data

### Task Parallel Skeletons

- par: Independent task execution
- pipe: Task dependency over time
- farm: Load balancing

Goal: write functions without worrying about parallel details.

Use C++ meta-programming to write generative programs.

### DSEL -- Embedded DSL

- Abuse operator overloading (Expression Templates)
- Carry semantic information around with templates
- Generic implementation can become aware of optimizations

> Heard you like languages -- so I put a language in your language so you can
> compile while you compile.

Generate expressions (meta-AST) and translate it to optimizable code.

### Examples

- BSP++
- Quaff
- Boost.SIMD
- NT2: MatLab for C++

NT2 has many more bindings available than other competing numerical libraries.

### Workflow

1. Take MatLab `.m` file and copy it to a `.cpp` file,
2. Do some cosmetic changes,
3. Compile,
4. Profit!

A Very Crazy Idea. Replace `;` with `,` so that we can construct gigantic
expressions which gives more space for the compiler to work. Not practical due
to size of symbols (megabytes!) and inability to handle exceptions.

### Move from Skeletons to Actors

Provide a facility for giving pipes / dependency graphs between various
statements and expressions.

### Sigma-Delta Motion Detection

Motion detection by background subtraction. Highly parallelizable.

### Black and Scholes Option Pricing

Faster, but not spectacular -- 4 separate statements.

Use `tie` function to tie variables to expressions.

    tie(x, y, z) = tie(ex, ey, ez);

Using `tie` magically makes things ~2x faster!

### LU Decomposition

Benefits greatly from multiple cores.

Wish -- lazy evaluation built into C++ as a language-level construct.

Standardize SIMD computation? `ast_of` operator? Customization for `auto`?
