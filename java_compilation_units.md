# Java Translation Units

VeriFast supports most of Java 1.0. More specifically,
VeriFast accepts `.java` files that match the <code><i>CompilationUnit</i></code> production of the grammar below. (Based on Chapter 19 of JLS21.)

<pre>
<i>Literal:</i>
  <i>IntegerLiteral</i>
  <i>FloatingPointLiteral</i>
  <i>BooleanLiteral</i>
  <i>CharacterLiteral</i>
  <i>StringLiteral</i>
  <s><i>TextBlock</i></s>
  <i>NullLiteral</i>

<i>Type:</i>
  <i>PrimitiveType</i>
  <i>ReferenceType</i>
  <font color=purple><i>Identifier</i> <i>[TypeArguments]</i>
  real
  fixpoint ( <i>TypeArgumentList</i> )
  predicate <i>PredicateParameters</i>
  any</font>

<i>PrimitiveType:</i>
  <i>NumericType</i>
  boolean

<i>NumericType:</i>
  <i>IntegralType</i>
  <i>FloatingPointType</i>

<i>IntegralType:</i>
  <i>(one of)</i>
  byte short int long char

<i>FloatingPointType:</i>
  <i>(one of)</i>
  float double

<i>ReferenceType:</i>
  <i>ClassOrInterfaceType</i>
  <i>TypeVariable</i>
  <i>ArrayType</i>

<i>ClassOrInterfaceType:</i>
  <i>ClassType</i>
  <i>InterfaceType</i>

<i>ClassType:</i>
  <i>TypeIdentifier</i> <i>[TypeArguments]</i>
  <i>PackageName</i> . <i>TypeIdentifier</i> <i>[TypeArguments]</i>

<i>InterfaceType:</i>
  <i>ClassType</i>

<i>TypeVariable:</i>
  <i>TypeIdentifier</i>

<i>ArrayType:</i>
  <i>PrimitiveType</i> <i>Dims</i>
  <i>ClassOrInterfaceType</i> <i>Dims</i>
  <i>TypeVariable</i> <i>Dims</i>

<i>Dims:</i>
  [ ] <i>{</i>[ ]<i>}</i>

<i>TypeParameter:</i>
  <i>TypeIdentifier</i> <s><i>[TypeBound]</i></s>

<i>TypeArguments:</i>
  &lt; <i>TypeArgumentList</i> &gt;

<i>TypeArgumentList:</i>
  <i>TypeArgument</i> <i>{</i>, <i>TypeArgument</i><i>}</i>

<i>TypeArgument:</i>
  <i>ReferenceType</i>
  <s><i>Wildcard</i></s>

<i>PackageName:</i>
  <i>Identifier</i>
  <i>PackageName</i> . <i>Identifier</i>

<i>TypeName:</i>
  <i>TypeIdentifier</i>
  <i>PackageOrTypeName</i> . <i>TypeIdentifier</i>

<i>ExpressionName:</i>
  <i>Identifier</i>
  <i>AmbiguousName</i> . <i>Identifier</i>

<i>MethodName:</i>
  <i>UnqualifiedMethodIdentifier</i>

<i>PackageOrTypeName:</i>
  <i>Identifier</i>
  <i>PackageOrTypeName</i> . <i>Identifier</i>

<i>AmbiguousName:</i>
  <i>Identifier</i>
  <i>AmbiguousName</i> . <i>Identifier</i>

<i>CompilationUnit:</i>
  <i>OrdinaryCompilationUnit</i>
  <s><i>ModularCompilationUnit</i></s>

<i>OrdinaryCompilationUnit:</i>
  <i>[PackageDeclaration] {ImportDeclaration} {TopLevelDeclaration}</i>

<i>PackageDeclaration:</i>
  package <i>Identifier</i> <i>{</i>. <i>Identifier</i><i>}</i> ;

<i>ImportDeclaration:</i>
  <i>SingleTypeImportDeclaration</i>
  <i>TypeImportOnDemandDeclaration</i>
  <s><i>SingleStaticImportDeclaration</i></s>
  <s><i>StaticImportOnDemandDeclaration</i></s>

<i>SingleTypeImportDeclaration:</i>
  import <i>TypeName</i> ;

<i>TypeImportOnDemandDeclaration:</i>
  import <i>PackageOrTypeName</i> . * ;

<i>TopLevelDeclaration:</i>
  <i>TopLevelClassOrInterfaceDeclaration</i>
  <font color=purple><i>GhostDeclarationBlock</i></font>

<i>TopLevelClassOrInterfaceDeclaration:</i>
  <i>ClassDeclaration</i>
  <i>InterfaceDeclaration</i>
  ;

<i>ClassDeclaration:</i>
  <i>NormalClassDeclaration</i>
  <s><i>EnumDeclaration</i></s>
  <s><i>RecordDeclaration</i></s>

<i>NormalClassDeclaration:</i>
  <i>{ClassModifier}</i> class <i>TypeIdentifier</i> <i>[TypeParameters[</i> <i>[ClassExtends] [ClassImplements] <s>[ClassPermits]</s> ClassBody</i>

<i>ClassModifier:</i>
  <i>(one of)</i>
  <s><i>Annotation</i></s> public protected private
  abstract <s>static</s> final <s>sealed</s> <s>non-sealed</s> <s>strictfp</s>

<i>TypeParameters:</i>
  &lt; <i>TypeParameterList</i> &gt;

<i>TypeParameterList:</i>
  <i>TypeParameter</i> <i>{</i>, <i>TypeParameter</i><i>}</i>

<i>ClassExtends:</i>
  extends <i>ClassType</i>

<i>ClassImplements:</i>
  implements <i>InterfaceTypeList</i>

<i>InterfaceTypeList:</i>
  <i>InterfaceType</i> <i>{</i>, <i>InterfaceType</i><i>}</i>

<i>ClassBody:</i>
  { <i>{ClassBodyDeclaration}</i> }

<i>ClassBodyDeclaration:</i>
  <i>ClassMemberDeclaration</i>
  <s><i>InstanceInitializer</i></s>
  <s><i>StaticInitializer</i></s>
  <i>ConstructorDeclaration</i>

<i>ClassMemberDeclaration:</i>
  <i>FieldDeclaration</i>
  <i>MethodDeclaration</i>
  <s><i>ClassDeclaration</i></s>
  <s><i>InterfaceDeclaration</i></s>
  ;

<i>FieldDeclaration:</i>
  <i>{FieldModifier}</i> <i>Type</i> <i>VariableDeclaratorList</i> ;

<i>FieldModifier:</i>
  <i>(one of)</i>
  <s><i>Annotation</i></s> public protected private
  static final <s>transient</s> <s>volatile</s>

<i>VariableDeclaratorList:</i>
  <i>VariableDeclarator</i> <i>{</i>, <i>VariableDeclarator</i><i>}</i>

<i>VariableDeclarator:</i>
  <i>VariableDeclaratorId</i> <i>[</i>= <i>VariableInitializer</i><i>]</i>

<i>VariableDeclaratorId:</i>
  <i>Identifier</i> <i>[Dims]</i>

<i>VariableInitializer:</i>
  <i>Expression</i>
  <i>ArrayInitializer</i>

<i>MethodDeclaration:</i>
  <i>{MethodModifier}</i> <i>MethodHeader</i> <i>MethodBody</i>

<i>MethodModifier:</i>
  <i>(one of)</i>
  <s><i>Annotation</i></s> public protected private
  abstract static final <s>synchronized</s> <s>native</s> <s>strictfp</s>

<i>MethodHeader:</i>
  <i>Result</i> <i>MethodDeclarator</i> <i>[Throws]</i> <font color=purple><i>{SpecificationClause}</i></font>
  <s><i>TypeParameters</i> <i>{Annotation}</i> <i>Result</i> <i>MethodDeclarator</i> <i>[Throws]</i></s>

<i>Result:</i>
  <i>Type</i>
  void

<i>MethodDeclarator:</i>
  <i>Identifier</i> ( <i>[FormalParameterList]</i> ) <s>[Dims]</s>

<i>FormalParameterList:</i>
  <i>FormalParameter</i> <i>{</i>, <i>FormalParameter</i><i>}</i>

<i>FormalParameter:</i>
  <s><i>{VariableModifier}</i></s> <i>Type</i> <i>VariableDeclaratorId</i>
  <s><i>VariableArityParameter</i></s>

<i>Throws:</i>
  throws <i>ExceptionTypeList</i>

<i>ExceptionTypeList:</i>
  <i>ExceptionType</i> <i>{</i>, <i>ExceptionType</i><i>}</i>

<i>ExceptionType:</i>
  <i>ClassType</i>
  <s><i>TypeVariable</i></s>

<font color=purple><i>SpecificationClause:</i>
  /*@ <i>SpecificationGhostClause</i> @*/
  <i>SpecificationGhostClause</i>

<i>SpecificationGhostClause:</i>
  nonghost_callers_only
  : <i>Identifier</i> <i>[</i>( <i>ArgumentList</i> )<i>]</i>
  requires <i>Assertion</i> ;
  ensures <i>Assertion</i> ;
  terminates ;</font>

<i>MethodBody:</i>
  <i>Block</i>
  ;

<i>ConstructorDeclaration:</i>
  <i>{ConstructorModifier}</i> <i>ConstructorDeclarator</i> <i>[Throws]</i> <i>ConstructorBody</i>

<i>ConstructorModifier:</i>
  <i>(one of)</i>
  <s><i>Annotation</i></s> public protected private

<i>ConstructorDeclarator:</i>
  <s><i>[TypeParameters]</i></s> <i>SimpleTypeName</i> ( <i>[FormalParameterList]</i> )

<i>SimpleTypeName:</i>
  <i>TypeIdentifier</i>

<i>ConstructorBody:</i>
  { <i>[ExplicitConstructorInvocation]</i> <i>[BlockStatements]</i> }

<i>ExplicitConstructorInvocation:</i>
  <s><i>[TypeArguments]</i></s> this ( <i>[ArgumentList]</i> ) ;
  <s><i>[TypeArguments]</i></s> super ( <i>[ArgumentList]</i> ) ;
  <s><i>ExpressionName</i> . <i>[TypeArguments]</i> super ( <i>[ArgumentList]</i> ] ;</s>
  <s><i>Primary</i> . <i>[TypeArguments]</i> super ( <i>[ArgumentList]</i> ) ;</s>

<i>InterfaceDeclaration:</i>
  <i>NormalInterfaceDeclaration</i>
  <s><i>AnnotationInterfaceDeclaration</i></s>

<i>NormalInterfaceDeclaration:</i>
  <i>{InterfaceModifier}</i> interface <i>TypeIdentifier</i> <i>[TypeParameters]</i> <i>[InterfaceExtends]</i> <s><i>[InterfacePermits]</i></s> <i>InterfaceBody</i>

<i>InterfaceModifier:</i>
  <i>(one of)</i>
  <s><i>Annotation</i></s> public protected private
  abstract static <s>sealed</s> <s>non-sealed</s> <s>strictfp</s>

<i>InterfaceExtends:</i>
  extends <i>InterfaceTypeList</i>

<i>InterfaceBody:</i>
  { <i>{InterfaceMemberDeclaration}</i> }

<i>InterfaceMemberDeclaration:</i>
  <i>ConstantDeclaration</i>
  <i>InterfaceMethodDeclaration</i>
  <s><i>ClassDeclaration</i></s>
  <s><i>InterfaceDeclaration</i></s>
  ;

<i>ConstantDeclaration:</i>
  <i>{ConstantModifier}</i> <i>Type</i> <i>VariableDeclaratorList</i> ;

<i>ConstantModifier:</i>
  <i>(one of)</i>
  <s><i>Annotation</i></s> public private
  abstract <s>default</s> static <s>strictfp</s>

<i>InterfaceMethodDeclaration:</i>
  <i>{InterfaceMethodModifier}</i> <i>MethodHeader</i> <i>MethodBody</i>

<i>InterfaceMethodModifier:</i>
  <i>(one of)</i>
  <s><i>Annotation</i></s> public private
  abstract <s>default</s> static <s>strictfp</s>

<i>ArrayInitializer:</i>
  { <i>[VariableInitializerList]</i> <i>[</i>,<i>]</i> }

<i>VariableInitializerList:</i>
  <i>VariableInitializer</i> <i>{</i>, <i>VariableInitializer</i><i>}</i>

<font color=purple><i>GhostDeclarationBlock:</i>
  /*@ <i>{GhostDeclaration}</i> @*/

<i>GhostDeclaration:
  InductiveDatatypeDeclaration
  FixpointFunctionDefinition
  PredicateDeclaration
  PredicateFamilyDeclaration
  FredicateFamilyInstanceDefinition
  PredicateConstructorDefinition
  LemmaFunctionDeclaration
  BoxClassDeclaration</i>

<i>InductiveDatatypeDeclaration:</i>
  inductive <i>Identifier</i> <i>[GenericParameters]</i> = <i>[</i>|<i>]</i> <i>InductiveDatatypeCase</i> <i>{</i>| <i>InductiveDatatypeCase</i><i>}</i> ;

<i>GenericParameters:</i>
  < <i>Identifier</i> <i>{</i>, <i>Identifier</i><i>}</i> >

<i>InductiveDatatypeCase:</i>
  <i>Identifier</i> <i>[</i> ( <i>[FormalParameterList]</i> ) <i>]</i>

<i>FixpointFunctionDefintion:</i>
  fixpoint <i>MethodDeclaration</i>

<i>PredicateDeclaration:</i>
  predicate <i>Identifier</i> <i>[GenericParameters]</i> <i>PredicateParameters</i> ;
  <i>PredicateKeyword</i> <i>Identifier</i> <i>[GenericParameters]</i> <i>PredicateParameters</i> = <i>Assertion</i> ;

<i>PredicateKeyword:</i>
  predicate
  copredicate

<i>PredicateParameters:</i>
  ( <i>[FormalParameterList]</i> )
  ( <i>[FormalParameterList]</i> ; <i>[FormalParameterList]</i> )

<i>PredicateFamilyDeclaration:</i>
  predicate_family <i>Identifier</i> ( <i>[FormalParameterList]</i> ) <i>PredicateParameters</i> ;

<i>PredicateFamilyInstanceDefinition:</i>
  predicate_family_instance <i>Identifier</i> ( <i>[ArgumentList]</i> ) <i>PredicateParameters</i> = <i>Assertion</i> ;

<i>PredicateConstructorDefinition:</i>
  predicate_ctor <i>Identifier</i> ( <i>FormalParameterList</i> ) ( <i>[FormalParameterList]</i> ) = <i>Assertion</i> ;

<i>LemmaFunctionDeclaration:</i>
  <i>LemmaKeyword</i> <i>MethodDeclaration</i>

<i>LemmaKeyword:</i>
  lemma
  lemma_auto <i>[</i>( <i>Expression</i> )<i>]</i></font>

<i>Block:</i>
  { <i>[BlockStatements]</i> }

<i>BlockStatements:</i>
  <i>BlockStatement</i> <i>{BlockStatement}</i>

<i>BlockStatement:</i>
  <s><i>LocalClassOrInterfaceDeclaration</i></s>
  <i>LocalVariableDeclarationStatement</i>
  <i>Statement</i>

<i>LocalVariableDeclarationStatement:</i>
  <i>LocalVariableDeclaration</i> ;

<i>LocalVariableDeclaration:</i>
  <s><i>{VariableModifier}</i></s> <i>LocalVariableType</i> <i>VariableDeclaratorList</i>

<i>LocalVariableType:</i>
  <i>Type</i>
  <s>var</s>

<i>Statement:</i>
  <i>StatementWithoutTrailingSubstatement</i>
  <i>LabeledStatement</i>
  <i>IfThenStatement</i>
  <i>IfThenElseStatement</i>
  <i>WhileStatement</i>
  <i>ForStatement</i>

<i>StatementNoShortIf:</i>
  <i>StatementWithoutTrailingSubstatement</i>
  <i>LabaledStatementNoShortIf</i>
  <i>IfThenElseStatementNoShortIf</i>
  <i>WhileStatementNoShortIf</i>
  <i>ForStatementNoShortIf</i>

<i>StatementWithoutTrailingSubstatement:</i>
  <i>Block</i>
  <i>EmptyStatement</i>
  <i>ExpressionStatement</i>
  <i>AssertStatement</i>
  <i>SwitchStatement</i>
  <i>DoStatement</i>
  <i>BreakStatement</i>
  <i>ContinueStatement</i>
  <i>ReturnStatement</i>
  <s><i>SynchronizedStatement</i></s>
  <i>ThrowStatement</i>
  <i>TryStatement</i>
  <s><i>YieldStatement</i></s>
  <font color=purple><i>GhostStatementBlock</i>
  <i>GhostStatement</i></font>

<i>EmptyStatement:</i>
  ;

<i>LabeledStatement:</i>
  <i>Identifier</i> : <i>Statement</i>

<i>LabeledStatementNoShortIf:</i>
  <i>Identifier</i> : <i>StatementNoShortIf</i>

<i>ExpressionStatement:</i>
  <i>StatementExpression</i> ;

<i>StatementExpression:</i>
  <i>Assignment</i>
  <i>PreIncrementExpression</i>
  <i>PreDecrementExpression</i>
  <i>PostIncrementExpression</i>
  <i>PostDecrementExpression</i>
  <i>MethodInvocation</i>
  <i>ClassInstanceCreationExpression</i>

<i>IfThenStatement:</i>
  if ( <i>Expression</i> ) <i>Statement</i>

<i>IfThenElseStatement:</i>
  if ( <i>Expression</i> ) <i>StatementNoShortIf</i> else <i>Statement</i>

<i>IfThenElseStatementNoShortIf:</i>
  if ( <i>Expression</i> ) <i>StatementNoShortIf</i> else <i>StatementNoShortIf</i>

<i>AssertStatement:</i>
  assert <i>Expression</i> ;
  <s>assert <i>Expression</i> : <i>Expression</i> ;</s>

<i>SwitchStatement:</i>
  switch ( <i>Expression</i> ) <i>SwitchBlock</i>

<i>SwitchBlock:</i>
  <s>{ <i>SwitchRule</i> <i>{SwitchRule}</i> }</s>
  { <i>{SwitchBlockStatementGroup}</i> <i>{SwitchLabel</i> :</i>}</i> }

<i>SwitchBlockStatementGroup:</i>
  <i>SwitchLabel</i> : <i>{SwitchLabel</i> :<i>}</i> <i>BlockStatements</i>

<i>SwitchLabel:</i>
  case <i>CaseConstant</i> <i>{</i>, <i>CaseConstant}</i>
  <s>case null <i>[</i>, default<i>]</i></s>
  <s>case <i>CasePatterm</i> <i>[Guard]</i></s>
  default
  <font color=purple>case <i>Identifier</i> <i>[</i>( <i>[Identifier</i> <i>{</i>, <i>Identifier}]</i> )<i>]</i></font>

<i>CaseConstant:</i>
  <i>ConditionalExpression</i>

<i>WhileStatement:</i>
  while ( <i>Expression</i> ) <font color=purple><i>{LoopAnnotation}</i></font> <i>Statement</i>

<i>WhileStatementNoShortIf:</i>
  while ( <i>Expression</i> ) <font color=purple><i>{LoopAnnotation}</i></font> <i>StatementNoShortIf</i>

<i>DoStatement:</i>
  do <font color=purple><i>{LoopAnnotation}</i></font> <i>Statement</i> while ( <i>Expression</i> ) ;

<i>ForStatement:</i>
  <i>BasicForStatement</i>
  <s><i>EnhancedForStatement</i></s>

<i>ForStatementNoShortIf:</i>
  <i>BasicForStatementNoShortIf</i>
  <s><i>EnhancedForStatementNoShortIf</i></s>

<i>BasicForStatement:</i>
  for ( <i>[ForInit]</i> ; <i>[Expression]</i> ; <i>[ForUpdate]</i> ) <font color=purple><i>{LoopAnnotation}</i></font> <i>Statement</i>

<i>BasicForStatementNoShortIf:</i>
  for ( <i>[ForInit]</i> ; <i>[Expression]</i> ; <i>[ForUpdate]</i> ) <font color=purple><i>{LoopAnnotation}</i></font> <i>StatementNoShortIf</i>

<i>ForInit:</i>
  <i>StatementExpressionList</i>
  <i>LocalVariableDeclaration</i>

<i>ForUpdate:</i>
  <i>StatementExpressionList</i>

<i>StatementExpressionList:</i>
  <i>StatementExpression</i> <i>{</i>, <i>StatementExpression</i><i>}</i>

<font color=purple><i>LoopAnnotation:</i>
  /*@ <i>LoopGhostAnnotation</i> @*/
  <i>LoopGhostAnnotation</i>

<i>LoopGhostAnnotation:</i>
  invariant <i>Assertion</i> ;
  requires <i>Assertion</i> ;
  ensures <i>Assertion</i> ;
  decreases <i>Assertion</i> ;</font>

<i>BreakStatement:</i>
  break <s><i>[Identifier]</i></s> ;

<i>ContinueStatement:</i>
  continue <s><i>[Identifier]</i></s> ;

<i>ReturnStatement:</i>
  return <i>[Expression]</i> ;

<i>ThrowStatement:</i>
  throw <i>Expression</i> ;

<i>TryStatement:</i>
  try <i>Block</i> <i>Catches</i>
  try <i>Block</i> <i>[Catches]</i> <i>Finally</i>
  <s><i>TryWithResourcesStatement</i></s>

<i>Catches:</i>
  <i>CatchClause</i> <i>{CatchClause}</i>

<i>CatchClause:</i>
  catch ( <i>CatchFormalParameter</i> ) <i>Block</i>

<i>CatchFormalParameter:</i>
  <s><i>{VariableModifier}</i></s> <i>CatchType</i> <i>VariableDeclaratorId</i>

<i>CatchType:</i>
  <i>ClassType</i> <s><i>{</i>| <i>ClassType</i><i>}</i></s>

<i>Finally:</i>
  finally <i>Block</i>

<font color=purple><i>GhostStatementBlock:</i>
  /*@ <i>GhostStatement</i> @*/

<i>GhostStatement:</i>
  open <i>[Coefficient]</i> <i>PredicateAssertion</i> ;
  close <i>[Coefficient]</i> <i>PredicateAssertion</i> ;
  leak <i>Assertion</i> ;
  invariant <i>Assertion</i> ;
  produce_lemma_function_pointer_chunk <i>Identifier</i> <i>[TypeArguments]</i> ( <i>[ArgumentList]</i> ) <i>Identifiers</i> <i>Block</i> <i>Statement</i>
  duplicate_lemma_function_pointer_chunk ( <i>Expression</i> ) ;
  produce_function_pointer_chunk <i>Identifier</i> <i>[TypeArguments]</i> ( <i>Expression</i> ) ( <i>[ArgumentList]</i> ) <i>Identifiers</i> <i>Block</i>
  merge_fractions <i>Assertion</i> ;
  <i>SharedBoxesGhostStatement</i>
  <i>Statement</i>

<i>PrimaryAssertion:</i>
  <i>[Coefficient]</i> <i>PointsToAssertion</i>
  <i>[Coefficient]</i> <i>PredicateAssertion</i>
  <i>Expression</i>
  switch ( <i>Expression</i> ) { <i>{SwitchAssertionCase}</i> }
  ( <i>Assertion</i> )
  forall_ ( <i>Type</i> <i>Identifier</i> ; <i>Expression</i> )
  emp

<i>Coefficient:</i>
  [ <i>Pattern</i> ]

<i>PointsToAssertion:</i>
  <i>ConditionalExpression</i> |-> <i>Pattern</i>

<i>PredicateAssertion:</i>
  <i>Identifier</i> <i>[TypeArguments]</i> <i>[Patterns]</i> <i>Patterns</i>

<i>Patterns:</i>
  ( <i>[Pattern {</i>, <i>Pattern}]</i> )

<i>Pattern:</i>
  _
  ? <i>Identifier</i>
  <i>Identifier</i> <i>Patterns</i>
  <i>Expression</i>
  ^ <i>Expression</i>

<i>SwitchAssertionCase:</i>
  case <i>Identifier</i> <i>[Identifiers]</i> : return <i>Assertion</i> ;

<i>Assertion:</i>
  <i>PrimaryAssertion</i>
  <i>PrimaryAssertion</i> &amp;*&amp; <i>Assertion</i>
  <i>Expression</i> ? <i>Assertion</i> : <i>Assertion</i>
  ensures <i>Assertion</i></font>

<i>Primary:</i>
  <i>PrimaryNoNewArray</i>
  <i>ArrayCreationExpression</i>

<i>PrimaryNoNewArray:</i>
  <i>Literal</i>
  <i>ClassLiteral</i>
  this
  <s><i>TypeName</i> . this</s>
  ( <i>Expression</i> )
  <i>ClassInstanceCreationExpression</i>
  <i>FieldAccess</i>
  <i>ArrayAccess</i>
  <i>MethodInvocation</i>
  <s><i>MethodReference</i></s>
  <font color=purple>switch ( <i>Expression</i> ) { <i>{SwitchExpressionCase}</i> }

<i>SwitchExpressionCase:</i>
  case <i>Identifier</i> <i>[Identifiers]</i> : return <i>Expression</i> ;

<i>Identifiers:</i>
  ( <i>[Identifier {</i>, <i>Identifier}]</i> )</font>

<i>ClassLiteral:</i>
  <i>TypeName</i> <s><i>{</i>[ ]<i>}</i></s> . class
  <s><i>NumericType</i> <i>{</i>[ ]<i>}</i> . class</s>
  <s>boolean <i>{</i>[ ]<i>}</i> . class</s>
  <s>void . class</s>

<i>ClassInstanceCreationExpression:</i>
  <i>UnqualifiedClassInstanceCreationExpression</i>
  <s><i>ExpressionName</i> . <i>UnqualifiedClassInstanceCreationExpression</i></s>
  <s><i>Primary</i> . <i>UnqualifiedClassInstanceCreationExpression</i></s>

<i>UnqualifiedClassInstanceCreationExpression</i>:
  new <s><i>[TypeArguments]</i></s> <i>ClassOrInterfaceTypeToInstantiate</i> ( <i>[ArgumentList]</i> ) <s><i>[ClassBody]</i></s>

<i>ClassOrInterfaceTypeToInstantiate:</i>
  <s><i>{Annotation}</i></s> <i>Identifier</i> <i>{</i>. <s><i>{Annotation}</i></s> <i>Identifier</i><i>}</i> <s><i>[TypeArgumentsOrDiamond]</i></s>

<i>ArrayCreationExpression:</i>
  <i>ArrayCreationExpressionWithoutInitializer</i>
  <i>ArrayCreationExpressionWithInitializer</i>

<i>ArrayCreationExpressionWithoutInitializer:</i>
  new <i>PrimitiveType</i> <i>DimExprs</i> <i>[Dims]</i>
  new <i>ClassOrInterfaceType</i> <i>DimExprs</i> <i>[Dims]</i>

<i>ArrayCreationExpressionWithInitializer:</i>
  new <i>PrimitiveType</i> <i>Dims</i> <i>ArrayInitializer</i>
  new <i>ClassOrInterfaceType</i> <i>Dims</i> <i>ArrayInitializer</i>

<i>DimExprs:</i>
  <i>DimExpr</i> <i>{DimExpr}</i>

<i>DimExpr:</i>
  <s><i>{Annotation}</i></s> [ <i>Expression</i> ]

<i>ArrayAccess:</i>
  <i>ExpressionName</i> [ <i>Expression</i> ]
  <i>PrimaryNoNewArray</i> [ <i>Expression</i> ]
  <i>ArrayCreationExpressionWithInitializer</i> [ <i>Expression</i> ]

<i>FieldAccess:</i>
  <i>Primary</i> . <i>Identifier</i>
  <s>super . <i>Identifier</i></s>
  <s><i>TypeName</i> . super . <i>Identifier</i></s>

<i>MethodInvocation:</i>
  <i>MethodName</i> ( <i>[ArgumentList]</i> )
  <i>Typename</i> . <s><i>[TypeArguments]</i></s> <i>Identifier</i> ( <i>[ArgumentList]</i> )
  <i>ExpressionName</i> . <s><i>[TypeArguments]</i></s> <i>Identifier</i> ( <i>[ArgumentList]</i> )
  <i>Primary</i> . <s><i>[TypeArguments]</i></s> <i>Identifier</i> ( <i>[ArgumentList]</i> )
  super . <s><i>[TypeArguments]</i></s> <i>Identifier</i> ( <i>[ArgumentList]</i> )
  <s><i>TypeName</i> . super . <i>[TypeArguments]</i> <i>Identifier</i> ( <i>[ArgumentList]</i> )</s>

<i>ArgumentList:</i>
  <i>Expression</i> <i>{</i>, <i>Expression}</i>

<i>Expression:</i>
  <s><i>LambdaExpression</i></s>
  <i>AssignmentExpression</i>

<i>AssignmentExpression:</i>
  <i>ConditionalExpression</i>
  <i>Assignment</i>

<i>Assignment:</i>
  <i>LeftHandSide</i> <i>AssignmentOperator</i> <i>Expression</i>

<i>LeftHandSide:</i>
  <i>ExpressionName</i>
  <i>FieldAccess</i>
  <i>ArrayAccess</i>

<i>AssignmentOperator:</i>
  <i>(one of)</i>
  = *= /= %= += -= &lt;&lt;= &gt;&gt;= &gt;&gt;&gt;= &amp;= ^= |= 

<i>ConditionalExpression:</i>
  <i>ConditionalOrExpression</i>
  <i>ConditionalOrExpression</i> ? <i>Expression</i> : <i>ConditionalExpression</i>
  <s><i>ConditionalOrExpression</i> ? <i>Expression</i> : <i>LambdaExpression</i></s>

<i>ConditionalOrExpression:</i>
  <i>ConditionalAndExpression</i>
  <i>ConditionalOrExpression</i> || <i>ConditionalAndExpression</i>

<i>ConditionalAndExpression:</i>
  <i>InclusiveOrExpression</i>
  <i>ConditionalAndExpression</i> &amp;&ampl; <i>InclusiveOrExpression</i>

<i>InclusiveOrExpression:</i>
  <i>ExclusiveOrExpression</i>
  <i>InclusiveOrExpression</i> | <i>ExclusiveOrExpression</i>

<i>ExclusiveOrExpression:</i>
  <i>AndExpression</i>
  <i>ExclusiveOrExpression</i> ^ <i>AndExpression</i>

<i>AndExpression:</i>
  <i>EqualityExpression</i>
  <i>AndExpression</i> &amp; <i>EqualityExpression</i>

<i>EqualityExpression:</i>
  <i>RelationalExpression</i>
  <i>EqualityExpression</i> == <i>RelationalExpression</i>
  <i>EqualityExpression</i> != <i>RelationalExpression</i>

<i>RelationalExpression:</i>
  <font color=purple><i>TruncatingExpression</i></font>
  <i>RelationalExpression</i> &lt; <font color=purple><i>TruncatingExpression</i></font>
  <i>RelationalExpression</i> &gt; <font color=purple><i>TruncatingExpression</i></font>
  <i>RelationalExpression</i> &lt;= <font color=purple><i>TruncatingExpression</i></font>
  <i>RelationalExpression</i> &gt;= <font color=purple><i>TruncatingExpression</i></font>
  <i>InstanceofExpression</i>

<i>InstanceofExpression:</i>
  <i>RelationalExpression</i> instancof <i>ReferenceType</i>
  <s><i>RelationalExpression</i> instanceof <i>Pattern</i></s>

<font color=purple><i>TruncatingExpression:</i>
  <i>ShiftExpression</i>
  /*@ truncating @*/ <i>PostfixExpression</i>
  truncating <i>PostfixExpression</i></font>

<i>ShiftExpression:</i>
  <i>AdditiveExpression</i>
  <i>ShiftExpression</i> &lt;&lt; <i>AdditiveExpression</i>
  <i>ShiftExpression</i> &gt;&gt; <i>AdditiveExpression</i>
  <i>ShiftExpression</i> &gt;&gt;&gt; <i>AdditiveExpression</i>

<i>AdditiveExpression:</i>
  <i>MultiplicativeExpression</i>
  <i>AdditiveExpression</i> + <i>MultiplicativeExpression</i>
  <i>AdditiveExpression</i> - <i>MultiplicativeExpression</i>

<i>MultiplicativeExpression:</i>
  <i>UnaryExpression</i>
  <i>MultiplicativeExpression</i> * <i>UnaryExpression</i>
  <i>MultiplicativeExpression</i> / <i>UnaryExpression</i>
  <i>MultiplicativeExpression</i> % <i>UnaryExpression</i>

<i>UnaryExpression:</i>
  <i>PreIncrementExpression</i>
  <i>PreDecrementExpression</i>
  + <i>UnaryExpression</i>
  - <i>UnaryExpression</i>
  <i>UnaryExpressionNotPlusMinus</i>

<i>PreIncrementExpression:</i>
  ++ <i>UnaryExpression</i>

<i>PreDecrementExpression:</i>
  -- <i>UnaryExpression</i>

<i>UnaryExpressionNotPlusMinus:</i>
  <i>PostfixExpression</i>
  ~ <i>UnaryExpression</i>
  ! <i>UnaryExpression</i>
  <i>CastExpression</i>
  <s><i>SwitchExpression</i></s>

<i>PostfixExpression:</i>
  <i>Primary</i>
  <i>ExpressionName</i> <font color=purple><i>[TypeArguments]</i></font>
  <i>PostIncrementExpression</i>
  <i>PostDecrementExpression</i>

<i>PostIncrementExpression:</i>
  <i>PostfixExpression</i> ++

<i>PostDecrementExpression:</i>
  <i>PostfixExpression</i> --

<i>CastExpression:</i>
  ( <i>PrimitiveType</i> ) <i>UnaryExpression</i>
  ( <i>ReferenceType</i> <s><i>{AdditionalBound}</i></s> ) <i>UnaryExpressionNotPlusMinus</i>
  <s>( <i>ReferenceType</i> <s><i>{AdditionalBound}</i></s> ) <i>LambdaExpression</i></s>
</pre>
