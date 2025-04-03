# SMTCoq

## Presentation
SMTCoq is a [Coq](http://coq.inria.fr) plugin that checks proof witnesses coming from external SAT and SMT solvers. It provides:
* a certified checker for proof witnesses coming from the SAT solver
  [ZChaff](http://www.princeton.edu/~chaff/zchaff.html) and the SMT
  solvers [veriT](http://www.verit-solver.org) and
  [CVC4](http://cvc4.cs.stanford.edu/web). This checker increases the
  confidence in these tools by checking their answers a posteriori and
  allows to import new theroems proved by these solvers in Coq. It is
  accessible in Coq or through a [C API](capi.md);
* decision procedures through new tactics that discharge some Coq goals to ZChaff, veriT, CVC4, and their combination.

## Installation and use
SMTCoq is freely available as an [opam package](https://coq.inria.fr/opam/extra-dev/packages/coq-smtcoq) and on [GitHub](https://github.com/smtcoq/smtcoq). See the [INSTALL.md](https://github.com/smtcoq/smtcoq/blob/master/INSTALL.md) file for instructions on how to install SMTCoq and the supported provers.

See [the examples](https://github.com/smtcoq/smtcoq/blob/master/examples/Example.v) to see how to use SMTCoq. More details are given in the [USE.md](https://github.com/smtcoq/smtcoq/blob/master/USE.md) file.

SMTCoq is distributed under the CeCILL-C license.

## Community
If you have any question or remark, you are invited to join the
[SMTCoq forum](https://framateam.org/smtcoq).

Bugs can be reported on [github](https://github.com/smtcoq/smtcoq/issues).

## Example
Here is a very small example of the possibilities of SMTCoq: automatic proofs in group theory.

```coq
From SMTCoq Require Import SMTCoq.

Section Group.
  Variable G : Type.
  (* We suppose that G has a decidable equality *)
  Variable HG : CompDec G.
  Variable op : G -> G -> G.
  Variable inv : G -> G.
  Variable e : G.

  Local Notation "a ==? b" := (@eqb_of_compdec G HG a b) (at level 60).

  (* We can prove automatically that we have a group if we only have the
     "left" versions of the axioms of a group *)
  Hypothesis associative :
    forall a b c : G, op a (op b c) ==? op (op a b) c.
  Hypothesis inverse :
    forall a : G, op (inv a) a ==? e.
  Hypothesis identity :
    forall a : G, op e a ==? a.
  Add_lemmas associative inverse identity.

  Lemma inverse' :
    forall a : G, op a (inv a) ==? e.
  Proof. smt. Qed.

  Lemma identity' :
    forall a : G, op a e ==? a.
  Proof. smt inverse'. Qed.

  Lemma unique_identity e':
    (forall z, op e' z ==? z) -> e' ==? e.
  Proof. intros pe'; smt pe'. Qed.

  Clear_lemmas.
End Group.
```

## Extension of SMTCoq

A project called [Sniper](https://smtcoq.github.io/sniper) is currently developed 
to extend the number of `Coq` goals that SMTCoq is able to handle.
It is based on small, compositional and certifying pre-processing transformations,
combined together to translate some `Coq` goals into the logic handled by SMTCoq, 
the first-order logic. Have a look at the website !

## Want to participate?

SMTCoq is an ambitious, collaborative project: everyone is welcome!
There are many and varied enhancement to do: join the [SMTCoq
forum](https://framateam.org/smtcoq) to discuss!

## People
### Current team
* [Clark Barrett](http://www.cs.nyu.edu/~barrett) (Stanford University)
* [Valentin Blot](https://valentinblot.org/pro) (Inria Saclay - Île-de-France)
* Boris Djalal (Inria Saclay - Île-de-France)
* [Louise Dubois de Prisque](https://louiseddp.github.io/) (Inria Saclay - Île-de-France)
* [Burak Ekici](http://ekiciburak.github.io/) (The University of Iowa)
* [Benjamin Grégoire](https://www-sop.inria.fr/members/Benjamin.Gregoire/) (Inria Sophia Antipolis)
* [Guy Katz](http://stanford.edu/~guyk) (Stanford University)
* [Chantal Keller](https://www.lri.fr/~keller/index-en.html) (Université Paris-Saclay)
* [Alain Mebsout](https://mebsout.github.io/) (OcamlPro)
* [Andrew Reynolds](http://homepage.divms.uiowa.edu/~ajreynol) (The University of Iowa)
* [Laurent Théry](https://www-sop.inria.fr/marelle/Laurent.Thery/moi.html) (Inria Sophia Antipolis)
* [Cesare Tinelli](http://homepage.cs.uiowa.edu/~tinelli) (The University of Iowa)
* [Pierre Vial](https://pierrevial.github.io) (Inria Saclay - Île-de-France)
* [Arjun Viswanathan](https://cs.union.edu/~viswanaa/) (Union College)

### Past contributors
* Mikaël Armand (Inria Sophia Antipolis)
* Amina Bousalem (Université Paris-Sud)
* Germain Faure (Inria Saclay)
* Quentin Garchery (Université Paris-Sud, Inria Saclay - Île-de-France)
* Tianyi Liang (The University of Iowa)
* [Benjamin Werner](http://www.lix.polytechnique.fr/Labo/Benjamin.Werner) (École polytechnique)


## Publications
### Reference
[A Modular Integration of SAT/SMT Solvers to Coq through Proof Witnesses](http://hal.inria.fr/docs/00/63/91/30/PDF/cpp11.pdf), Armand, Michaël; Faure, Germain; Grégoire, Benjamin; Keller, Chantal; Thery, Laurent; Werner, Benjamin, [CPP - Certified Programs and Proofs - First International Conference - 2011](http://formes.asia/cpp).

### Other publications
1. [Formal Verification of Bit-Vector invertibility Conditions in Coq](https://link.springer.com/chapter/10.1007/978-3-031-43369-6_3), Eikici, Burak; Viswanathan, Arjun; Zohar, Yoni; Tinelli, Cesare; Barrett, Clark, [Frontiers of Combining Systems](https://frocos.cs.uiowa.edu/index.shtml) - 2023.
2. [An Interactive SMT Tactic in Coq using Abductive Reasoning](https://cs.union.edu/~viswanaa/lpar2023.pdf), Barbosa, Haniel; Keller, Chantal; Reynolds, Andrew; Viswanathan, Arjun; Tinelli, Cesare; Barrett, Clark, [EPiC Series in Computing](https://easychair.org/publications/EPiC/Computing) - 2023
3. [SMTCoq: A plug-in for integrating SMT solvers into Coq (Tool Paper)](http://homepage.divms.uiowa.edu/~tinelli/papers/EkiEtAl-CAV-17.pdf), Ekici, Burak; Mebsout, Alain; Tinelli, Cesare; Keller, Chantal; Katz, Guy; Reynolds, Andrew; Barrett, Clark, [CAV - International Conference on Computer Aided Verification](http://cavconference.org/2017) - 2017.
4. [Extending SMTCoq, a Certified Checker for SMT (Extended Abstract)](https://hal.inria.fr/hal-01388984/document), Ekici, Burak; Katz, Guy; Keller, Chantal; Mebsout, Alain; Reynolds, Andrew; Tinelli, Cesare, [HaTT - on Hammers for Type Theories - First International Workshop](https://hatt2016.inria.fr) - 2016.
5. [Verifying SAT and SMT in Coq for a fully automated decision procedure](http://hal.inria.fr/docs/00/61/40/41/PDF/ArmandAl.pdf), Armand, Mickaël; Faure, Germain; Grégoire, Benjamin; Keller, Chantal; Théry, Laurent; Werner, Benjamin, [PSATTT - International Workshop on Proof-Search in Axiomatic Theories and Type Theories](http://www.lix.polytechnique.fr/~lengrand/Events/PSATTT11) - 2011.
6. [SMTCoq : automatisation expressive et extensible dans Coq](https://hal.archives-ouvertes.fr/hal-02369249), Blot, Valentin; Bousalem, Amina; Garchery, Quentin; Keller, Chantal, [JFLA - Journées Francophones des Langages Applicatifs](http://dpt-info.u-strasbg.fr/~magaud/JFLA2019) - 2019.


## Talk
[Overview of SMTCoq (February, 2019)](https://github.com/smtcoq/smtcoq.github.io/blob/master/documents/overview_19-02-11.pdf)
