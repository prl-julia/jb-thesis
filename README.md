# Julia Belyakova's PhD Thesis

- [![YourActionName Actions Status](https://github.com/prl-julia/jb-thesis/workflows/CI/badge.svg)](https://github.com/prl-julia/jb-thesis/actions)
- [Thesis PDF (Nov 2023)](https://github.com/prl-julia/jb-thesis/blob/build/main.pdf)
- [Thesis proposal PDF (Apr 2023)](proposal/main.pdf)

## Notes and updates

### Semantic incompleteness without negation and intersection

> Aug 2023

As discussed in the thesis (page 70, Chapter 5 Section 2),
one of the examples of unsupported type annotations

```julia
AbstractArray{Union{Missing, T}} where T<:Number
```

has a semantically equivalent one that is representable under the restriction:

```julia
AbstractArray{S} where Missing<:S<:Union{Missing, Number}
```

According to Julia 1.8.5, the former type is a subtype of the latter
but not vice versa.
Interestingly, to derive the second subtyping syntactically,
the type language would need to support **intersection and negation types**.

Consider subtyping

```julia
Missing<:S<:Union{Missing, Number} | T<:Number |- AbstractArray{S} <: AbstractArray{Union{Missing, T}}
```

It requires, in particular, that

```julia
Missing<:S<:Union{Missing, Number} | T<:Number |- S <: Union{Missing, T}
```

constrains unification variable `T` as `S & ~Missing <= T` rather than
`Number <= T`, because `S & Missing` part of subtyping is covered by
`Missing` in the right-hand side type.

### Incompleteness of constrained subtyping with respect to distributivity

> Sep 2023

Constrained subtyping was intended to be complete with respect to
unification-free subtyping, meaning that whenever there is a substitution
for unification variables such that unification-free subtyping holds
for the result of the substitution,
constrained subtyping would produce a constraint set compatible with the
substitution.

However, in the presence of distributivity, this is not actually true
due to imprecise intersections, as discovered by [Ross Tate](https://rosstate.org/).
For example, when the upper bound of `X` is `Union{Int,Bool}`,

```julia
meet(
    Tuple{X, Any}, 
    Union{Tuple{Int, String}, Tuple{Bool, String}}
)
```

is `Tuple{X, String}`,
but the intersection function in the thesis computes `Tuple{Union{}, String}`.

**Note.** Without the distributivity of tuples over unions and existentials,
constrained subtyping is complete.

> Oct 2023

Unfortunately, we discovered that **Julia does not have meets**.
To represent meets, the type language needs to be extended with
intersection types (or, at least, intersections where one component
is a type variable).

It is unclear whether such an extension is warranted.
The impact of incomplete handling of distributivity is limited,
judging by our [OOPSLA 2018](https://julbinb.github.io/papers.html#oopsla2018)
work on reconstructing Julia subtyping.
There, the rule `Tuple_Unlift_Union` (which is analogous to `SC-UVar-UnionRight`
of constrained subtyping, which is responsible for incompleteness)
is virtually unused (27 usages out of 6 million subtype queries tested).
