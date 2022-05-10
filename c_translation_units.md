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

<i>inductive-datatype-declaration</i>:
    <b>inductive</b> <i>identifier</i> <i>generic-parameters</i><sub>opt</sub> <b>=</b> <b>|</b><sub>opt</sub> <i>inductive-datatype-case-list</i> <b>;</b>

<i>generic-parameters</i>:
    <b>&lt;</b> <i>identifier-list</i> <b>></b>

<i>inductive-datatype-case-list</i>:
    <i>inductive-datatype-case-declaration</i>
    <i>inductive-datatype-case-list</i> <b>|</b> <i>inductive-datatype-case-declaration</i>

<i>inductive-datatype-case-declaration</i>:
    <i>identifier</i>
    <i>identifier</i> <b>(</b> <i>parameter-list</i><sub>opt</sub> <b>)</b>

<i>fixpoint-function-definition</i>:
    <b>fixpoint</b> <i>function-definition</i>

<i>predicate-declaration</i>:
    <b>predicate</b> <i>identifier</i> <b>(</b> <i>parameter-list</i> <b>)</b> <b>;</b>
    <i>predicate-keyword</i> <i>identifier</i> <b>(</b> <i>parameter-list</i> <b>)</b> <b>=</b> <i>assertion</i> <b>;</b>

<i>predicate-keyword</i>:
    <b>predicate</b>
    <b>copredicate</b>

<i>predicate-family-declaration</i>:
    <b>predicate_family</b> <i>identifier</i> <b>(</b> <i>parameter-list</i> <b>)</b> <b>(</b> <i>parameter-list</i> <b>)</b> <b>;</b>

<i>predicate-family-instance-definition</i>:
    <b>predicate_family_instance</b> <i>identifier</i> <b>(</b> <i>argument-expression-list</i><sub>opt</sub> <b>)</b> <b>(</b> <i>parameter-list</i> <b>)</b> <b>=</b> <i>assertion</i> <b>;</b>

<i>predicate-constructor-definition</i>:
    <b>predicate_ctor</b> <i>identifier</i> <b>(</b> <i>parameter-list</i> <b>)</b> <b>(</b> <i>parameter-list</i> <b>)</b> <b>=</b> <i>assertion</i> <b>;</b>

<i>lemma-function-declaration</i>:
    <b>lemma</b> <i>declaration</i>
    <b>lemma</b> <i>function-definition</i></span>

<i>function-definition</i>:
    <i>declaration-specifiers</i> <i>declarator</i> <i>declaration-list</i><sub>opt</sub> <i>compound-statement</i>

<i>declaration-list</i>:
    <i>declaration</i>
    <i>declaration-list</i> <i>declaration</i>

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
    <b>predicate</b> <b>(</b> <i>type-list</i> <b>)</b>
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
    <i>direct-declarator</i> <span style="color: purple"><i>generic-parameters</i><sub>opt</sub></span> <b>(</b> <i>parameter-type-list</i> <b>)</b> <i style="color: purple">specification</i><sub>opt</sub>
    <i>direct-declarator</i> <b>(</b> <i>identifier-list</i><sub>opt</sub> <b>)</b>

<span style="color: purple"><i>specification</i>:
    <i>function-type-clause</i><sub>opt</sub> <i>requires-clause</i> <i>ensures-clause</i> <i>terminates-clause</i><sub>opt</sub>

<i>function-type-clause</i>:
    <b>/*@</b> <b>:</b> <i>identifier</i> <i>arguments</i><sub>opt</sub> </b>*@/</b>

<i>arguments</i>:
    <b>(</b> <i>argument-expression-list</i> <b>)</b>

<i>requires-clause</i>:
    <b>/*@</b> <b>requires</b> <i>assertion</i> <b>;</b> <b>@*/</b>
    <b>requires</b> <i>assertion</i> <b>;</b>

<i>ensures-clause</i>:
    <b>/*@</b> <b>ensures</b> <i>assertion</i> <b>;</b> <b>@*/</b>
    <b>ensures</b> <i>assertion</i> <b>;</b>

<i>terminates-clause</i>:
    <b>/*@</b> <b>terminates</b> <b>;</b> <b>@*/</b>
    <b>terminates</b> <b>;</b></span>

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
    <b>while</b> <b>(</b> <i>expression</i> <b>)</b> <i>statement</i>
    <b>do</b> <i>statement</i> <b>while</b> <b>(</b> <i>expression</i> <b>)</b> <b>;</b>
    <b>for</b> <b>(</b> <i>expression</i><sub>opt</sub> <b>;</b> <i>expression</i><sub>opt</sub> <b>;</b> <i>expression</i><sub>opt</sub> <b>)</b> <i>statement</i>
    <b>for</b> <b>(</b> <i>declaration</i> <i>expression</i><sub>opt</sub> <b>;</b> <i>expression</i><sub>opt</sub> <b>)</b> <i>statement</i>

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
    <i>statement</i></span>

</pre>

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
    <i>identifier</i> <i>patterns</i><sub>opt</sub> <i>patterns</i>

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

<i>relational-expression</i>:
    <i>shift-expression</i>
    <i>relational-expression</i> <b>&lt;</b> <i>shift-expression</i>
    <i>relational-expression</i> <b>></b> <i>shift-expression</i>
    <i>relational-expression</i> <b>&lt;=</b> <i>shift-expression</i>
    <i>relational-expression</i> <b>>=</b> <i>shift-expressions</i>

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
