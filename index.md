## Presentation
SMTCoq is a [Coq](http://coq.inria.fr) plugin that checks proof witnesses coming from external SAT and SMT solvers. It provides:
* a certified checker for proof witnesses coming from the SAT solver [ZChaff](http://www.princeton.edu/~chaff/zchaff.html) and the SMT solvers [veriT](http://www.verit-solver.org) and [CVC4](http://cvc4.cs.stanford.edu/web). This checker increases the confidence in these tools by checking their answers a posteriori and allows to import new theroems proved by these solvers in Coq;
* decision procedures through new tactics that discharge some Coq goals to ZChaff, veriT and CVC4.

## Installation and use
SMTCoq is freely available as an [opam package](https://coq.inria.fr/opam/www/) and on [GitHub](https://github.com/smtcoq/smtcoq).

To use the CVC4 SMT solver, you must download the [lfsc branch](https://github.com/smtcoq/smtcoq/tree/lfsc).

New:
* the `verit` tactic now has support for quantifiers: see the end of [this file](https://github.com/smtcoq/smtcoq/blob/master/examples/Example.v) for examples
* tactics `nat_convert`, `pos_convert` and `N_convert` now allow to reason about other types of integers: see also the end of [this file](https://github.com/smtcoq/smtcoq/blob/master/examples/Example.v) for examples

These three features will be merged in a realease to come in December 2018.

SMTCoq is distributed under the CeCILL-C license.

## People
### Current team
* [Clark Barrett](http://www.cs.nyu.edu/~barrett) (Stanford University)
* [Valentin Blot](https://valentinblot.org/pro) (Université Paris-Sud, Université Paris-Diderot)
* Amina Bousalem (Université Paris-Sud)
* [Burak Ekici](http://ekiciburak.github.io/) (The University of Iowa)
* Quentin Garchery (Université Paris-Sud, Inria)
* [Benjamin Grégoire](https://www-sop.inria.fr/members/Benjamin.Gregoire/) (Inria Sophia Antipolis)
* [Guy Katz](http://stanford.edu/~guyk) (Stanford University)
* [Chantal Keller](https://www.lri.fr/~keller/index-en.html) (Université Paris-Sud)
* [Alain Mebsout](https://mebsout.github.io/) (OcamlPro)
* [Andrew Reynolds](http://homepage.divms.uiowa.edu/~ajreynol) (The University of Iowa)
* [Laurent Théry](https://www-sop.inria.fr/marelle/Laurent.Thery/moi.html) (Inria Sophia Antipolis)
* [Cesare Tinelli](http://homepage.cs.uiowa.edu/~tinelli/) (The University of Iowa)

### Past contributors
* Mikaël Armand (Inria Sophia Antipolis)
* Germain Faure (Inria Saclay)
* Tianyi Liang (The University of Iowa)
* [Benjamin Werner](http://www.lix.polytechnique.fr/Labo/Benjamin.Werner) (École polytechnique)

## Publications
### Reference
[A Modular Integration of SAT/SMT Solvers to Coq through Proof Witnesses](http://hal.inria.fr/docs/00/63/91/30/PDF/cpp11.pdf), Armand, Michaël; Faure, Germain; Grégoire, Benjamin; Keller, Chantal; Thery, Laurent; Werner, Benjamin, [CPP - Certified Programs and Proofs - First International Conference - 2011](http://formes.asia/cpp).

### Other publications
1. [SMTCoq: A plug-in for integrating SMT solvers into Coq (Tool Paper)](http://homepage.divms.uiowa.edu/~tinelli/papers/EkiEtAl-CAV-17.pdf), Ekici, Burak; Mebsout, Alain; Tinelli, Cesare; Keller, Chantal; Katz, Guy; Reynolds, Andrew; Barrett, Clark, [CAV - International Conference on Computer Aided Verification](http://cavconference.org/2017) - 2017.
2. [Extending SMTCoq, a Certified Checker for SMT (Extended Abstract)](https://hal.inria.fr/hal-01388984/document), Ekici, Burak; Katz, Guy; Keller, Chantal; Mebsout, Alain; Reynolds, Andrew; Tinelli, Cesare, [HaTT - on Hammers for Type Theories - First International Workshop](https://hatt2016.inria.fr) - 2016.
3. [Verifying SAT and SMT in Coq for a fully automated decision procedure](http://hal.inria.fr/docs/00/61/40/41/PDF/ArmandAl.pdf), Armand, Mickaël; Faure, Germain; Grégoire, Benjamin; Keller, Chantal; Théry, Laurent; Wener, Benjamin, [PSATTT - International Workshop on Proof-Search in Axiomatic Theories and Type Theories](http://www.lix.polytechnique.fr/~lengrand/Events/PSATTT11) - 2011.
