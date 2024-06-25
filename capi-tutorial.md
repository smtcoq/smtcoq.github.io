# C API for SMTCoq - Tutorial
Start with installing the API and reading [the introductory example of
the documentation](capi.md#introductory-example) (no need to read the
"Advanced details" section or the full documentation of the API at this
stage).

> [!TIP]
> When doing the exercises, if you observe that the certified checker
> answers `false`, do not hesitate to use the debugging checker.

## (*) Exercise 1
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

## (*) Exercise 2
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

## (*) Exercise 3
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

## (**) Exercise 4
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

## (***) Exercise 5
State the pigeonhole principle for 2 holes and 3 pigeons, and prove that
it is unsatisfiable.

<details>
<summary>Tip</summary>
One can use 6 Boolean variables: `xij` represents the fact that pigeon
`i` is in hole `j`.
</details>
