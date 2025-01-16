# Safe Cppfront
Safe Cppfront is an experiment for a transition towards Safe C++ built on top of Cppfront and its cpp2 syntax. This readme explains the design, but doesn't reflect the current state of the fork which has not implemented all of the design yet.

# Roadmap
The plan is to follow these simple steps:

1. Bikeshed new so-called "viral" keywords for safe and unsafe and perform all necessary restrictions on what can be done in the safe context, severely restricting expressivity.
   1. Introduce `safe` and `unsafe` function coloring
   2. Introduce `unsafe` blocks
   3. Produce diagnostics for unsafe calls
   4. Produce diagnostics for other unsafe operations and unsafe constructs
2. Start working on core language proposals that reintroduce expressivity in the safe context (ex: sean's choice)
3. Start working on library proposals that reintroduce expressivity in the safe context (ex: sean's std2::box)
4. Repeat steps 2 and 3 as often as necessary over many different iterations of the standard (C++26, C++29, C++32, etc.)

This is basically the same recipy that worked quite well for `constexpr`. Step #1 is the MVP to deliver something. It could be delivered extremely fast. It doesn't even require a working borrow checker, because the safe context can simply disallow pointers and references at first (willingly limiting expressivity until we can restore it with new safe constructs at a later time).

This idea was well received by the reddit crowd [over here](https://www.reddit.com/r/cpp/comments/1h7lfo3/can_people_who_think_standardizing_safe_cp3390r0/m0nusgb/). And also [over here](https://www.reddit.com/r/cpp/comments/1ffgz49/safe_c_language_extensions_for_memory_safety/lmv9163/).

# Safe C++ can start with a subset
The interesting thing about starting with a subset is that it's an idea that could be delivered long before we commit to a specific approach to safety that enables references and pointers. One can safely assume that no borrow checker will be standardized in C++26. If it were to happen with C++29, we'd have the opportunity to pull the proposal from the draft for many years to come if a big enough breakthrough in another area justifies it. And some would say C++29 is still very optimistic, which means we'd get until 2032 to pull a first iteration of borrow checking from the draft before we are committed to an approach. But we could already start the transition towards writing safe code using pure value semantics in C++26 at the earliest, or hopefully C++29 at the latest.

# Safe by default
Cppfront allows a mix of Cpp2 and standard C++ source code within the same *.cpp2 file. Cpp2 code is safe by default. Standard C++ is unsafe by default.

Cpp2 functions can be made explicitly `safe` or `unsafe`. To avoid viral annotations, `unsafe` blocks provide an escape hatch that allows safe functions to perform unsafe operations such as calling an unsafe function. Unsafe functions can call anything they like, including safe functions.

Because Cppfront does not currently analyze standard C++ code and simply echoes it back, calling an unknown function from within a safe context is ambiguous:
1. It could be that the function doesn't exist at all (not yet written, typo, etc.)
2. It could be that an include is missing
3. It could be that it is an unsafe standard C++ function

Cppfront usually delegates the first two cases to the C++ compiler that will consume the transpiled standard C++ code. But Safe Cppfront must diagnose the third case. For this reason, Safe Cppfront will bundle all three errors in the same diagnostic. There is no meaningful implementation experience to be gained with regards to a Safe C++ transition by doing the work to disambiguate all three cases.

# Unsafe operations and constructs
The initial proof of concept should disallow these unsafe operations and constructs within a safe context:
1. Calling an unsafe function
2. Calling `std::addressof` or builtin `operator&`
3. Declaring an uninitialized variable
4. Declaring a variable or function parameter of type `T&` or `T*`
5. Declaring a variable with `static` or `thread-local` storage
6. Declaring a variable or function parameter of class type `T` where `T` has at least one of these characteristics:
    1. Has a member variable of trivial type without a default value
    2. Has a member variable of class type `T` without a default value and where `T` does not provide a default constructor
    3. Has a member variable of type `T&` or `T*`
    4. Has a member variable with `static` or `thread-local` storage
    5. Has a base class of type `B` that has at least one of these characteristics
7. Referencing a global variable

This short list is not exhaustive, and is meant to enable exploration of the impacts of safe function coloring on incremental adoption. While disallowing `union`, `reinterpret_cast` and many other unsafe constructs is desirable, and can be done, there is no reason to believe it'll provide further insights for a proposal.

# Dynamic checks
Code within safe contexts would enable bound checks by default. Unsafe blocks would be required

Bounds checking is out of the scope of this project. 

# Cppfront
[Click here for the Cppfront readme](https://github.com/hsutter/cppfront/blob/main/README.md)
