# C API for SMTCoq - Tutorial
Start with installing the API and reading [the introductory example of
the documentation](capi.md#introductory-example) (no need to read the
"Advanced details" section or the full documentation of the API at this
stage).

> [!TIP]
> When doing the exercises, if you observe that the certified checker
> answers `false`, do not hesitate to use the debugging checker.

## Propositional logic
### (*) Exercise 1
Modify the introductory example of [the documentation](capi.md) so that,
instead of asserting `a ∧ ¬a`, there are two assertions: `a` and `¬a`.
Modify the certificate accordingly.

<details>
<summary>Tip</summary>
Fewer rule kinds are needed for the certificate.
</details>

<details>
<summary>Solution</summary>
```c
int main(int argc, char ** argv)
{
  caml_startup(argv);
  start_smt2();
  FUNSYM asymb = funsym("a", 0, NULL, sort("Bool"));
  declare_fun(asymb);
  EXPR a = efun(asymb, NULL);

  /* Two assertions instead of one */
  assertf(a);
  assertf(enot(a));

  /* The two assumptions already prove steps 2 and 3 */
  CERTIF step2 = cassume("step2", 0);   // Proves the clause `a`
  CERTIF step3 = cassume("step3", 1);   // Proves the clause `¬a`
  CERTIF clauses[2] = {step2, step3};
  CERTIF step4 = cresolution("step4", 2, clauses);   // Proves the empty clause
  assert(check_proof(step4));
  return 0;
}
```
</details>

### (*) Exercise 2
Check a proof of the unsatisfiability of the conjunction of the two
assertions `a ∧ ¬b` and `¬a ∧ b`.

<details>
<summary>Solution</summary>
```c
int main(int argc, char ** argv)
{
  caml_startup(argv);
  start_smt2();

  /* The two variables */
  FUNSYM asymb = funsym("a", 0, NULL, sort("Bool"));
  declare_fun(asymb);
  EXPR a = efun(asymb, NULL);
  FUNSYM bsymb = funsym("b", 0, NULL, sort("Bool"));
  declare_fun(bsymb);
  EXPR b = efun(bsymb, NULL);

  /* The two assertions */
  EXPR args1[2] = {a, enot(b)};
  assertf(eand(2, args1));
  EXPR args2[2] = {enot(a), b};
  assertf(eand(2, args2));

  /* Certificate: assertions */
  CERTIF ass0 = cassume("ass0", 0);
  CERTIF ass1 = cassume("ass1", 1);

  /* Certificate: only one side of each conjunction is useful */
  CERTIF and0 = cand("and0", ass0, 1);
  CERTIF and1 = cand("and1", ass1, 1);

  /* Certificate: resolution */
  CERTIF res[2] = {and0, and1};
  CERTIF proof = cresolution("proof", 2, res);

  /* Proof checking */
  assert(check_proof(proof));
  return 0;
}
```
</details>

### (*) Exercise 3
Read the documentation about [disjunctive
expressions](doc/capi/group__expr.html#gab60d6ebf23e56fb0b44f87e8f259fdd6)
and the [or
rule](doc/capi/group__certif.html#gab8056b691f59ebb7bebeff31fb8f267e).

Check a proof of the unsatisfiability of the conjunction of the two
assertions `a ∧ ¬b` and `¬a ∨ b`.

<details>
<summary>Solution</summary>
```c
int main(int argc, char ** argv)
{
  caml_startup(argv);
  start_smt2();

  /* The two variables */
  FUNSYM asymb = funsym("a", 0, NULL, sort("Bool"));
  declare_fun(asymb);
  EXPR a = efun(asymb, NULL);
  FUNSYM bsymb = funsym("b", 0, NULL, sort("Bool"));
  declare_fun(bsymb);
  EXPR b = efun(bsymb, NULL);

  /* The two assertions */
  EXPR args1[2] = {a, enot(b)};
  assertf(eand(2, args1));
  EXPR args2[2] = {enot(a), b};
  assertf(eor(2, args2));

  /* Certificate: assertions */
  CERTIF ass0 = cassume("ass0", 0);
  CERTIF ass1 = cassume("ass1", 1);

  /* Certificate: both sides of the conjunction of the first assertion
     are useful */
  CERTIF and0 = cand("and0", ass0, 1);
  CERTIF and1 = cand("and1", ass0, 2);

  /* Certificate: decompose the disjunction of the second assertion */
  CERTIF or = cor("or", ass1);

  /* Certificate: resolution */
  CERTIF res[3] = {or, and0, and1};
  CERTIF proof = cresolution("proof", 3, res);

  /* Proof checking */
  assert(check_proof(proof));
  return 0;
}
```
</details>

### (**) Exercise 4
Read [the Wikipedia page of the pigeonhole
principle](https://en.wikipedia.org/wiki/Pigeonhole_principle). State
the principle for 1 hole and 2 pigeons, and prove that it is
unsatisfiable.

<details>
<summary>Tip 1</summary>
One can use 2 Boolean variables: `xi` represents the fact that pigeon
`i` is in the hole.
</details>

<details>
<summary>Tip 2</summary>
There are two conditions:
1. every pigeon must be in the hole
2. the hole cannot contain two pigeons
</details>

<details>
<summary>Tip 3</summary>
The two conditions can be translated as:
1. `x1` and `x2` are `true`
2. `¬x1 ∨ ¬x2` is true
</details>

<details>
<summary>Tip 4</summary>
The proof consists in destroying the disjunction in condition 2, then
resolving with the two conditions 1.
</details>

<details>
<summary>Solution</summary>
```c
int main(int argc, char ** argv)
{
  caml_startup(argv);
  start_smt2();

  /* The 2 variables */
  FUNSYM x1symb = funsym("x1", 0, NULL, sort("Bool"));
  FUNSYM x2symb = funsym("x2", 0, NULL, sort("Bool"));
  declare_fun(x1symb);
  declare_fun(x2symb);
  EXPR x1 = efun(x1symb, NULL);
  EXPR x2 = efun(x2symb, NULL);

  /* Every pigeon is in the hole */
  assertf(x1);
  assertf(x2);

  /* The hole cannot contain more than one pigeon */
  EXPR args[2] = {enot(x1), enot(x2)};
  assertf(eor(2, args));

  /* Certif: assertions */
  CERTIF ass0 = cassume("ass0", 0);
  CERTIF ass1 = cassume("ass1", 1);
  CERTIF ass2 = cassume("ass2", 2);

  /* Certif: or rule */
  CERTIF or = cor("or", ass2);

  /* Certif: resolution */
  CERTIF res[3] = {or, ass0, ass1};
  CERTIF proof = cresolution("proof", 3, res);

  /* Check the proof */
  assert(check_proof(proof));
  return 0;
}
```
</details>

### (***) Exercise 5
State the pigeonhole principle for 2 holes and 3 pigeons, and prove that
it is unsatisfiable.

<details>
<summary>Tip</summary>
One can use 6 Boolean variables: `xij` represents the fact that pigeon
`i` is in hole `j`.
</details>

## Equality
The theory of equality adds the `=` symbol and states that it is
reflexive, symmetric, transitive, and congruent with respect to function
and predicate symbols, independently of types.

Read the documentation about [equality in
expressions](doc/capi/group__expr.html#ga5b4c571a6e7d5e2dd5df2f3b460ec264)
and [defining new uninterpreted sorts](doc/capi/group__sort.html).

### (*) Exercise 6
Test the following program:
```c
int main(int argc, char ** argv)
{
  caml_startup(argv);
  start_smt2();

  /* An uninterpreted sort */
  SORT u = sort("U");
  declare_sort(u);

  /* Two variables of this sort */
  FUNSYM asymb = funsym("a", 0, NULL, u);
  declare_fun(asymb);
  EXPR a = efun(asymb, NULL);
  FUNSYM bsymb = funsym("b", 0, NULL, u);
  declare_fun(bsymb);
  EXPR b = efun(bsymb, NULL);

  /* Assert that `a = b` and `¬(b = a)` */
  assertf(eeq(a, b));
  assertf(enot(eeq(b, a)));

  /* Certificate: assertions */
  CERTIF ass0 = cassume("ass0", 0);
  CERTIF ass1 = cassume("ass1", 1);

  /* Certificate: resolution */
  CERTIF res[2] = {ass0, ass1};
  CERTIF proof = cresolution("proof", 2, res);

  /* Proof checking */
  assert(check_proof(proof));
  return 0;
}
```
What can you deduce from the way the SMTCoq checker treats symmetry of equality?

### (**) Exercise 7
Read the documentation about the rules of
[reflexivity](doc/capi/group__certif.html#ga70b095e9d0b3f1a384694a5153229dc8),
[transitivity](doc/capi/group__certif.html#ga5c6d6243a1007746561fd3ae87cdbb63),
[congruence with function
symbols](doc/capi/group__certif.html#ga2c9d482c3108dcba2e531bc5f3176dbe),
and [congruence with predicate
symbols](doc/capi/group__certif.html#gad473e91564e6a49236df6addab232e1a)
(and [its
variant](doc/capi/group__certif.html#ga305676154055af2d9bf830a5b7b543b3)).

Given an uninterpreted sort `U`, a variable `a` of type `U`, and a
function symbol `f : U → U`, prove that the conjunction of `x = f(x)`
and `¬(f(f(x)) = x)` is unsatisfiable.

<details>
<summary>Solution</summary>
```c
int main(int argc, char ** argv)
{
  caml_startup(argv);
  start_smt2();

  /* The uninterpreted sort */
  SORT u = sort("U");
  declare_sort(u);

  /* A variable of this sort */
  FUNSYM asymb = funsym("a", 0, NULL, u);
  declare_fun(asymb);
  EXPR a = efun(asymb, NULL);

  /* A function symbol of type U → U */
  FUNSYM fsymb = funsym("f", 1, &u, u);
  declare_fun(fsymb);
  EXPR fa = efun(fsymb, &a);
  EXPR ffa = efun(fsymb, &fa);

  /* The two assertions */
  assertf(eeq(a, fa));
  assertf(enot(eeq(ffa, a)));

  /* Certificate: assertions */
  CERTIF ass0 = cassume("ass0", 0);
  CERTIF ass1 = cassume("ass1", 1);

  /* Certificate: congruence */
  EXPR clause[2] = {enot(eeq(a, fa)), eeq(fa, ffa)};
  CERTIF congr = ceq_congruent("congr", 2, clause);

  /* Certificate: transitivity */
  EXPR es[3] = {a, fa, ffa};
  CERTIF trans = ceq_transitive("trans", 3, es);

  /* Certificate: resolution */
  CERTIF res[4] = {congr, trans, ass0, ass1};
  CERTIF proof = cresolution("proof", 4, res);

  /* Proof checking */
  assert(check_proof(proof));
  return 0;
}
```
</details>

### Other rules for equality
Read the documentation about the rules of
[reflexivity](doc/capi/group__certif.html#ga70b095e9d0b3f1a384694a5153229dc8)
and congruence with predicate symbols, [version
1](doc/capi/group__certif.html#gad473e91564e6a49236df6addab232e1a) and
[version
2](doc/capi/group__certif.html#ga305676154055af2d9bf830a5b7b543b3).

## Linear integer arithmetic
Finally, the theory of linear integer arithmetic adds the sort `"Int"`,
integers, and operations on them. We refer the reader to [the
documentation of the expressions](doc/capi/group__expr.html) to
visualize them.

SMTCoq's checker integrates a powerful certifying linear arithmetic
solver called Micromega, designed and implemented by Frédéric Besson.
Thus, there is a single rule for this theory:
[lia_generic](doc/capi/group__certif.html#ga438b36ae6d1fa0056aec06c5b4c5d85b).

> [!NOTE]
> The literals in the clause given to `lia_generic` must belong to the
> theory of linear integer arithmetic only.

### (**) Exercise 8
Given a function symbol `f` of type `Int → Int`, and two variable `x`
and `y` of type `Int`, prove that the conjunction of `y = x+1` and
`¬(f(x) = f(y-1))` is unsatisfiable.

<details>
<summary>Tip</summary>
Remember that the sort `"Int`" is interpreted and can be defined using
`sort("Int")`, as presented in [the documentation about
sorts](doc/capi/group__sort.html).
</details>

<details>
<summary>Solution</summary>
```c
int main(int argc, char ** argv)
{
  caml_startup(argv);
  start_smt2();

  /* The two variables */
  SORT s = sort("Int");
  FUNSYM xsymb = funsym("x", 0, NULL, s);
  declare_fun(xsymb);
  EXPR x = efun(xsymb, NULL);
  FUNSYM ysymb = funsym("y", 0, NULL, s);
  declare_fun(ysymb);
  EXPR y = efun(ysymb, NULL);

  /* A function symbol of type Int → Int */
  FUNSYM fsymb = funsym("f", 1, &s, s);
  declare_fun(fsymb);
  EXPR fx = efun(fsymb, &x);
  EXPR yminone = eminus(y, eint(1));
  EXPR fyminone = efun(fsymb, &yminone);

  /* The two assertions */
  assertf(eeq(y, eadd(x, eint(1))));
  assertf(enot(eeq(fx, fyminone)));

  /* Certificate: assertions */
  CERTIF ass0 = cassume("ass0", 0);
  CERTIF ass1 = cassume("ass1", 1);

  /* Certificate: LIA */
  EXPR clauselia[2] = {enot(eeq(y, eadd(x, eint(1)))), eeq(x, eminus(y, eint(1)))};
  CERTIF lia = clia_generic("lia", 2, clauselia);

  /* Certificate: congruence */
  EXPR clause[2] = {enot(eeq(x, eminus(y, eint(1)))), eeq(fx, fyminone)};
  CERTIF congr = ceq_congruent("congr", 2, clause);

  /* Certificate: resolution */
  CERTIF res[4] = {congr, lia, ass0, ass1};
  CERTIF proof = cresolution("proof", 4, res);

  /* Proof checking */
  assert(check_proof(proof));
  return 0;
}
```
</details>
