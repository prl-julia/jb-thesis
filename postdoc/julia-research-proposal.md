# Title

## Background

Julia (ref) is a dynamic, high-level, yet high-performance programming language,
originally designed for scientific computing.
Julia aims to solve the two-language problem, combining the ease of use of
productivity languages such as Python and R, and performance of languages such
as Fortran and C (ref).

Although Julia is dynamically typed and does not require any type annotations,
types have a profound role in the language semantics and performance.
Thus, Julia is designed around symmetric multiple dynamic dispatch.
Multiple dispatch allows a function to have multiple implementations, called
methods, tailored to different argument types; at run time, a function
call is dispatched to the most specific method for the given arguments.
To deliver performance, Julia relies on a type-specializing just-in-time
compiler: with a few exceptions (ref), every time a method is called with a new
set of argument types, it is specialized for those types.

The key to efficient compilation of Julia programs is dispatch elimination:
dispatched calls are expensive, especially for functions like `(*)` that have
hundreds of methods, and they prevent further optimizations, e.g. inlining.
With type specialization and dispatch elimination, a function like
`mul42(x) = x * 42`
when called with an integer, is efficiently compiled to the following LLVM code:
```
define i64 @julia_mul42_503(i64 signext %0) #0
{ %1 = mul i64 %0, 42
  ret i64 %1 }
```

## Type stability

While Julia can achieve performance comparable to that of C (ref),
to be compiled effectively, programs need to be written in a particular style that
is conducive to optimizations.
Thus, for example, Julia programmers are encouraged to write so-called
_type-stable_ code. Informally, a function is type stable if the type of its
output can be predicted from the input types.
For instance, function `pos(x::Number) = x < 0 ? 0 : x` is not type stable: when
called with a float, it returns either a float or an integer depending on the
_value_ of `x`.

Type-unstable functions impede optimizations of their callers: in particular,
instability prevents dispatch elimination.
For example, a `Float64`-specialized version of
`h(x) = f(pos(x))` can optimize only the call to `pos` but not to `f`:
because of `pos`'s instability, the run-time type of `pos(x)` cannot be predicted, and thus, dispatch cannot be resolved for `f` ahead of time.

As discussed in our paper (ref), type stability is a compiler-dependent
property: formally, a function is type stable for the given input types
if for these types, the compiler is able to infer a _concrete_ return type,
such as `Int64` or `Vector{Float64}`.
Furthermore, stability is not a property of the source language but rather of an
intermediate representation targeted by the type inference algorithm.
To debug type stability,
Julia programmers need to inspect type-inferred IR produced by the compiler.
Although the compiler provides easy access to compiled code
(e.g., `code_warntype(pos, (Float64,))` would show the result of type inference
for a call to `pos` with a float),
type-stable development can be a tedious process that requires
understanding the IR and manually querying the compiler.

## Proposed work

To make the development of type-stable and efficient Julia code more accessible,
we propose to design an IDE tool for type stability analysis.
To guide the design, we will interview Julia programmers with different levels
of expertise in Julia and familiarity with compilers.
Basing off the analysis, we will further expand the tool to provide
suggestions for transforming existing unstable code into type-stable one.

## References

[1]:
    https://docs.julialang.org/en/v1/manual/performance-tips/#Be-aware-of-when-Julia-avoids-specializing
[2]: https://docs.julialang.org/en/v1/manual/performance-tips/#kernel-functions
[3]: https://julialang.org/blog/2020/08/invalidations/