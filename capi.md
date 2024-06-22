<!-- layout: page -->
<!-- title: "C API" -->
<!-- permalink: /capi -->

# C API for SMTCoq
## Presentation
SMTCoq provides a C API for calling the certified checker from C or C++.

This documentation supposes you are knowledgeable in SAT/SMT solving.

## Installation
Under progress

## Questions and bug report
If you have any question or remark, you are invited to join the
[SMTCoq forum](https://framateam.org/smtcoq).

Bugs can be reported on [github](https://github.com/smtcoq/smtcoq/issues).

## Introductory example
### The program
The following C program builds, step by step, a proof of the
unsatisfiability of the formula `a ∧ ¬a`, and checks it with the
certified checker.
```c
#include <stdio.h>
#include <assert.h>
#include <caml/callback.h>
#include "types.h"
#include "checker.h"

int main(int argc, char ** argv)
{
  // The main should always start with this command
  // (more explanations later)
  caml_startup(argv);

  // We state the problem, in a way similar to SMT-LIB2 <https://smt-lib.org>
  // Start
  start_smt2();

  // `asymb` is a function symbol with name "a", empty domain (`0` and
  // `NULL`), and codomain in the sort `Bool`
  FUNSYM asymb = funsym("a", 0, NULL, sort("Bool"));
  declare_fun(asymb);

  // From this function symbol, we can define our expression `a ∧ ¬a`
  // First, `a` is the application of `asym` to an empty list
  EXPR a = efun(asymb, NULL);

  // Then, let's build the expression `a ∧ ¬a`. `¬` is defined by the
  // unary `enot` function, and ∧ by the n-ary `eand` function.
  EXPR args[2] = {a, enot(a)};
  EXPR anota = eand(2, args);

  // Finally, let's assert this formula in our problem
  assertf(anota);

  // We can now build a proof of unsatisfiability, step by step
  // The proof format is a subset of the Alethe format
  //   <https://verit.loria.fr/documentation/alethe-spec.pdf>
  // All the possible steps take as a first argument a name, that can be
  // useful for debugging (more explanations later)
  // First, let's get back our assumption, which is labeled `0` as it is
  // the first (and only) one we declared
  CERTIF step1 = cassume("step1", 0);   // Proves the clause `a ∧ ¬a`

  // Then, let's break the ∧ in two, from step1
  CERTIF step2 = cand("step2", step1, 1);   // Proves the clause `a`
  CERTIF step3 = cand("step3", step1, 2);   // Proves the clause `¬a`

  // Finally, we can resolve these two clauses to obtain the empty
  // clause, which witnesses that our assertion was unsatisfiable
  CERTIF clauses[2] = {step2, step3};   // Proves the empty clause
  CERTIF step4 = cresolution("step4", 2, clauses);   // Proves the empty clause

  // Let's us now call the certified checker, which should return `true`
  assert(check_proof(step4));

  printf("Example successful\n");

  return 0;
}
```

- how to compile and run
- debugging checker

## Advanced information
- main starts with loading the OCaml runtime
- sorts
- funsyms
- expressions
- certificates
- imperative vs functional SMTLIB2 interface
- link to the doc of the full API
