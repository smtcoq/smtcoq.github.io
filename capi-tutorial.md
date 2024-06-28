# C API for SMTCoq - Tutorial
Start with installing the API and reading the introductory example of
[the documentation](capi.md) (no need to read the "Advanced details"
section or the full documentation of the API at this stage).

> Tip:
> When doing the exercises, if you observe that the certified checker
> answers `false`, do not hesitate to use the debugging checker.

## Propositional logic
### (*) Exercise 1
Modify the introductory example of [the documentation](capi.md) so that,
instead of asserting `a ∧ ¬a`, there are two assertions: `a` and `¬a`.
Modify the certificate accordingly.

<details>
<summary>Tip</summary>
<p>Fewer rule kinds are needed for the certificate.</p>
</details>

<details>
<summary>Solution</summary>
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">**</span> <span class="n">argv</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">caml_startup</span><span class="p">(</span><span class="n">argv</span><span class="p">);</span>
  <span class="n">start_smt2</span><span class="p">();</span>
  <span class="n">FUNSYM</span> <span class="n">asymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"a"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">asymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">a</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">asymb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>

  <span class="cm">/* Two assertions instead of one */</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">a</span><span class="p">);</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">enot</span><span class="p">(</span><span class="n">a</span><span class="p">));</span>

  <span class="cm">/* The two assumptions already prove steps 2 and 3 */</span>
  <span class="n">CERTIF</span> <span class="n">step2</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"step2"</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>   <span class="c1">// Proves the clause `a`</span>
  <span class="n">CERTIF</span> <span class="n">step3</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"step3"</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>   <span class="c1">// Proves the clause `¬a`</span>
  <span class="n">CERTIF</span> <span class="n">clauses</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">step2</span><span class="p">,</span> <span class="n">step3</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">step4</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"step4"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">clauses</span><span class="p">);</span>   <span class="c1">// Proves the empty clause</span>
  <span class="n">assert</span><span class="p">(</span><span class="n">check_proof</span><span class="p">(</span><span class="n">step4</span><span class="p">));</span>
  <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
</details>

### (*) Exercise 2
Check a proof of the unsatisfiability of the conjunction of the two
assertions `a ∧ ¬b` and `¬a ∧ b`.

<details>
<summary>Solution</summary>
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">**</span> <span class="n">argv</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">caml_startup</span><span class="p">(</span><span class="n">argv</span><span class="p">);</span>
  <span class="n">start_smt2</span><span class="p">();</span>

  <span class="cm">/* The two variables */</span>
  <span class="n">FUNSYM</span> <span class="n">asymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"a"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">asymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">a</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">asymb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">FUNSYM</span> <span class="n">bsymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"b"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">bsymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">b</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">bsymb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>

  <span class="cm">/* The two assertions */</span>
  <span class="n">EXPR</span> <span class="n">args1</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">a</span><span class="p">,</span> <span class="n">enot</span><span class="p">(</span><span class="n">b</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eand</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">args1</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">args2</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">a</span><span class="p">),</span> <span class="n">b</span><span class="p">};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eand</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">args2</span><span class="p">));</span>

  <span class="cm">/* Certificate: assertions */</span>
  <span class="n">CERTIF</span> <span class="n">ass0</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass0"</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass1</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass1"</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>

  <span class="cm">/* Certificate: only one side of each conjunction is useful */</span>
  <span class="n">CERTIF</span> <span class="n">and0</span> <span class="o">=</span> <span class="n">cand</span><span class="p">(</span><span class="s">"and0"</span><span class="p">,</span> <span class="n">ass0</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">and1</span> <span class="o">=</span> <span class="n">cand</span><span class="p">(</span><span class="s">"and1"</span><span class="p">,</span> <span class="n">ass1</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>

  <span class="cm">/* Certificate: resolution */</span>
  <span class="n">CERTIF</span> <span class="n">res</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">and0</span><span class="p">,</span> <span class="n">and1</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">proof</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"proof"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">res</span><span class="p">);</span>

  <span class="cm">/* Proof checking */</span>
  <span class="n">assert</span><span class="p">(</span><span class="n">check_proof</span><span class="p">(</span><span class="n">proof</span><span class="p">));</span>
  <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
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
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">**</span> <span class="n">argv</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">caml_startup</span><span class="p">(</span><span class="n">argv</span><span class="p">);</span>
  <span class="n">start_smt2</span><span class="p">();</span>

  <span class="cm">/* The two variables */</span>
  <span class="n">FUNSYM</span> <span class="n">asymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"a"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">asymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">a</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">asymb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">FUNSYM</span> <span class="n">bsymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"b"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">bsymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">b</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">bsymb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>

  <span class="cm">/* The two assertions */</span>
  <span class="n">EXPR</span> <span class="n">args1</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">a</span><span class="p">,</span> <span class="n">enot</span><span class="p">(</span><span class="n">b</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eand</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">args1</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">args2</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">a</span><span class="p">),</span> <span class="n">b</span><span class="p">};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">args2</span><span class="p">));</span>

  <span class="cm">/* Certificate: assertions */</span>
  <span class="n">CERTIF</span> <span class="n">ass0</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass0"</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass1</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass1"</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>

  <span class="cm">/* Certificate: both sides of the conjunction of the first assertion
     are useful */</span>
  <span class="n">CERTIF</span> <span class="n">and0</span> <span class="o">=</span> <span class="n">cand</span><span class="p">(</span><span class="s">"and0"</span><span class="p">,</span> <span class="n">ass0</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">and1</span> <span class="o">=</span> <span class="n">cand</span><span class="p">(</span><span class="s">"and1"</span><span class="p">,</span> <span class="n">ass0</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>

  <span class="cm">/* Certificate: decompose the disjunction of the second assertion */</span>
  <span class="n">CERTIF</span> <span class="n">or</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or"</span><span class="p">,</span> <span class="n">ass1</span><span class="p">);</span>

  <span class="cm">/* Certificate: resolution */</span>
  <span class="n">CERTIF</span> <span class="n">res</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">or</span><span class="p">,</span> <span class="n">and0</span><span class="p">,</span> <span class="n">and1</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">proof</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"proof"</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="n">res</span><span class="p">);</span>

  <span class="cm">/* Proof checking */</span>
  <span class="n">assert</span><span class="p">(</span><span class="n">check_proof</span><span class="p">(</span><span class="n">proof</span><span class="p">));</span>
  <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
</details>

### (**) Exercise 4
Read [the Wikipedia page of the pigeonhole
principle](https://en.wikipedia.org/wiki/Pigeonhole_principle). State
the principle for 1 hole and 2 pigeons, and prove that it is
unsatisfiable.

<details>
<summary>Tip 1</summary>
<p>One can use 2 Boolean variables: <code class="language-plaintext highlighter-rouge">xi</code> represents the fact that pigeon
<code class="language-plaintext highlighter-rouge">i</code> is in the hole.</p>
</details>

<details>
<summary>Tip 2</summary>
<p>There are two conditions:</p>
<ol>
  <li>every pigeon must be in the hole</li>
  <li>the hole cannot contain two pigeons
</li>
</ol>
</details>

<details>
<summary>Tip 3</summary>
<p>The two conditions can be translated as:</p>
<ol>
  <li><code class="language-plaintext highlighter-rouge">x1</code> and <code class="language-plaintext highlighter-rouge">x2</code> are <code class="language-plaintext highlighter-rouge">true</code></li>
  <li><code class="language-plaintext highlighter-rouge">¬x1 ∨ ¬x2</code> is true
</li>
</ol>
</details>

<details>
<summary>Tip 4</summary>
<p>The proof consists in destroying the disjunction in condition 2, then
resolving with the two conditions 1.
</p>
</details>

<details>
<summary>Solution</summary>
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">**</span> <span class="n">argv</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">caml_startup</span><span class="p">(</span><span class="n">argv</span><span class="p">);</span>
  <span class="n">start_smt2</span><span class="p">();</span>

  <span class="cm">/* The 2 variables */</span>
  <span class="n">FUNSYM</span> <span class="n">x1symb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x1"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">FUNSYM</span> <span class="n">x2symb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x2"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">x1symb</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">x2symb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x1</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">x1symb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x2</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">x2symb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>

  <span class="cm">/* Every pigeon is in the hole */</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">x1</span><span class="p">);</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">x2</span><span class="p">);</span>

  <span class="cm">/* The hole cannot contain more than one pigeon */</span>
  <span class="n">EXPR</span> <span class="n">args</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">x1</span><span class="p">),</span> <span class="n">enot</span><span class="p">(</span><span class="n">x2</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">args</span><span class="p">));</span>

  <span class="cm">/* Certif: assertions */</span>
  <span class="n">CERTIF</span> <span class="n">ass0</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass0"</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass1</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass1"</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass2</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass2"</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>

  <span class="cm">/* Certif: or rule */</span>
  <span class="n">CERTIF</span> <span class="n">or</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or"</span><span class="p">,</span> <span class="n">ass2</span><span class="p">);</span>

  <span class="cm">/* Certif: resolution */</span>
  <span class="n">CERTIF</span> <span class="n">res</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">or</span><span class="p">,</span> <span class="n">ass0</span><span class="p">,</span> <span class="n">ass1</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">proof</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"proof"</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="n">res</span><span class="p">);</span>

  <span class="cm">/* Check the proof */</span>
  <span class="n">assert</span><span class="p">(</span><span class="n">check_proof</span><span class="p">(</span><span class="n">proof</span><span class="p">));</span>
  <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
</details>

### (***) Exercise 5
State the pigeonhole principle for 2 holes and 3 pigeons, and prove that
it is unsatisfiable.

<details>
<summary>Tip 1</summary>
<p>Again, one can use 6 Boolean variables: <code class="language-plaintext highlighter-rouge">xij</code> represents the fact that
pigeon <code class="language-plaintext highlighter-rouge">i</code> is in hole <code class="language-plaintext highlighter-rouge">j</code>. The conditions are similar to Exercise 4.
</p>
</details>

<details>
<summary>Tip 2</summary>
<p>First prove that pigeon 1 is in hole 1.</p>
</details>

<details>
<summary>Tip 3</summary>
<p>Then deduce that pigeon 2 cannot be in hole 1, and that pigeon 3 cannot
be in hole 1.
</p>
</details>

<details>
<summary>Tip 4</summary>
<p>Then deduce that pigeon 2 must be in hole 2, and that pigeon 3 must also
be in hole 2. Conclude.
</p>
</details>

<details>
<summary>Tip 5</summary>
<p>Do not forget to break <code class="language-plaintext highlighter-rouge">or</code>s first.
</p>
</details>

<details>
<summary>Solution</summary>
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">**</span> <span class="n">argv</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">caml_startup</span><span class="p">(</span><span class="n">argv</span><span class="p">);</span>
  <span class="n">start_smt2</span><span class="p">();</span>

  <span class="cm">/* The 6 variables */</span>
  <span class="n">FUNSYM</span> <span class="n">x11symb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x11"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">FUNSYM</span> <span class="n">x12symb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x12"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">FUNSYM</span> <span class="n">x21symb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x21"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">FUNSYM</span> <span class="n">x22symb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x22"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">FUNSYM</span> <span class="n">x31symb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x31"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">FUNSYM</span> <span class="n">x32symb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x32"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Bool"</span><span class="p">));</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">x11symb</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">x12symb</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">x21symb</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">x22symb</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">x31symb</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">x32symb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x11</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">x11symb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x12</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">x12symb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x21</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">x21symb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x22</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">x22symb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x31</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">x31symb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x32</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">x32symb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>

  <span class="cm">/* Every pigeon is in a hole */</span>
  <span class="n">EXPR</span> <span class="n">p1</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">x11</span><span class="p">,</span> <span class="n">x12</span><span class="p">};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">p1</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">p2</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">x21</span><span class="p">,</span> <span class="n">x22</span><span class="p">};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">p2</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">p3</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">x31</span><span class="p">,</span> <span class="n">x32</span><span class="p">};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">p3</span><span class="p">));</span>

  <span class="cm">/* A hole cannot contain more than one pigeon */</span>
  <span class="n">EXPR</span> <span class="n">h1</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">x11</span><span class="p">),</span> <span class="n">enot</span><span class="p">(</span><span class="n">x21</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">h1</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">h2</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">x11</span><span class="p">),</span> <span class="n">enot</span><span class="p">(</span><span class="n">x31</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">h2</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">h3</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">x21</span><span class="p">),</span> <span class="n">enot</span><span class="p">(</span><span class="n">x31</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">h3</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">h4</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">x12</span><span class="p">),</span> <span class="n">enot</span><span class="p">(</span><span class="n">x22</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">h4</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">h5</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">x12</span><span class="p">),</span> <span class="n">enot</span><span class="p">(</span><span class="n">x32</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">h5</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">h6</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">x22</span><span class="p">),</span> <span class="n">enot</span><span class="p">(</span><span class="n">x32</span><span class="p">)};</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eor</span><span class="p">(</span><span class="mi">2</span><span class="p">,</span> <span class="n">h6</span><span class="p">));</span>

  <span class="cm">/* Certif: assertions */</span>
  <span class="n">CERTIF</span> <span class="n">ass0</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass0"</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass1</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass1"</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass2</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass2"</span><span class="p">,</span> <span class="mi">2</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass3</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass3"</span><span class="p">,</span> <span class="mi">3</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass4</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass4"</span><span class="p">,</span> <span class="mi">4</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass5</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass5"</span><span class="p">,</span> <span class="mi">5</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass6</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass6"</span><span class="p">,</span> <span class="mi">6</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass7</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass7"</span><span class="p">,</span> <span class="mi">7</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass8</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass8"</span><span class="p">,</span> <span class="mi">8</span><span class="p">);</span>

  <span class="cm">/* Certif: or rules */</span>
  <span class="n">CERTIF</span> <span class="n">or0</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or0"</span><span class="p">,</span> <span class="n">ass0</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">or1</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or1"</span><span class="p">,</span> <span class="n">ass1</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">or2</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or2"</span><span class="p">,</span> <span class="n">ass2</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">or3</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or3"</span><span class="p">,</span> <span class="n">ass3</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">or4</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or4"</span><span class="p">,</span> <span class="n">ass4</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">or5</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or5"</span><span class="p">,</span> <span class="n">ass5</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">or6</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or6"</span><span class="p">,</span> <span class="n">ass6</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">or7</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or7"</span><span class="p">,</span> <span class="n">ass7</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">or8</span> <span class="o">=</span> <span class="n">cor</span><span class="p">(</span><span class="s">"or8"</span><span class="p">,</span> <span class="n">ass8</span><span class="p">);</span>

  <span class="cm">/* Certif: prove that pigeon 1 is in hole 1 */</span>
  <span class="n">CERTIF</span> <span class="n">res1</span><span class="p">[</span><span class="mi">4</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">or0</span><span class="p">,</span> <span class="n">or6</span><span class="p">,</span> <span class="n">or1</span><span class="p">,</span> <span class="n">or5</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">reso1</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"reso1"</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="n">res1</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">res2</span><span class="p">[</span><span class="mi">4</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">or0</span><span class="p">,</span> <span class="n">or7</span><span class="p">,</span> <span class="n">or2</span><span class="p">,</span> <span class="n">reso1</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">reso2</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"reso2"</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="n">res2</span><span class="p">);</span>

  <span class="cm">/* Certif: deduce that pigeon 2 cannot be in hole 1, and that pigeon 3
     cannot be in hole 1 */</span>
  <span class="n">CERTIF</span> <span class="n">res3</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">reso2</span><span class="p">,</span> <span class="n">or3</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">reso3</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"reso3"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">res3</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">res4</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">reso2</span><span class="p">,</span> <span class="n">or4</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">reso4</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"reso4"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">res4</span><span class="p">);</span>

  <span class="cm">/* Certif: deduce that pigeon 2 must be in hole 2, and that pigeon 3
     must also be in hole 2 */</span>
  <span class="n">CERTIF</span> <span class="n">res5</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">reso3</span><span class="p">,</span> <span class="n">or1</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">reso5</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"reso5"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">res5</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">res6</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">reso4</span><span class="p">,</span> <span class="n">or2</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">reso6</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"reso6"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">res6</span><span class="p">);</span>

  <span class="cm">/* Conclude */</span>
  <span class="n">CERTIF</span> <span class="n">resp</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">or8</span><span class="p">,</span> <span class="n">reso5</span><span class="p">,</span> <span class="n">reso6</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">proof</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"proof"</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="n">resp</span><span class="p">);</span>

  <span class="n">assert</span><span class="p">(</span><span class="n">check_proof</span><span class="p">(</span><span class="n">proof</span><span class="p">));</span>
  <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
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
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">**</span> <span class="n">argv</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">caml_startup</span><span class="p">(</span><span class="n">argv</span><span class="p">);</span>
  <span class="n">start_smt2</span><span class="p">();</span>

  <span class="cm">/* The uninterpreted sort */</span>
  <span class="n">SORT</span> <span class="n">u</span> <span class="o">=</span> <span class="n">sort</span><span class="p">(</span><span class="s">"U"</span><span class="p">);</span>
  <span class="n">declare_sort</span><span class="p">(</span><span class="n">u</span><span class="p">);</span>

  <span class="cm">/* A variable of this sort */</span>
  <span class="n">FUNSYM</span> <span class="n">asymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"a"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">u</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">asymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">a</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">asymb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>

  <span class="cm">/* A function symbol of type U → U */</span>
  <span class="n">FUNSYM</span> <span class="n">fsymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"f"</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">u</span><span class="p">,</span> <span class="n">u</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">fsymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">fa</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">fsymb</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">a</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">ffa</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">fsymb</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">fa</span><span class="p">);</span>

  <span class="cm">/* The two assertions */</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eeq</span><span class="p">(</span><span class="n">a</span><span class="p">,</span> <span class="n">fa</span><span class="p">));</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">enot</span><span class="p">(</span><span class="n">eeq</span><span class="p">(</span><span class="n">ffa</span><span class="p">,</span> <span class="n">a</span><span class="p">)));</span>

  <span class="cm">/* Certificate: assertions */</span>
  <span class="n">CERTIF</span> <span class="n">ass0</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass0"</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass1</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass1"</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>

  <span class="cm">/* Certificate: congruence */</span>
  <span class="n">EXPR</span> <span class="n">clause</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">eeq</span><span class="p">(</span><span class="n">a</span><span class="p">,</span> <span class="n">fa</span><span class="p">)),</span> <span class="n">eeq</span><span class="p">(</span><span class="n">fa</span><span class="p">,</span> <span class="n">ffa</span><span class="p">)};</span>
  <span class="n">CERTIF</span> <span class="n">congr</span> <span class="o">=</span> <span class="n">ceq_congruent</span><span class="p">(</span><span class="s">"congr"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">clause</span><span class="p">);</span>

  <span class="cm">/* Certificate: transitivity */</span>
  <span class="n">EXPR</span> <span class="n">es</span><span class="p">[</span><span class="mi">3</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">a</span><span class="p">,</span> <span class="n">fa</span><span class="p">,</span> <span class="n">ffa</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">trans</span> <span class="o">=</span> <span class="n">ceq_transitive</span><span class="p">(</span><span class="s">"trans"</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="n">es</span><span class="p">);</span>

  <span class="cm">/* Certificate: resolution */</span>
  <span class="n">CERTIF</span> <span class="n">res</span><span class="p">[</span><span class="mi">4</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">congr</span><span class="p">,</span> <span class="n">trans</span><span class="p">,</span> <span class="n">ass0</span><span class="p">,</span> <span class="n">ass1</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">proof</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"proof"</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="n">res</span><span class="p">);</span>

  <span class="cm">/* Proof checking */</span>
  <span class="n">assert</span><span class="p">(</span><span class="n">check_proof</span><span class="p">(</span><span class="n">proof</span><span class="p">));</span>
  <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
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

> Note:
> The literals in the clause given to `lia_generic` may belong to
> multiple theories, but the proof that it is a tautology must use only
> linear integer arithmetic reasoning.

### (**) Exercise 8
Given a function symbol `f` of type `Int → Int`, and two variable `x`
and `y` of type `Int`, prove that the conjunction of `y = x+1` and
`¬(f(x) = f(y-1))` is unsatisfiable.

<details>
<summary>Tip</summary>
<p>Remember that the sort <code class="language-plaintext highlighter-rouge">"Int</code>” is interpreted and can be defined using
<code class="language-plaintext highlighter-rouge">sort("Int")</code>, as presented in <a href="doc/capi/group__sort.html">the documentation about
sorts</a>.
</p>
</details>

<details>
<summary>Solution</summary>
<div class="language-c highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kt">int</span> <span class="nf">main</span><span class="p">(</span><span class="kt">int</span> <span class="n">argc</span><span class="p">,</span> <span class="kt">char</span> <span class="o">**</span> <span class="n">argv</span><span class="p">)</span>
<span class="p">{</span>
  <span class="n">caml_startup</span><span class="p">(</span><span class="n">argv</span><span class="p">);</span>
  <span class="n">start_smt2</span><span class="p">();</span>

  <span class="cm">/* The two variables */</span>
  <span class="n">SORT</span> <span class="n">s</span> <span class="o">=</span> <span class="n">sort</span><span class="p">(</span><span class="s">"Int"</span><span class="p">);</span>
  <span class="n">FUNSYM</span> <span class="n">xsymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"x"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">s</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">xsymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">x</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">xsymb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>
  <span class="n">FUNSYM</span> <span class="n">ysymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"y"</span><span class="p">,</span> <span class="mi">0</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">,</span> <span class="n">s</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">ysymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">y</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">ysymb</span><span class="p">,</span> <span class="nb">NULL</span><span class="p">);</span>

  <span class="cm">/* A function symbol of type Int → Int */</span>
  <span class="n">FUNSYM</span> <span class="n">fsymb</span> <span class="o">=</span> <span class="n">funsym</span><span class="p">(</span><span class="s">"f"</span><span class="p">,</span> <span class="mi">1</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">s</span><span class="p">,</span> <span class="n">s</span><span class="p">);</span>
  <span class="n">declare_fun</span><span class="p">(</span><span class="n">fsymb</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">fx</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">fsymb</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">x</span><span class="p">);</span>
  <span class="n">EXPR</span> <span class="n">yminone</span> <span class="o">=</span> <span class="n">eminus</span><span class="p">(</span><span class="n">y</span><span class="p">,</span> <span class="n">eint</span><span class="p">(</span><span class="mi">1</span><span class="p">));</span>
  <span class="n">EXPR</span> <span class="n">fyminone</span> <span class="o">=</span> <span class="n">efun</span><span class="p">(</span><span class="n">fsymb</span><span class="p">,</span> <span class="o">&amp;</span><span class="n">yminone</span><span class="p">);</span>

  <span class="cm">/* The two assertions */</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">eeq</span><span class="p">(</span><span class="n">y</span><span class="p">,</span> <span class="n">eadd</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">eint</span><span class="p">(</span><span class="mi">1</span><span class="p">))));</span>
  <span class="n">assertf</span><span class="p">(</span><span class="n">enot</span><span class="p">(</span><span class="n">eeq</span><span class="p">(</span><span class="n">fx</span><span class="p">,</span> <span class="n">fyminone</span><span class="p">)));</span>

  <span class="cm">/* Certificate: assertions */</span>
  <span class="n">CERTIF</span> <span class="n">ass0</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass0"</span><span class="p">,</span> <span class="mi">0</span><span class="p">);</span>
  <span class="n">CERTIF</span> <span class="n">ass1</span> <span class="o">=</span> <span class="n">cassume</span><span class="p">(</span><span class="s">"ass1"</span><span class="p">,</span> <span class="mi">1</span><span class="p">);</span>

  <span class="cm">/* Certificate: LIA */</span>
  <span class="n">EXPR</span> <span class="n">clauselia</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">eeq</span><span class="p">(</span><span class="n">y</span><span class="p">,</span> <span class="n">eadd</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">eint</span><span class="p">(</span><span class="mi">1</span><span class="p">)))),</span> <span class="n">eeq</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">eminus</span><span class="p">(</span><span class="n">y</span><span class="p">,</span> <span class="n">eint</span><span class="p">(</span><span class="mi">1</span><span class="p">)))};</span>
  <span class="n">CERTIF</span> <span class="n">lia</span> <span class="o">=</span> <span class="n">clia_generic</span><span class="p">(</span><span class="s">"lia"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">clauselia</span><span class="p">);</span>

  <span class="cm">/* Certificate: congruence */</span>
  <span class="n">EXPR</span> <span class="n">clause</span><span class="p">[</span><span class="mi">2</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">enot</span><span class="p">(</span><span class="n">eeq</span><span class="p">(</span><span class="n">x</span><span class="p">,</span> <span class="n">eminus</span><span class="p">(</span><span class="n">y</span><span class="p">,</span> <span class="n">eint</span><span class="p">(</span><span class="mi">1</span><span class="p">)))),</span> <span class="n">eeq</span><span class="p">(</span><span class="n">fx</span><span class="p">,</span> <span class="n">fyminone</span><span class="p">)};</span>
  <span class="n">CERTIF</span> <span class="n">congr</span> <span class="o">=</span> <span class="n">ceq_congruent</span><span class="p">(</span><span class="s">"congr"</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="n">clause</span><span class="p">);</span>

  <span class="cm">/* Certificate: resolution */</span>
  <span class="n">CERTIF</span> <span class="n">res</span><span class="p">[</span><span class="mi">4</span><span class="p">]</span> <span class="o">=</span> <span class="p">{</span><span class="n">congr</span><span class="p">,</span> <span class="n">lia</span><span class="p">,</span> <span class="n">ass0</span><span class="p">,</span> <span class="n">ass1</span><span class="p">};</span>
  <span class="n">CERTIF</span> <span class="n">proof</span> <span class="o">=</span> <span class="n">cresolution</span><span class="p">(</span><span class="s">"proof"</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="n">res</span><span class="p">);</span>

  <span class="cm">/* Proof checking */</span>
  <span class="n">assert</span><span class="p">(</span><span class="n">check_proof</span><span class="p">(</span><span class="n">proof</span><span class="p">));</span>
  <span class="k">return</span> <span class="mi">0</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
</details>
