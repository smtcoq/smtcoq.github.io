# C API for SMTCoq - Tutorial
Start with installing the API and reading [the documentation](capi.md)
(no need to read the full documentation of the API at this stage).

> [!TIP]
> When doing the exercises, if you observe that the certified checker
> answers `false`, do not hesitate to use the debugging checker.

## (*) Exercise 1
Modify the introductory example of [the documentation](capi.md) so that,
instead of asserting `a ∧ ¬a`, there are two assertions: `a` and `¬a`.
Modify the certificate accordingly.

<details>
<summary>Tip</summary>
Fewer rule kinds have to be used in the certificate.
</details>
