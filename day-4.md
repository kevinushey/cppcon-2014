Quick Game Development with C++11/C++14
=======================================

> Vittorio Romeo

Focused on techniques used in C++ game programming. Very example driven (with
lots of on-screen code), which is very nice to see. Some notable bits:

- Lots of use of 'curly-brace' construction / initialization,
- Liberal use of `constexpr`,
- Functions are declared `noexcept` wherever possible.

E.g. when ball hits brick -- don't immediately destroy it. Mark a flag that
would set it as destroyed, and let other machinery handle it.

Lots of thinking about what constitutes a collision: e.g. rectangle vs.
rectangle, circle vs. rectangle, box models, etc...

`static constexpr` is the new `static const`. Nice to have static elements
that might execute some code at compile time.

Side note: damn is sublime text pretty -- especially the fluid scrolling.

Side note: I **hate** STL idioms that combine operations in the standard
library to do 'common' operations. E.g. the [`Erase-remove
idiom`](http://en.wikipedia.org/wiki/Erase-remove_idiom): Can we just have a
function, please?

Data-Oriented Design and C++
============================

> Mike Action

Intro: games is hard! deadlines, performance, usability, debuggability...

In games:

- No STL
- Lots of custom allocators

Match mental models to actual computations -- e.g. 'look up value by key'.

Solve the most common case first -- then make it generic.

Cache locality matters! Cache misses suck!

The compiler cannot reason about the fast majority of problems. It is not a
magic wand. We need to be thinking about the cache and how we can fill the
cache.

Pack `bools` to use the cache. Organize data members to think about how items
fit in the cache. Write code to help the compiler -- don't ask "why can't the
compiler do that?" and help the compiler do it.

Rule of Thumb: Store each state type separately; store same states together.

Avoid last minute decision making.

Convert switch-case to separate functions (compile-time switching?)

Ultimately:

- Uses a very limited subset of C++ (it's basically C); prefers external tools
  for metaprogramming over C++ templates

- Important to understand how the cache is used, have some understanding of
  what code the compiler will generate, understand how we can 'help' the
  compiler generate optimal code.

- Talk was somewhat controversial in this regard.

STL Features and Implementation Techniques
==========================================

> Stephan T. Lavavej

`operator ""` for user defined literals. Yay!

Inline namespace == drag things inside to be members of outer namespace.

Coming -- STL algorithms taking pairs of start, end iterators (rather than
range and a half)

Discussion of tags for providing overloads (remember -- don't specialize
function templates, overload them!)

Allocator interface has improved in C++11 (minimal allocator interface)

`construct()` should be removed from custom allocators (C++03 allocator
`construct` cannot be moved). C++11 will do perfect forwarding for you.

C++14 solves problem with `find()` -- we no longer have to construct
(potentially large) objects in order to `find()`. But it's not enabled by
default. Use:

    map<K, V, less<>>

to enable overload.

Or, make the type `C::is_transparent`. See:
http://stackoverflow.com/questions/20317413/what-are-transparent-comparators

Interesting method for avoiding type deduction problems. This doesn't work:

    template <typename U, typename C::Meow>

while this does:

    template <typename U, typename C2 = C, typename Dummy = typename C2::Meow>

STL avoids storing pointers to temporaries.

    foo (const X&);           // takes modifiable/const lvalues,
    foo (const X&&) = delete; // rejects modifiable/const rvalues

Templating is also okay.

auto&&
------

Prefer `(const auto& elem : range)` over `(auto elem : range)`. Even for
`int`s! (?? really?)

C++17: `for (elem : range) { ... }`. Don't even use `auto`!

Cast to `void` to avoid people who do terrible things `operator,`.

Mo' Hustle Mo' Problems
=======================

> Andrei Alexandrescu

Compiler's Most Vexing Inlining
-------------------------------

Inline isn't always better. Can thrash the I-cache.

Fix on `gcc`:

    --max-inline-instns-auto=100
    --early-inlining-instns=200

Transitive effects of constructors / destructors can be bad.

Instruction cache (I-cache) spills can be nasty (caused by excessive inlining)

### Beware Inline Destructors

- Called _everywhere_, implicitly
- Often generated automatically
- Watch dtor size carefully

Consider writing destructor in such a way that it cannot be inlined (ie, in
impl file)

### Faster Smart Pointers

- Hide reference count in the pointer. Gross but workable!

Note: Many refcounts are 0 or 1. Can we take advantage of this?

### Lazy Refcount Allocation

Will have to get slides -- fairly deep concepts...

Indirect writes are bad -- do everything in registers.

### Skip Last Decrement

- Most objects have low refcounts
- Last refcount decrement high percentage-wise
- Avoid dirtying memory on moribund objects
- Replace interlocked decrement with atomic read
- - On x86, _all_ reads are atomic!

### Prefer Zero

Compare against zero -- smaller instructions! Zero is special to the CPU.

- Special assignment
- Special comparisons

E.g., in `enum`, make `0` the most frequent value.

Make default state all zeros. Initializing to zero == very simple.

### Use Dedicated Allocators

- No generic allocator handles small allocs well
- Keep refcounts together

### Use Smaller Refcounters

- Don't worry about leaks for objects with many refs
- Vast majority of objects have < 8; < 16 refs

Micro-optimization means reaching far, scraping and doing things that maybe
you shouldn't do. But do it only when you really, really need to.

Ubisoft Parallel Games
======================

> Jeff Preshing

Game industry + C++ community have been approaching parallel programming in
different ways. Bring the communities together `<3`

Single threaded main loop
-------------------------

> Engine -> Graphics -> Engine -> Graphics -> ...

Threading patterns at Ubisoft:

1. Pipeline work (2 threads working in lockstep with eachother)
2. Dedicated threads (one thread does thing)
3. Task scheduling

Concurrent object == object handled / modified by multiple threads.

Platform primitives vs. atomic operations.

> Engine -> Engine    -> Engine
>       |-> Graphics |-> Graphics

How to avoid concurrent modifications?

1. Double-buffer graphics state relevant to both threads. Two copies of state,
   each thread knows which copy it's working with. Alternate.

2. Separate graphics objects from engine objects. Copy state at beginning of
   each frame.

Content Streaming
-----------------

E.g. single dedicated thread loading the environment (don't want to load the
entire environment, only what is visible)

Other threads ask this thread to produce world and then retrieve it.

Use thread-safe queue. Make sure it can sleep when un-needed.

Ubisoft uses platform-specific events, rather than locks, for signalling (so
threads can sleep / wake up)

Also want to improve the queue -- cancel requests, interrupt requests,
re-prioritize requests...

Task Schedulers
---------------

Can run many operations in parallel -- so have a scheduler distribute work on
threads / tasks. Use concurrent queue again (don't care which thread pulls it
out, as long as someone does)

Organize tasks into groups -- allow multiple threads to work on task group.

Use `volatile` to indicate a variable might be changed at any time.

Task groups need something more than a queue -- queue with separate tails for
each worker? Give each worker its own queue? Many variations. But need custom
concurrent objects / strategies.

Task groups let you develop dependency graphs for execution.

Each project might use its own task scheduler.

... computer died here :( ...

Post-cap:

C++11 atomics are basically made up of two libraries -- one low-level, like
C++ volatile, and one higher level, like Java volatile. Outlined the use of
fences, different kind of memory store / load semantics / ordering, and
essentially advocated for the 'regular' use of atomics as safe (but tunable if
performance there ever became a concern).

