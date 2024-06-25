<!-- layout: page -->
<!-- title: "C API" -->
<!-- permalink: /capi -->

# C API for SMTCoq
## Presentation
SMTCoq provides a C API for calling the certified checker from C or C++.

This documentation assumes that the reader is knowledgeable in SAT/SMT
solving, and in C or C++.

## Installation
The simplest way to having the API directly installed on the machine is
to use [opam](https://opam.ocaml.org). It has been tested only under
Linux, but opam is also available for MacOS and Windows.

Otherwise, we propose a docker and a VirtualBox images.

### With opam
Create a new switch with OCaml-4.09.1:
```bash
opam switch create smtcoqcapi 4.09.1
eval $(opam env --switch=smtcoqcapi)
```
Download the git repository and switch to the correct branch:
```bash
git clone https://github.com/smtcoq/smtcoq.git
cd smtcoq
git checkout extrapi-coq-8.19
```
Compile and install:
```bash
opam install -y ./coq-smtcoq-extrapi.opam
```
> [!NOTE]
> There is no need to be root, everything is installed in user
> space, in the opam switch.

To compile a C program that uses the API, see
[below](capi.md#compiling-and-running).

### With docker
Under progress

### With VirtualBox
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
  // First, let's get back our assertion, which is labeled `0` as it is
  // the first (and only) one we declared
  CERTIF step1 = cassume("step1", 0);   // Proves the clause `a ∧ ¬a`

  // Then, let's break the ∧ in two, from step1
  CERTIF step2 = cand("step2", step1, 1);   // Proves the clause `a`
  CERTIF step3 = cand("step3", step1, 2);   // Proves the clause `¬a`

  // Finally, we can resolve these two clauses to obtain the empty
  // clause, which witnesses that our assertion was unsatisfiable
  CERTIF clauses[2] = {step2, step3};
  CERTIF step4 = cresolution("step4", 2, clauses);   // Proves the empty clause

  // Let's us now call the certified checker, which should return `true`
  assert(check_proof(step4));

  printf("Example successful\n");

  return 0;
}
```

### Compiling and running
To compile this program, save it in a file called `main.c`, and run the
commands:
```bash
eval $(opam env --switch=smtcoqcapi)
cc -o prog -I `ocamlc -where` -L `ocamlc -where` -L `ocamlc -where`/../zarith -L `ocamlc -where`/../coq-core/perf -L `ocamlc -where`/../coq-core/vm -L `ocamlc -where`/../smtcoq main.c -lsmtcoqapi -lm -lunix -lthreads -lnums -lcamlstr -lzarith -lgmp -lcoqperf_stubs -lcoqrun_stubs
```
It generates a file `prog` which can be run with the command `./prog`.

### What is a certificate?
In this setting, a *certificate* is a proof of the unsatisfiability of
the conjunction of the assertions. Such a proof, starting from the
assertions (using the rule `cassume`), derives new clauses, until it
derives the empty clause.

The invariant is that all the derived clauses are implied by the
assertions. As the empty clause can be derived, and is unsatisfiable,
it entails that the input problem is also unsatisfiable.

We provide a set of rules that is complete for our logic, and which is a
subset of [the Alethe proof
format](https://verit.loria.fr/documentation/alethe-spec.pdf). See below
for details.

### What is a certified checker?
When running the program, the assertion goes through, meaning that
`check_proof(step4)` returned `true`. It means that the certificate
`step4` is indeed a proof of the unsatisfiability of the problem that
was stated with calls to `declare_fun` and `assertf`, **with strong
guaranties**: the checker is extracted from a program that has been
written and certified using the Coq proof assistant. Thus, the degree of
confidence in the fact that the stated problem is unsatisfiable is the
one of Coq, with its extraction mechanism, and a bit of OCaml and C
glue.

What is certified is the correctness of the checker: when it answers
`true`, then the input problem is unsatisfiable. Note that the checker
is not certified to be complete: when it answers `false`, we learn
nothing: it may be that the problem is satisfiable, or that there is a
bug in the certificate.

In addition to correctness, great care has been taken to make the
checker as efficient as possible.

### Debugging
To understand what happens when the certified checker answers `false`,
we also provide a debugging checker. This checker is **not** certified
and is much less efficient, but it outputs the result of each step and
details when something goes wrong.

Let us try on the example: we replace the lines
```c
  assert(check_proof(step4));

  printf("Example successful\n");
```
with
```c
  debug_check_proof(step4);
```
After compiling and running the program, we obtain the following output:
```text
Checking node step1: success, produces the clause {(and a (not a))}
Checking node step2: success, produces the clause {a}
Checking node step3: success, produces the clause {(not a)}
Checking node step4: success, produces the clause {}
Proof checking was successful (but this checker is NOT certified)
```
We can visualize proof checking step by step, referring to them by the
names that were given. The produces clauses are written.

> [!NOTE]
> The steps can be checked in any valid order.

As an example, suppose we make a mistake in the proof, and instead of
resolving the results of `step2` and `step3`, we resolve the results of
`step2` with `step1`. It amounts to replacing the line
```c
  CERTIF clauses[2] = {step2, step3};
```
with
```c
  CERTIF clauses[2] = {step2, step1};
```

Now, the output becomes:
```text
Checking node step1: success, produces the clause {(and a (not a))}
Checking node step2: success, produces the clause {a}
Checking node step4: failure [resolution: could not find resolvant]
Stopping proof checking
```

### Supported logic
This running example belongs to propositional logic, but the checker
currently supports the logic `QF_UFLIA` of SMT-LIB2: propositional logic
with the theories of equality and linear integer arithmetic.

> [!NOTE]
> We plan to add support for bit vectors and arrays in the short term,
> and universal quantifiers and the mid-term.

It means that two sorts are built-in: `sort("Bool")` and `sort("Int")`.
New uninterpreted sorts can be added by:
```c
SORT u = sort("U");
declare_sort(u);
```
Parameterized sorts are not supported yet.

Most expressions of this logic are supported. All the details can be
found in [the full documentation of the API](doc/capi/modules.html).


## Advanced details
### OCaml runtime
As explained, the certified checker is extracted from Coq, in the OCaml
language. Callbacks and a bit of glue allows to use it in C/C++.

This is why the files that uses the checker must load the library
`caml/callback.h`, and the entry point of the program (the `main`
function) must start with `caml_startup(argv);`.

### Imperative vs functional SMT-LIB2 problem
In the example above, the input problem is stated in a way which is
similar to SMT-LIB2: sorts are declared with `declare_sort(•)`, function
and predicate symbols with `declare_fun(•)`, assertions with
`assertf(•)`; and finally, the checker is called with `check_proof(•)`.
For this to work, the whole process **must** start with `start_smt2();`,
which initialize the checker. It implies that, if calling
`start_smt2();` again, all the sorts, function symbols and assertions
must be declared again.

We also propose a functional interface. In the example, it can be used
like this:
```c
int main(int argc, char ** argv)
{
  // The main should always start with this command
  caml_startup(argv);

  // We declare an empty list of uninterpreted sorts
  SORTS s = sorts(0, NULL);

  // `asymb` is a function symbol with name "a", empty domain (`0` and
  // `NULL`), and codomain in the sort `Bool`
  FUNSYM asymb = funsym("a", 0, NULL, sort("Bool"));
  FUNSYMS f = funsyms(1, &asymb);

  // From this function symbol, we can define our expression `a ∧ ¬a`
  // First, `a` is the application of `asym` to an empty list
  EXPR a = efun(asymb, NULL);

  // Then, let's build the expression `a ∧ ¬a`. `¬` is defined by the
  // unary `enot` function, and ∧ by the n-ary `eand` function.
  EXPR args[2] = {a, enot(a)};
  EXPR anota = eand(2, args);

  // Finally, let's assert this formula in our problem
  ASSERTIONS ass = assertions(1, &anota);

  // We obtain the full SMT-LIB2 problem
  SMTLIB2 problem = smtlib2(s, f, ass);

  // We can now build a proof of unsatisfiability, step by step
  // The proof format is a subset of the Alethe format
  //   <https://verit.loria.fr/documentation/alethe-spec.pdf>
  // All the possible steps take as a first argument a name, that can be
  // useful for debugging (more explanations later)
  // First, let's get back our assertion, which is labeled `0` as it is
  // the first (and only) one we declared
  CERTIF step1 = cassume("step1", 0);   // Proves the clause `a ∧ ¬a`

  // Then, let's break the ∧ in two, from step1
  CERTIF step2 = cand("step2", step1, 1);   // Proves the clause `a`
  CERTIF step3 = cand("step3", step1, 2);   // Proves the clause `¬a`

  // Finally, we can resolve these two clauses to obtain the empty
  // clause, which witnesses that our assertion was unsatisfiable
  CERTIF clauses[2] = {step2, step3};
  CERTIF step4 = cresolution("step4", 2, clauses);   // Proves the empty clause

  // Let's us now call the functional certified checker, which should
  // return `true`
  assert(checker(problem, step4));

  printf("Example successful\n");

  return 0;
}
```

### Sorts, function symbols and expressions
Sorts are of type `SORT`. Sorts can be defined using the function
```c
SORT sort(char* s);
```
As explained above, the sorts `"Bool"` and `"Int"` are interpreted
respectively as Booleans and integers (similarly to the SMT-LIB2
standard), and all the other strings define new uninterpreted sorts
which must be declared using the function `declare_sort` in the
imperative way, or `sorts` in the functional variant.

Uninterpreted function and predicate symbols are of type FUNSYM, and can
be defined using the function
```c
FUNSYM funsym(char* name, size_t arity, const SORT* domain, SORT codomain);
```
Again, they must be declared using either `declare_fun` or `funsyms`.

Finally, terms and formulas of first-order logic are of type `EXPR`. We
provide a variety of function to define them using uninterpreted and
interpreted functions and predicates.

### Certificates
Certificates are of type `CERTIF`. They represent proofs of
unsatisfiability of conjunctions of formulas given as assertions. We
provide a subset of the Alethe proof format, as presented
[here](https://verit.loria.fr/documentation/alethe-spec.pdf), with rules
for:
- resolution
- congruence closure
- linear integer arithmetic
- CNF computation
as well as an additional weakening rule.

We refer the reader to the documentation of the API for details.

### The full API
The full documentation of the API is available
[here](doc/capi/modules.html).

### Tutorial
We provide [a tutorial](capi-tutorial.md) with exercises to
progressively take over the API.
