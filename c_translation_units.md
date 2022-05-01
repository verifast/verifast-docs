# C Translation Units

<pre>
<i>translation-unit</i>:
    <i>external-declaration</i>
    <i>translation-unit</i> <i>external-declaration</i>

<i>external-declaration</i>:
    <i>function-definition</i>
    <i>declaration</i>

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
    <i>direct-declarator</i> <b>[</b> <i>type-qualifier-list</i><sub>opt</sub> <i>assignment-expression</i></sub>opt</sub> <b>]</b>
    <i>direct-declarator</i> <b>[</b> <b>static</b> <i>type-qualifier-list</i><sub>opt</sub> <i>assignment-expression</i> <b>]</b>
    <i>direct-declarator</i> <b>[</b> <i>type-qualifier-list</i> <b>static</b> <i>assignment-expression</i> <b>]</b>
    <i>direct-declarator</i> <b>[</b> <i>type-qualifier-list</i><sub>opt</sub> <b>*</b> <b>]</b>
    <i>direct-declarator</i> <b>(</b> <i>parameter-type-list</i> <b>)</b>
    <i>direct-declarator</i> <b>(</b> <i>identifier-list</i><sub>opt</sub> <b>)</b>

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

<i>paraneter-declaration</i>:
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

<i>statement</i>:
    <i>labeled-statement</i>
    <i>compound-statement</i>
    <i>expression-statement</i>
    <i>selection-statement</i>
    <i>iteration-statement</i>
    <i>jump-statement</i>

<i>labeled-statement</i>:
    <i>identifier</i> <b>:</b> <i>statement</i>
    <b>case</b> <i>constant-expression</i> <b>:</b> <i>statement</i>
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

<i>primary-expression</i>:
    <i>identifier</i>
    <i>constant</i>
    <i>string-literal</i>
    <b>(</b> <i>expression</i> <b>)</b>
    <i>generic-selection</i>

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
</pre>
