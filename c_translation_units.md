# C Translation Units

This document describes how VeriFast verifies a `.c` file. It does so when the `.c` file is specified as a command-line argument of the `verifast` command-line tool, or when the `.c` file is loaded into the VeriFast IDE and the *Verify* command is issued.

VeriFast first performs preprocessing on the `.c` file. While doing so, it checks that the expansion of a header file does not depend on macros defined before the inclusion of the header file. That is, it checks that the expansion of header files is *context-free*. This is part of ensuring the soundness of the verification of C programs consisting of multiple `.c` files.

The sequence of tokens obtained after preprocessing is called a *translation unit*. VeriFast then parses the translation unit, checking that it conforms with the grammar shown below.

It then checks well-formedness of the various declarations, and then verifies each function that has a specification. It verifies the functions in the order in which they appear in the translation unit. We detail the well-formedness conditions for the various declarations and we describe the verification of functions below.

## Declarations

<pre>
<i>translation-unit</i>:
    <i>external-declaration</i>
    <i>translation-unit</i> <i>external-declaration</i>

<i>external-declaration</i>:
    <i>function-definition</i>
    <i>declaration</i>
    <i style="color: purple">ghost-declaration-block</i>

<span style="color: purple"><i>ghost-declaration-block</i>:
    <b>/*@</b> <i>ghost-declaration-list</i> <b>@*/</b>

<i>ghost-declaration-list</i>:
    <i>ghost-declaration</i>
    <i>ghost-declaration-list</i> <i>ghost-declaration</i>

<i>ghost-declaration</i>:
    <i>inductive-datatype-declaration</i>
    <i>fixpoint-function-definition</i>
    <i>predicate-declaration</i>
    <i>predicate-family-declaration</i>
    <i>predicate-family-instance-definition</i>
    <i>predicate-constructor-definition</i>
    <i>lemma-function-declaration</i>
    <i>box-class-declaration</i></span>
</pre>

### Inductive datatypes

<pre>
<span style="color: purple"><i>inductive-datatype-declaration</i>:
    <b>inductive</b> <i>identifier</i> <i>generic-parameters</i><sub>opt</sub> <b>=</b> <b>|</b><sub>opt</sub> <i>inductive-datatype-case-list</i> <b>;</b>

<i>generic-parameters</i>:
    <b>&lt;</b> <i>identifier-list</i> <b>></b>

<i>inductive-datatype-case-list</i>:
    <i>inductive-datatype-case-declaration</i>
    <i>inductive-datatype-case-list</i> <b>|</b> <i>inductive-datatype-case-declaration</i>

<i>inductive-datatype-case-declaration</i>:
    <i>identifier</i>
    <i>identifier</i> <b>(</b> <i>parameter-list</i><sub>opt</sub> <b>)</b></span>
</pre>

We describe inductive datatypes by example. The declaration

<pre>
<b>inductive</b> list&lt;t> = nil | cons(t, list&lt;t>);
</pre>

introduces the generic type name `list` and generic pure function names `nil` and `cons`. For any type `T`, type `list<T>` denotes the smallest set that satisfies the following constraints:
- `nil<T>` is an element of the set.
- If `v` is a value of type `T` and `l` is an element of the set, then `cons<T>(v, l)` is an element of the set.
- For any value `v` of type `T` and element `l` of the set, `nil<T>` is distinct from `cons<T>(v, l)`.
- For any values `v1` and `v2` of type `T` and elements `l1` and `l2` of the set, `cons<T>(v1, l1)` is equal to `cons<T>(v2, l2)` only if `v1` is equal to `v2` and `l1` is equal to `l2`.

The inductive datatype declarations of a translation unit may refer to each other. That is, types may be defined by mutual induction. VeriFast checks that each type is well-defined (recursion through positive positions only) and inhabited.

### Fixpoint functions

<pre>
<span style="color: purple"><i>fixpoint-function-definition</i>:
    <b>fixpoint</b> <i>function-definition</i></span>
</pre>

A fixpoint function definition defines a pure function in one of three ways:
1. primitive recursive definition
2. simple non-recursive definition
3. definition by well-founded recursion

We describe each case by example. The declaration

<pre>
<b>fixpoint</b> <b>int</b> length&lt;t>(list&lt;t> xs) {
    <b>switch</b> (xs) {
        <b>case</b> nil: <b>return</b> 0;
        <b>case</b> cons(x, xs0): <b>return</b> length(xs0);
    }
}
</pre>

introduces the generic pure function name `length` by primitive recursion on its argument `xs`. The body of a fixpoint function defined by primitive recursion must consist solely of a `switch` statement whose operand is one of the function's parameters. The type of this parameter (called the *inductive parameter*) must be an inductive datatype. The body of each case must be of the form `return E;`. The expression `E` may call fixpoint functions defined earlier, and it may also recursively call the fixpoint function being defined. In the latter case, the argument supplied for the inductive parameter must be one of the variables bound by the parameter list of the current case. (For example, the recursive call `length(xs0)` above is allowed because `xs0` is a variable bound in the header of the case.

The declaration

<pre>
<b>fixpoint</b> <b>int</b> abs(<b>int</b> x) { <b>return</b> x < 0 ? -x : x; }
</pre>

introduces the pure function name `abs`. The body of a fixpoint function defined non-recursively must be of the form `return E;` where `E` calls only fixpoint functions defined earlier.

For fixpoint functions defined by well-founded recursion, see the `wf_funcX.c` examples.

### Predicates

<pre>
<span style="color: purple"><i>predicate-declaration</i>:
    <b>predicate</b> <i>identifier</i> <i>generic-parameters</i><sub>opt</sub> <i>predicate-parameters</i> <b>;</b>
    <i>predicate-keyword</i> <i>identifier</i> <i>generic-parameters</i><sub>opt</sub> <i>predicate-parameters</i> <b>=</b> <i>assertion</i> <b>;</b>

<i>predicate-keyword</i>:
    <b>predicate</b>
    <b>copredicate</b>

<i>predicate-parameters</i>:
    <b>(</b> <i>parameter-list</i> <b>)</b>
    <b>(</b> <i>parameter-list</i> <b>;</b> <i>parameter-list</i> <b>)</b>

<i>predicate-family-declaration</i>:
    <b>predicate_family</b> <i>identifier</i> <b>(</b> <i>parameter-list</i> <b>)</b> <i>predicate-parameters</i> <b>;</b>

<i>predicate-family-instance-definition</i>:
    <b>predicate_family_instance</b> <i>identifier</i> <b>(</b> <i>argument-expression-list</i> <b>)</b> <i>predicate-parameters</i> <b>=</b> <i>assertion</i> <b>;</b>

<i>predicate-constructor-definition</i>:
    <b>predicate_ctor</b> <i>identifier</i> <b>(</b> <i>parameter-list</i> <b>)</b> <b>(</b> <i>parameter-list</i> <b>)</b> <b>=</b> <i>assertion</i> <b>;</b></span>
</pre>

### Lemma functions

<pre>
<span style="color: purple"><i>lemma-function-declaration</i>:
    <i>lemma-keyword</i> <i>declaration</i>
    <i>lemma-keyword</i> <i>function-definition</i>

<i>lemma-keyword</i>:
    <b>lemma</b>
    <b>lemma_auto</b>
    <b>lemma_auto</b> <b>(</b> <i>expression</i> <b>)</b></span>
</pre>

### Function definitions

<pre>
<i>function-definition</i>:
    <i>declaration-specifiers</i> <i>declarator</i> <i>declaration-list</i><sub>opt</sub> <i>compound-statement</i>

<i>declaration-list</i>:
    <i>declaration</i>
    <i>declaration-list</i> <i>declaration</i>
</pre>

### Declarations

<pre>
<i>declaration</i>:
    <i>declaration-specifiers</i> <i>init-declarator-list</i><sub>opt</sub> <b>;</b>
    <i>static_assert-declaration</i>

<i>declaration-specifiers</i>:
    <i>storage-class-specifier</i> <i>declaration-specifiers</i><sub>opt</sub>
    <i>type-specifier</i> <i>declaration-specifiers</i><sub>opt</sub>
    <i>type-qualifier</i> <i>declaration-specifiers</i><sub>opt</sub>
    <i>function-specifier</i> <i>declaration-specifiers</i><sub>opt</sub>
    <i>alignment-specifier</i> <i>declaration-specifiers</i><sub>opt</sub>

<i>init-declarator-list</i>:
    <i>init-declarator</i>
    <i>init-declarator-list</i> <b>,</b> <i>init-declarator</i>

<i>init-declarator</i>:
    <i>declarator</i>
    <i>declarator</i> <b>=</b> <i>initializer</i>

<i>storage-class-specifier</i>:
    <b>typedef</b>
    <b>extern</b>
    <b>static</b>
    <b>_Thread_local</b>
    <b>auto</b>
    <b>register</b>

<i>type-specifier</i>:
    <b>void</b>
    <b>char</b>
    <b>short</b>
    <b>int</b>
    <b>long</b>
    <b>float</b>
    <b>double</b>
    <b>signed</b>
    <b>unsigned</b>
    <b>_Bool</b>
    <b>_Complex</b>
    <i>atomic-type-specifier</i>
    <i>struct-or-union-specifier</i>
    <i>enum-specifier</i>
    <i>typedef-name</i>
    <span style="color: purple"><i>identifier</i> <i>generic-arguments</i><sub>opt</sub>
    <b>real</b>
    <b>fixpoint</b> <b>(</b> <i>type-list</i> <b>)</b>
    <b>predicate</b> <i>predicate-parameters</i>
    <b>any</b>

<i>generic-arguments</i>:
    <b>&lt;</b> <i>type-list</i> <b>></b>

<i>type-list</i>:
    <i>type</i>
    <i>type-list</i> <b>,</b> <i>type</i>

<i>type</i>:
    <i>declaration-specifiers</i></span>

<i>struct-or-union-specifier</i>:
    <i>struct-or-union</i> <i>identifier</i><sub>opt</sub> <b>{</b> <i>struct-declaration-list</i> <b>}</b>
    <i>struct-or-union</i> <i>identifier</i>

<i>struct-or-union</i>:
    <b>struct</b>
    <b>union</b>

<i>struct-declaration-list</i>:
    <i>struct-declaration</i>
    <i>struct-declaration-list</i> <i>struct-declaration</i>

<i>struct-declaration</i>:
    <i>specifier-qualifier-list</i> <i>struct-declarator-list</i><sub>opt</sub> <b>;</b>
    <i>static_assert-declaration</i>

<i>specifier-qualifier-list</i>:
    <i>type-specifier</i> <i>specifier-qualifier-list</i><sub>opt</sub>
    <i>type-qualifier</i> <i>specifier-qualifier-list</i><sub>opt</sub>
    <i>alignment-specifier</i> <i>specifier-qualifier-list</i><sub>opt</sub>

<i>struct-declarator-list</i>:
    <i>struct-declarator</i>
    <i>struct-declarator-list</i> <b>,</b> <i>struct-declarator</i>

<i>struct-declarator</i>:
    <i>declarator</i>
    <i>declarator</i><sub>opt</sub> <b>:</b> <i>constant-expression</i>

<i>enum-specifier</i>:
    <b>enum</b> <i>identifier</i><sub>opt</sub> <b>{</b> <i>enumerator-list</i> <b>}</b>
    <b>enum</b> <i>identifier</i><sub>opt</sub> <b>{</b> <i>enumerator-list</i> <b>,</b> <b>}</b>
    <b>enum</b> <i>identifier</i>

<i>enumerator-list</i>:
    <i>enumerator</i>
    <i>enumerator-list</i> <b>,</b> <i>enumerator</i>

<i>enumerator</i>:
    <i>enumeration-constant</i>
    <i>enumeration-constant</i> <b>=</b> <i>constant-expression</i>

<i>atomic-type-specifier</i>:
    <b>_Atomic</b> <b>(</b> <i>type-name</i> <b>)</b>

<i>type-qualifier</i>:
    <b>const</b>
    <b>restrict</b>
    <b>volatile</b>
    <b>_Atomic</b>

<i>function-specifier</i>:
    <b>inline</b>
    <b>_Noreturn</b>

<i>alignment-specifier</i>:
    <b>_Alignas</b> <b>(</b> <i>type-name</i> <b>)</b>
    <b>_Alignas</b> <b>(</b> <i>constant-expression</i> <b>)</b>

<i>declarator</i>:
    <i>pointer</i><sub>opt</sub> <i>direct-declarator</i>

<i>direct-declarator</i>:
    <i>identifier</i>
    <b>(</b> <i>declarator</i> <b>)</b>
    <i>direct-declarator</i> <b>[</b> <i>type-qualifier-list</i><sub>opt</sub> <i>assignment-expression</i><sub>opt</sub> <b>]</b>
    <i>direct-declarator</i> <b>[</b> <b>static</b> <i>type-qualifier-list</i><sub>opt</sub> <i>assignment-expression</i> <b>]</b>
    <i>direct-declarator</i> <b>[</b> <i>type-qualifier-list</i> <b>static</b> <i>assignment-expression</i> <b>]</b>
    <i>direct-declarator</i> <b>[</b> <i>type-qualifier-list</i><sub>opt</sub> <b>*</b> <b>]</b>
    <i>direct-declarator</i> <span style="color: purple"><i>generic-parameters</i><sub>opt</sub></span> <b>(</b> <i>parameter-type-list</i> <b>)</b> <span style="color: purple"><i>specification-clauses</i><sub>opt</sub></span>
    <i>direct-declarator</i> <b>(</b> <i>identifier-list</i><sub>opt</sub> <b>)</b>

<span style="color: purple"><i>specification-clauses</i>:
    <i>specification-clause</i>
    <i>specification-clauses</i> <i>specification-clause</i>

<i>specification-clause</i>:
    <b>/*@</b> <i>specification-ghost-clause</i> <b>@*/</b>
    <i>specification-ghost-clause</i>

<i>specification-ghost-clause</i>:
    <b>nonghost_callers_only</b>
    <b>:</b> <i>identifier</i> <i>arguments</i><sub>opt</sub>
    <b>requires</b> <i>assertion</i> <b>;</b>
    <b>ensures</b> <i>assertion</i> <b>;</b>
    <b>terminates</b> <b>;</b></span>

<i>arguments</i>:
    <b>(</b> <i>argument-expression-list</i> <b>)</b>

<i>pointer</i>:
    <b>*</b> <i>type-qualifier-list</i><sub>opt</sub>
    <b>*</b> <i>type-qualifier-list</i><sub>opt</sub> <i>pointer</i>

<i>type-qualifier-list</i>:
    <i>type-qualifier</i>
    <i>type-qualifier-list</i> <i>type-qualifier</i>

<i>parameter-type-list</i>:
    <i>parameter-list</i>
    <i>parameter-list</i> <b>,</b> <b>...</b>

<i>parameter-list</i>:
    <i>parameter-declaration</i>
    <i>parameter-list</i> <b>,</b> <i>parameter-declaration</i>

<i>parameter-declaration</i>:
    <i>declaration-specifiers</i> <i>declarator</i>
    <i>declaration-specifiers</i> <i>abstract-declarator</i><sub>opt</sub>

<i>identifier-list</i>:
    <i>identifier</i>
    <i>identifier-list</i> <b>,</b> <i>identifier</i>

<i>type-name</i>:
    <i>specifier-qualifier-list</i> <i>abstract-declarator</i><sub>opt</sub>

<i>abstract-declarator</i>:
    <i>pointer</i>
    <i>pointer</i><sub>opt</sub> <i>direct-abstract-declarator</i>

<i>direct-abstract-declarator</i>:
    <b>(</b> <i>abstract-declarator</i> <b>)</b>
    <i>direct-abstract-declarator</i><sub>opt</sub> <b>[</b> <i>type-qualifier-list</i><sub>opt</sub> <i>assignment-expression</i><sub>opt</sub> <b>]</b>
    <i>direct-abstract-declarator</i><sub>opt</sub> <b>[</b> <b>static</b> <i>type-qualifier-list</i><sub>opt</sub> <i>assignment-expression</i> <b>]</b>
    <i>direct-abstract-declarator</i><sub>opt</sub> <b>[</b> <i>type-qualifier-list</i> <b>static</b> <i>assignment-expression</i> <b>]</b>
    <i>direct-abstract-declarator</i><sub>opt</sub> <b>[</b> <b>*</b> <b>]</b>
    <i>direct-abstract_declarator</i><sub>opt</sub> <b>(</b> <i>parameter-type-list</i><sub>opt</sub> <b>)</b>

<i>typedef-name</i>:
    <i>identifier</i>

<i>initializer</i>:
    <i>assignment-expression</i>
    <b>{</b> <i>initializer-list</i> <b>}</b>
    <b>{</b> <i>initializer-list</i> <b>,</b> <b>}</b>

<i>initializer-list</i>:
    <i>designation</i><sub>opt</sub> <i>initializer</i>
    <i>initializer-list</i> <b>,</b> <i>designation</i><sub>opt</sub> <i>initializer</i>

<i>designation</i>:
    <i>designator-list</i> <b>=</b>

<i>designator-list</i>:
    <i>designator</i>
    <i>designator-list</i> <i>designator</i>

<i>designator</i>:
    <b>[</b> <i>constant-expression</i> <b>]</b>
    <b>.</b> <i>identifier</i>

<i>static_assert-declaration</i>:
    <b>_Static_assert</b> <b>(</b> <i>constant-expression</i> <b>,</b> <i>string-literal</i> <b>)</b> <b>;</b>
</pre>

### Shared boxes

See [*Shared boxes: rely-guarantee reasoning in VeriFast*](http://www.cs.kuleuven.be/publicaties/rapporten/cw/CW662.pdf).

## Statements and blocks

<pre>
<i>statement</i>:
    <i>labeled-statement</i>
    <i>compound-statement</i>
    <i>expression-statement</i>
    <i>selection-statement</i>
    <i>iteration-statement</i>
    <i>jump-statement</i>
    <i style="color: purple">ghost-statement-block</i>
    <i style="color: purple">ghost-statement</i>

<i>labeled-statement</i>:
    <i>identifier</i> <b>:</b> <i>statement</i>
    <b>case</b> <i>constant-expression</i> <b>:</b> <i>statement</i>
    <span style="color: purple"><b>case</b> <i>identifier</i> <i>case-arguments</i><sub>opt</sub> <b>:</b> <i>statement</i></span>
    <b>default</b> <b>:</b> <i>statement</i>

<i>compound-statement</i>:
    <b>{</b> <i>block-item-list</i><sub>opt</sub> <b>}</b>

<i>block-item-list</i>:
    <i>block-item</i>
    <i>block-item-list</i> <i>block-item</i>

<i>block-item</i>:
    <i>declaration</i>
    <i>statement</i>

<i>expression-statement</i>:
    <i>expression</i><sub>opt</sub> <b>;</b>

<i>selection-statement</i>:
    <b>if</b> <b>(</b> <i>expression</i> <b>)</b> <i>statement</i>
    <b>if</b> <b>(</b> <i>expression</i> <b>)</b> <i>statement</i> <b>else</b> <i>statement</i>
    <b>switch</b> <b>(</b> <i>expression</i> <b>)</b> <i>statement</i>

<i>iteration-statement</i>:
    <b>while</b> <b>(</b> <i>expression</i> <b>)</b> <i style="color: purple">loop-annotations</i> <i>statement</i>
    <b>do</b> <i style="color: purple">loop-annotations</i> <i>statement</i> <b>while</b> <b>(</b> <i>expression</i> <b>)</b> <b>;</b>
    <b>for</b> <b>(</b> <i>expression</i><sub>opt</sub> <b>;</b> <i>expression</i><sub>opt</sub> <b>;</b> <i>expression</i><sub>opt</sub> <b>)</b> <i style="color: purple">loop-annotations</i> <i>statement</i>
    <b>for</b> <b>(</b> <i>declaration</i> <i>expression</i><sub>opt</sub> <b>;</b> <i>expression</i><sub>opt</sub> <b>)</b> <i style="color: purple">loop-annotations</i> <i>statement</i>

<span style="color: purple"><i>loop-annotations</i>:
    <i>loop-annotation</i>
    <i>loop-annotations</i> <i>loop-annotation</i>

<i>loop-annotation</i>:
    <b>/*@</b> <i>loop-ghost-annotation</i> <b>@*/</b>
    <i>loop-ghost-annotation</i>

<i>loop-ghost-annotation</i>:
    <b>invariant</b> <i>assertion</i> <b>;</b>
    <b>requires</b> <i>assertion</i> <b>;</b>
    <b>ensures</b> <i>assertion</i> <b>;</b>
    <b>decreases</b> <i>expression</i> <b>;</b></span>

<i>jump-statement</i>:
    <b>goto</b> <i>identifier</i> <b>;</b>
    <b>continue</b> <b>;</b>
    <b>break</b> <b>;</b>
    <b>return</b> <i>expression</i><sub>opt</sub> <b>;</b>

<span style="color: purple"><i>ghost-statement-block</i>:
    <b>/*@</b> <i>ghost-statement</i> <b>@*/</b>

<i>ghost-statement</i>:
    <b>open</b> <i>coefficient</i><sub>opt</sub> <i>predicate-assertion</i> <b>;</b>
    <b>close</b> <i>coefficient</i><sub>opt</sub> <i>predicate-assertion</i> <b>;</b>
    <b>leak</b> <i>assertion</i> <b>;</b>
    <b>invariant</b> <i>assertion</i> <b>;</b>
    <b>produce_lemma_function_pointer_chunk</b> <i>identifier</i> <i>generic-arguments</i><sub>opt</sub> <i>arguments</i> <b>(</b> <i>identifier-list</i> <b>)</b> <i>compound-statement</i> <i>statement</i>
    <b>duplicate_lemma_function_pointer_chunk</b> <b>(</b> <i>expression</i> <b>)</b> <b>;</b>
    <b>produce_function_pointer_chunk</b> <i>identifier</i> <i>generic-arguments</i><sub>opt</sub> <b>(</b> <i>expression</i> <b>)</b> <i>arguments</i> <b>(</b> <i>identifier-list</i> <b>)</b> <i>compound-statement</i>
    <b>merge_fractions</b> <i>assertion</i> <b>;</b>
    <i>shared-boxes-ghost-statements</i>
    <i>statement</i></span>

</pre>

### Shared boxes

See [*Shared boxes: rely-guarantee reasoning in VeriFast*](http://www.cs.kuleuven.be/publicaties/rapporten/cw/CW662.pdf).

## Assertions

<pre>
<span style="color: purple"><i>primary-assertion</i>:
    <i>coefficient</i><sub>opt</sub> <i>points-to-assertion</i>
    <i>coefficient</i><sub>opt</sub> <i>predicate-assertion</i>
    <i>expression</i>
    <b>switch</b> <b>(</b> <i>expression</i> <b>)</b> <b>{</b> <i>switch-assertion-case-list</i> <b>}</b>
    <b>(</b> <i>assertion</i> <b>)</b>
    <b>forall_</b> <b>(</b> <i>type</i> <i>identifier</i> <b>;</b> <i>expression</i> <b>)</b>
    <b>emp</b>

<i>coefficient</i>:
    <b>[</b> <i>pattern</i> <b>]</b>

<i>points-to-assertion</i>:
    <i>conditional-expression</i> <b>|-></b> <i>pattern</i>

<i>predicate-assertion</i>:
    <i>identifier</i> <i>generic-arguments</i><sub>opt</sub> <i>patterns</i><sub>opt</sub> <i>patterns</i>

<i>patterns</i>:
    <b>(</b> <i>pattern-list</i><sub>opt</sub> <b>)</b>

<i>pattern-list</i>:
    <i>pattern</i>
    <i>pattern-list</i> <b>,</b> <i>pattern</i>

<i>pattern</i>:
    <b>_</b>
    <b>?</b> <i>identifier</i>
    <i>identifier</i> <i>patterns</i>
    <i>expression</i>
    <b>^</b> <i>expression</i>

<i>switch-assertion-case-list</i>:
    <i>switch-assertion-case</i>
    <i>switch-assertion-case-list</i> <i>switch-assertion-case</i>

<i>switch-assertion-case</i>:
    <b>case</b> <i>identifier</i> <i>case-arguments</i><sub>opt</sub> <b>:</b> <b>return</b> <i>assertion</i> <b>;</b>

<i>case-arguments</i>:
    <b>(</b> <i>identifier-list</i><sub>opt</sub> <b>)</b>

<i>assertion</i>:
    <i>primary-assertion</i>
    <i>primary-assertion</i> <b>&*&</b> <i>assertion</i>
    <i>expression</i> <b>?</b> <i>assertion</i> <b>:</b> <i>assertion</i>
    <b>ensures</b> <i>assertion</i></span>
</pre>

## Expressions

<pre>
<i>primary-expression</i>:
    <i>identifier</i> <span style="color: purple"><i>generic-arguments</i><sub>opt</sub></span>
    <i>constant</i>
    <i>string-literal</i>
    <b>(</b> <i>expression</i> <b>)</b>
    <i>generic-selection</i>
    <span style="color: purple"><b>switch</b> <b>(</b> <i>expression</i> <b>)</b> <b>{</b> <i>switch-expression-case-list</i> <b>}</b>

<i>switch-expression-case-list</i>:
    <i>switch-expression-case</i>
    <i>switch-expression-case-list</i> <i>switch-expression-case</i>

<i>switch-expression-case</i>:
    <b>case</b> <i>identifier</i> <i>case-arguments</i><sub>opt</sub> <b>:</b> <b>return</b> <i>expression</i> <b>;</b></span>

<i>generic-selection</i>:
    <b>_Generic</b> <b>(</b> <i>assignment-expression</i> <b>,</b> <i>generic-assoc-list</i> <b>)</b>

<i>generic-assoc-list</i>:
    <i>generic-association</i>
    <i>generic-assoc-list</i> <b>,</b> <i>generic-association</i>

<i>generic-association</i>:
    <i>type-name</i> <b>:</b> <i>assignment-expression</i>
    <b>default</b> <b>:</b> <i>assignment-expression</i>

<i>postfix-expression</i>:
    <i>primary-expression</i>
    <i>postfix-expression</i> <b>[</b> <i>expression</i> <b>]</b>
    <i>postfix-expression</i> <b>(</b> <i>argument-expression-list</i><sub>opt</sub> <b>)</b>
    <i>postfix-expression</i> <b>.</b> <i>identifier</i>
    <i>postfix-expression</i> <b>-></b> <i>identifier</i>
    <i>postfix-expression</i> <b>++</b>
    <i>postfix-expression</i> <b>-</b>
    <b>(</b> <i>type-name</i> <b>)</b> <b>{</b> <i>initializer-list</i> <b>}</b>
    <b>(</b> <i>type-name</i> <b>)</b> <b>{</b> <i>initializer-list</i> <b>,</b> <b>}</b>

<i>argument-expression-list</i>:
    <i>assignment-expression</i>
    <i>argument-expression-list</i> <b>,</b> <i>assignment-expression</i>

<i>unary-expression</i>:
    <i>postfix-expression</i>
    <b>++</b> <i>unary-expression</i>
    <b>-</b> <i>unary-expression</i>
    <i>unary-operator</i> <i>cast-expression</i>
    <b>sizeof</b> <i>unary-expression</i>
    <b>sizeof</b> <b>(</b> <i>type-name</i> <b>)</b>
    <b>_Alignof</b> <b>(</b> <i>type-name</i> <b>)</b>

<i>unary-operator</i>: one of
    <b>&amp;</b> <b>*</b> <b>+</b> <b>-</b> <b>~</b> <b>!</b>

<i>cast-expression</i>:
    <i>unary-expression</i>
    <b>(</b> <i>type-name</i> <b>)</b> <i>cast-expression</i>

<i>multiplicative-expression</i>:
    <i>cast-expression</i>
    <i>multiplicative-expression</i> <b>*</b> <i>cast-expression</i>
    <i>multiplicative-expression</i> <b>/</b> <i>cast-expression</i>
    <i>multiplicative-expression</i> <b>%</b> <i>cast-expression</i>

<i>additive-expression</i>:
    <i>multiplicative-expression</i>
    <i>additive-expression</i> <b>+</b> <i>multiplicative-expression</i>
    <i>additive-expression</i> <b>-</b> <i>multiplicative-expression</i>

<i>shift-expression</i>:
    <i>additive-expression</i>
    <i>shift-expression</i> <b>&lt;&lt;</b> <i>additive-expression</i>
    <i>shift-expression</i> <b>>></b> <i>additive-expression</i>

<span style="color: purple"><i>truncating-expression</i>:
    <i>shift-expression</i>
    <b>/*@</b> <b>truncating</b> <b>@*/</b> <i>postfix-expression</i>
    <b>truncating</b> <i>postfix-expression</i></span>

<i>relational-expression</i>:
    <i style="color: purple">truncating-expression</i>
    <i>relational-expression</i> <b>&lt;</b> <i style="color: purple">truncating-expression</i>
    <i>relational-expression</i> <b>></b> <i style="color: purple">truncating-expression</i>
    <i>relational-expression</i> <b>&lt;=</b> <i style="color: purple">truncating-expression</i>
    <i>relational-expression</i> <b>>=</b> <i style="color: purple">truncating-expression</i>

<i>equality-expression</i>:
    <i>relational-expression</i>
    <i>equality-expression</i> <b>==</b> <i>relational-expression</i>
    <i>equality-expression</i> <b>!=</b> <i>relational-expression</i>

<i>AND-expression</i>:
    <i>equality-expression</i>
    <i>AND-expression</i> <b>&amp;</b> <i>equality-expression</i>

<i>exclusive-OR-expression</i>:
    <i>AND-expression</i>
    <i>exclusive-OR-expression</i> <b>^</b> <i>AND-expression</i>

<i>inclusive-OR-expression</i>:
    <i>exclusive-OR-expression</i>
    <i>inclusive-OR-expression</i> <b>|</b> <i>exclusive-OR-expression</i>

<i>logical-AND-expression</i>:
    <i>inclusive-OR-expression</i>
    <i>logical-AND-expression</i> <b>&amp;&amp;</b> <i>inclusive-OR-expression</i>

<i>logical-OR-expression</i>:
    <i>logical-AND-expression</i>
    <i>logical-OR-expression</i> <b>||</b> <i>logical-AND-expression</i>

<i>conditional-expression</i>:
    <i>logical-OR-expression</i>
    <i>logical-OR-expression</i> <b>?</b> <i>expression</i> <b>:</b> <i>conditional-expression</i>

<i>assignment-expression</i>:
    <i>conditional-expression</i>
    <i>unary-expression</i> <i>assignment-operator</i> <i>assignment-expression</i>

<i>assignment-operator</i>: one of
    <b>=</b> <b>*=</b> <b>/=</b> <b>%=</b> <b>+=</b> <b>-=</b> <b>&lt;&lt;=</b> <b>>>=</b> <b>&amp;=</b> <b>^=</b> <b>|=</b>

<i>expression</i>:
    <i>assignment-expression</i>
    <i>expression</i> <b>,</b> <i>assignment-expression</i>

<i>constant-expression</i>:
    <i>conditional-expression</i>
</pre>
