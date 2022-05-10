# Symbolic execution

This document describes VeriFast's symbolic execution.

## Terms and symbols

As the name suggests, symbolic execution of programs differs from regular execution of programs mainly in that symbolic execution manipulates symbolic representations of values rather than actual concrete values. In VeriFast, a value is represented as a *term* of multi-sorted first-order logic.

Some examples of terms:
- `x:integer`: a *symbol* of *sort* `integer`
- `5`: a *literal term*
- `x:integer + 5`: the *operator* `+` applied to two subterms
- `div:integer(x:integer, 5)`: the *function symbol* `div:integer(integer,integer)` applied to two subterms
- `length:integer(cons:any(3, cons:any(5, nil:any)))`: a nested application of uninterpreted function symbols. The symbols involved are `length:integer(any)`, `cons:any(integer,any)` and `nil:any`.

## Symbolic execution states

A symbolic execution state consists of the following components:
- The set of used symbols.
- The *path condition*. This is a set of formulae of first-order logic (essentially terms of sort `bool`).
- The *symbolic heap*. This is a multiset of *heap chunks*. A heap chunk consists of a *coefficient* (a term of sort `real`), a *predicate term*, a list of *generic arguments* (a list of types), a list of *arguments* (a list of terms), and optionally some *size tracking information*.
- The *symbolic store*. This maps the local variables in scope to their symbolic value.

Note that the name *symbolic heap* is a bit of a misnomer: it also contains chunks for local variables whose address is taken and for global variables. A better name would be *symbolic pointer-addressable memory*, or, more generally, *symbolic stock of aliasable resources*. In the case of a local variable whose address is taken, the symbolic store maps the variable name to a term denoting the variable's address, not its content. Such store bindings are immutable; assigning to such a variable updates the symbolic heap, not the symbolic store.

For each predicate declaration in the translation unit, a unique symbol is allocated that is used as the predicate term for heap chunks obtained by closing that predicate. Furthermore, for each predicate constructor that takes N constructor arguments, a unique N-ary function symbol is allocated that is used to construct predicate terms that represent applications of the predicate constructor.

Adopting the terminology of [Featherweight VeriFast](https://arxiv.org/pdf/1507.07697.pdf), the *meaning* of a symbolic state can be defined by relating a symbolic state to a set of *semiconcrete states*. A semiconcrete state consists of a semiconcrete heap and a semiconcrete store. A semiconcrete heap is a multiset of semiconcrete heap chunks. A semiconcrete heap chunk consists of a coefficient (a real number), a predicate value (uniquely identifying a predicate definition or a predicate constructor applied to a specific list of constructor argument values), a list of generic arguments, and a list of argument values. A semiconcrete store maps the local variables in scope to their value. We say that a symbolic state *represents* a given semiconcrete state if there exists an interpretation of the sorts and the symbols used in the symbolic state that satisfies the path condition and that maps the symbolic heap and store to their semiconcrete counterparts.

For example, if we represent pointers as integers, the symbolic state `({p}, {p != 0}, {| [1]boolean(p, false), [.5]boolean(p + 1, true) |}, { x |-> p })` represents the semiconcrete states `({| [1]boolean(1000, false), [.5]boolean(1001, true) |}, { x |-> 1000 })` and `({| [1]boolean(2000, false), [.5]boolean(2001, true) |}, { x |-> 2000 })` (among other ones) but not `({| [1]boolean(0, false), [.5]boolean(0, true) |}, { x |-> 0 })`.

Given a set of predicate definitions, the meaning of a semiconcrete heap can be further clarified by relating it to a set of *concrete heaps*. A concrete heap is like a semiconcrete heap except that it contains only chunks for predicates that have no definition (i.e. "built-in predicates"). A semiconcrete heap *represents* a given concrete heap if the semiconcrete heap can be obtained from the concrete heap by performing some finite sequence of *close* operations, replacing a sub-multiset of chunks that satisfies the body of some predicate definition applied to some argument list and scaled by some coefficient by a chunk representing that predicate application.

For example, assuming the predicate definition `predicate boolean(bool *p; bool v) = char(p, ?c) &*& v == (c != 0);`, the semiconcrete state `({| [1]boolean(1000, true) |}, { x |-> 1000 })` represents the concrete states `({| [1]char(1000, 1) |}, { x |-> 1000 })` and `({| [1]char(1000, -1) |}, { x |-> 1000 })`, among other ones.

## Satisfiability Modulo Theories

During symbolic execution, VeriFast often needs to decide whether a given symbolic state is *feasible*, i.e. whether it represents any semiconcrete states at all. If not, VeriFast can safely abandon the current symbolic execution path.

To decide this, VeriFast uses an *SMT solver*. This is a type of automated theorem prover that, if given a set of formulae, reports either "Unsatisfiable" or "Failure". Failure means that the SMT solver was unable to establish unsatisfiability. (Note: SMT solvers are sound but incomplete: if an SMT solver reports "Unsatisfiable", there is indeed no interpretation of the symbols that satisfies the given set of formulae. But if it reports "Failure", this does not mean that the set of formulae is satisfiable.)
