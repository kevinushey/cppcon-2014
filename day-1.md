CppCon 2014 -- Day One
======================

Type Deduction and Why You Care -- Scott Meyers
-----------------------------------------------

Type deduction is no longer simple. Different type deduction rules are used for
different contexts, and Scott discussed the different forms:

1. `auto`,
2. `decltype`,
3. lambda capture, e.g. `[x] { ... }`,
4. lambda init, e.g. `[x = x] { ... }`,
5. 'Universal references', e.g. references to templated type paramters,
6. `decltype(auto)`

Important to know about, but details are mainly important to library designers.
Clarified a number of things.

Also discussed Scott Meyers lookalikes, laughed very heartily at the He-Man
comparison.

Emscripten and asm.js: C++'s role in the modern web -- Alon Zakai
-----------------------------------------------------------------

Overview of the tools used to transform C++ into JavaScript for fast execution
of the web.

    C++ -----> LLVM ----------> asm.js
        clang       emscripten

Very high level overview of [`asm.js`](http://asmjs.org/), a very optimiziable
subset of JavaScript emitted by
[`emscripten`](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Emscripten).
Detailed how JavaScript is now the assembly language of the web.

Some interesting discussions on some of the difficulties in compiling  C++ to
JavaScript, which are very, very different languages. Mainly uses tricks to
enforce types which can be statically inferred for JavaScript code, and also
use of [typed
arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)
for fast lookup.

Lock-Free Programming (or, Juggling Razor Blades) -- Herb Sutter
----------------------------------------------------------------

Some ideas on how to implement concurrent programs in C++ with the use of
locks, locks and atomics, and just atomics. Very interesting discussion on
'task distribution' model as a post office -- producers fill mailboxes,
consumers look into mailboxes and do things with mail if available. Discussed
different strategies for how this work should be distributed. Some key concepts:

> Use [`std::lock_guard`](http://en.cppreference.com/w/cpp/thread/lock_guard)
> to provide locks within a scope. The destructor can guarantee the lock is
> unlocked at the end (avoiding pitfalls with `.lock()`, `.unlock()` code)

> Understand the critical sections: what work is being done under the protection
> of a lock, and is it desirable for that work to be done within the scope of a
> lock? The question does not always have a simple answer.

> It's okay for one worker to do a bit more work, if it means less work for
> everyone else.

> Always measure performance, because it's so easy for our intuitions to be
> wrong.

> Ask the standards committee to kindly provide an atomic shared pointer.

Also gave a very wonderful piano introduction to each part of his talk.

Efficiency with Algorithms, Performance with Data Structures -- Chandler Carruth
--------------------------------------------------------------------------------

In short: use `std::vector`, everything else sucks because they either are
linked lists, or use linked lists under the hood. Cache locality is of utmost
importance in achieving performant data structures.

Lamented the lack of a good hashmap implementation in the C++ standard library.
Suggested one built without linked lists and buckets would be preferable.

Also, two simple examples on some basic components:

1. Pre-allocate when possible,
2. Use temporary references when appropriate, rather than repeating a
   (potentially expensive) computation multiple times.

Also drove home the [Named Return Value
Optimization](http://en.wikipedia.org/wiki/Return_value_optimization) point a
lot, and that compilers are now all smart enough that this:

    std::vector<std::string> f(...) {
        std::vector<std::string> vec;
        // fill vec
        return vec;
    }

is just as fast, and more readable, than

    void f(..., std::vector<std::string>* vec) {
        // fill vec
    }

Algorithmic complexity is important, but often misleading -- e.g. bubble sort
is often the best way to sort a very small vector. Need to bring theoretical
concerns in line with reality of the CPU architecture / instructions being
executed.
