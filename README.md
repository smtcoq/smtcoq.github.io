## Presentation
SMTCoq is a Coq plugin that checks proof witnesses coming from external SAT and SMT solvers. It provides:
* a certified checker for proof witnesses coming from the SAT solver zChaff and the SMT solver veriT. This checker increases the confidence in these tools by checking their answers a posteriori and allows to import new theroems proved by these solvers in Coq;
* decision procedures through new tactics named verit, that calls veriT on any Coq goal, and zchaff, that calls zChaff on any Coq goal.

## Sources and use
SMTCoq is freely available on [GitHub](https://github.com/smtcoq/smtcoq). A [new branch](https://github.com/ekiciburak/smtcoq) incorporates also the SMT solver CVC4.

SMTCoq is distributed under the CeCILL-C license.

## People
### Current team
* [Burak Ekici](http://ljk.imag.fr/membres/Burak.Ekici) (University of Iowa)
* [Benjamin Grégoire](https://www-sop.inria.fr/members/Benjamin.Gregoire/) (Inria Sophia Antipolis)
* [Chantal Keller](https://www.lri.fr/~keller/index-en.html) (Université Paris-Sud)
* [Alain Mebsout](http://homepage.divms.uiowa.edu/~amebsout/) (University of Iowa)
* [Laurent Théry](https://www-sop.inria.fr/marelle/Laurent.Thery/moi.html) (Inria Sophia Antipolis)
* [Cesare Tinelli](http://homepage.cs.uiowa.edu/~tinelli/) (University of Iowa)

### Past contributors
* Mikaël Armand (Inria Sophia Antipolis)
* Germain Faure (Inria Saclay)
* Tianyi Liang (University of Iowa)
* [Benjamin Werner](http://www.lix.polytechnique.fr/Labo/Benjamin.Werner) (École polytechnique)

## Publications
In progress
