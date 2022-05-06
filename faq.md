# Frequently Asked Questions

## General

### How to express a disjunction "P or Q" in an assertion?

- If P and Q are Boolean expressions, you can simply write `P || Q`. (In this case, the disjunction is made available to the SMT solver. The SMT solver decides when to case split on it. If many disjunctions are available to the SMT solver, this can cause performance degradation.)
- If P is a Boolean expression, you can use a conditional assertion `P ? Q : true`
- If neither P nor Q are Boolean expressions (i.e. they are both *spatial assertions*), then one way to encode a disjunction is as follows: `exists<bool>(?b) &*& b ? P : Q`. The `exists` predicate is defined in the prelude as follows:
  ```
  predicate exists<t>(t x) = true;
  ```
  You need to explicitly close an `exists<bool>` chunk to indicate which disjunct holds. For example, if `P` holds, write:
  ```
  close exists(true);
  ```

*Rationale:* VeriFast does not directly support general disjunction in assertions because we want to avoid backtracking in the symbolic execution engine. Backtracking reduces responsiveness and diagnosability.
