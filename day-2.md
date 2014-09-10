The Joint Strike Fighter Coding Standard
========================================

C++ and Lockheed Martin. How can we write C++ code in a safe enough way to use
on airplanes (e.g. the Joint Strike Fighter)?

### Ada vs. C++

Seems strange to abandon
[`ada`](http://en.wikipedia.org/wiki/Ada_(programming_language) in favour of
the 'expert-friendly' language `C++`.

- Ada tools in decline
- C++ tools improving
- C++ attractive to engineers

### C++ vs. C

C++ advantages over C for safety:

- Stronger type checking
- Templates over macros
- Containers over C arrays
- Smart pointers for memory management

### Constraints

- Hard Real Time
- - Failure to meet schedule == fatal error

- Limited Memory
- - No memory paging -- need more guarantees that memory allocation succeeds

- Safety
- - Failure == loss of life, aircraft

- Portability
- - Must run in several operating systems (simulators on Linux, Visual Studio,
  etc.)

- Maintainability
- - Code will be maintained for decades
- - Static errors vs. dynamic errors

### Notable Limitations

- No exceptions! (Yay!)
- No RTTI! (Dangerous tool)
- Allocation prohibited except during initialization
- - Less fragmentation
- - Class-specific allocators must be used to avoid fragmentation

### Raw Pointers are Bad

- Use RAII, use shared pointers.
- Proper use of RAII implies that compiler-generated copy constructor,
  assignment operator, destructor should 'just do the right thing'.
- Mutexes should also be wrapped in RAII classes (`std::lock_guard<>`)

### No Arrays

The use of arrays in interfaces **is prohibited** in JSF++!

- Pointer decay makes interfaces confusing for what could ostensibly arrays of
  constant size.

- Prefer `Array<T, N>` types

- Template wrapper functions wrap over legacy interfaces

### Multiple Inheritance

Allowed specifically for [`policy-based
design`](http://en.wikipedia.org/wiki/Policy-based_design).

Use for public interface inheritance, or private implementation inheritance.

### Fault Handling

Functions should return 'error information', not exceptions.

Prefer factory functions over constructors if more explicit exception handling
is required.

### tldr

**RStudio is ready for the JSF.**

Keep Simple Things Simple
=========================

> We like to write clever code, but we don't like to maintain clever code.

Design for clarity -- minimize complexity. Don't over-abstract, hide
complexity, provide good interfaces.

Often best to go from concrete to abstract, instead of abstract to concrete --
more likely that you preserve the actual original solution.

Simple means different things to different people.

Use algorithms from the standard template library when possible.

Talked about checking if container has member: `std::find_if(...) != c.end()`.
Can't help but think, "why not just have a `contains` function, or method?"
E.g.: `if (contains(c, v) { ... })`, or `if (c.contains(v)) { ... }`...
I think C++ people love iterators too much.

In other words -- I don't care how a container is implemented. I can iterate
over a container? That's nice, but it is not relevant to the question "does
this container have this object?". I don't want to know where an iterator ends
up, I just want to know "does it have the object".

### Templates have Problems

Noisy, boilerplate-y, odd/different, bad errors. Want to make 'generic
programming' feel like 'orginary programming'.

### Remedies

- `constexpr` functions preferred when appropriate
- Use variadic templates
- Constrain template arguments

Concepts will be nice. But don't overdo it.

Modern Template Metaprogramming: A Compendium, Part 1
=====================================================

Main ideas:

1. Define 'primary' template, giving default behaviour,
2. Specialize for 'base' cases, or 'pattern-matching' specializations.
3. Favor `f<T>::type` for types, `f<T>::value` for values.
4. Take advantage of SFINAE to handle substitution failures.

Compile time absolute value *metafunction*:

    template <int N>
    struct abs {
        static_assert(N != INT_MIN);                      // C++17-style guard
        static constexpr int value = (N < 0) ? -N : N;    // "return"
    }

### Usage

Metafunction arguments are supplied as __template__ arguments,

"Call" syntax is a request for the template's published **value**.

    int const n = ...;
    ... abs<n>::value;

Metafunctions as **struct**s give us more tools, e.g.

- Public member type declarations (`typedef`, `using`),
- Public member data declarations (`static const/constexpr`)
- - Initialized by constant expression
- Public member templates, `static_assert`s, and so on.

Function declarations can be useful without definitions.

Compile time recursion! *head explodes*

    template <unsigned M, unsigned N>
    struct gcd {
       static int const value = gcd<N, M % N>::value;
    };

    template <unsigned N>
    struct gcd<N, 0> {
       static_assert(M != 0);
       static int const value = M;
    }

#### Identity Metafunction

    template <typename T>
    struct type_is { using type = T; }

    // partial specialization recognizes volatile-qualified types;
    template <typename U>
    struct remove_volatile<U volatile> : type_is<U> {};

#### SFINAE Example

    template <class T>
    enable_if< is_integral<T>::value, maxint_t >::type f(T val) { ... }
    
    template <class T>
    enable_if< is_floating_point<T>::value, long double>::type f(T val) { ... }

Concepts would obviate some of this, e.g.

    template <Integral T>
    f(T val) { ... }

### void_t

Enables an incredibly useful property: ask if class has member variable by
name !!

See [N3911](http://isocpp.org/files/papers/N3911.pdf) for details.

    template <class, class = void>
    struct has_type_member : false_type { };

    template <class T>
    struct has_type_member<T, void_t<typename T::type>> : true_type { };

TODO: What about doing something more generic; e.g. `has_member<T, U>` for
some type `U`?

0xBADC0DE
=========

C++ is expert-friendly, but not everyone writing C++ is an expert (or even
cares about the quality of their code). But domain-specific experts do not
have the time to become C++ experts -- we should still do our best to educate
on best practices, and refactor as necessary.


