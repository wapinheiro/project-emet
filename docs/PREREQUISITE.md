# Prerequisites & Study Guide for Project Emet

## Purpose

Project Emet draws on several established ideas from programming language theory and formal
methods — refinement types, SMT-backed verification, dependent types, well-founded recursion,
content-addressed/projectional code representations, and effect systems. Understanding these
underlying concepts first makes it much easier to evaluate, critique, and contribute to Emet's
design, since most of Emet's architecture is a synthesis of real prior art rather than something
invented from scratch.

This guide is for anyone joining the project who wants to build that background before diving
into the `docs/concepts/` documents. It's organized as a rough learning path, roughly in the
order we'd recommend working through it, with a note on **why each stop matters specifically for
understanding Emet**.

None of this is mandatory to contribute — but working through even Phase 1–2 below (roughly
2–3 weeks of casual study) will let you evaluate most of Emet's core design claims firsthand
rather than taking them on faith.

---

## Phase 1: Foundations

Skip this phase if you already have a working knowledge of type systems.

- **[*Types and Programming Languages*](https://www.cis.upenn.edu/~bcpierce/tapl/) by Benjamin
  Pierce** — the standard reference. You don't need the whole book to start; the introductory
  chapters and the chapter on subtyping are the most relevant here.
- **[Programming Language Foundations in Agda (PLFA)](https://plfa.github.io/)** — free, and
  doubles as a gentle introduction to dependent types (see Phase 3), so it's a good alternative
  or supplement to Pierce if you want something more hands-on and interactive.

**Relevance to Emet:** Emet's "refinement types," "invariants," and "constraints" are standard
type-system vocabulary under different names. This phase makes the rest of the material legible.

---

## Phase 2: SMT Solvers

This is the single most load-bearing piece of Emet's architecture — nearly every verification
claim in the `docs/concepts/` documents depends on SMT solving.

- **[Z3 Programming Guide](https://microsoft.github.io/z3guide/)** — Z3's own tutorial. Do the
  exercises, don't just read them. Try the Python bindings and write a few toy constraints
  yourself: e.g., prove a loop terminates, or prove a division is safe given an upstream check.
- **[*Decision Procedures: An Algorithmic Point of View*](https://link.springer.com/book/10.1007/978-3-662-50497-0)
  by Kroening & Strichman** — for real depth on how SMT solving works internally, and why it can
  fail to terminate or return "unknown" on certain classes of constraints.

**Relevance to Emet:** Once you've personally hit a case where Z3 returns `unknown` or times out,
the project's open question about how the compilation pipeline should handle non-SAT/UNSAT
outcomes becomes concrete rather than abstract.

---

## Phase 3: Dependent Types and Refinement Types

- **[Idris 2 documentation](https://idris2.readthedocs.io/)**, or *Type-Driven Development with
  Idris* by Edwin Brady — Idris has an explicit **totality checker** (`total` keyword), which is
  the closest real-world analog to Emet's Recurrence Nodes and ranking functions. Seeing how
  Idris handles (and sometimes struggles with) totality checking directly informs one of Emet's
  open design questions around loop termination.
- **[LiquidHaskell](https://ucsd-progsys.github.io/liquidhaskell/)** — refinement types bolted
  onto an existing practical language (e.g. `{v:Int | v /= 0}` for a safe divisor). This is the
  closest working analog to what `emet-std`'s axiomatic constraints are trying to achieve.

**Relevance to Emet:** Semantic Nodes and axiomatic constraints are, structurally, refinement
types embedded in a graph. This phase is probably the single most useful one for evaluating
Emet's core data model.

---

## Phase 4: Verification-First Languages

- **[Dafny](https://dafny.org/)** — genuinely the best entry point for verification-oriented
  programming; has an in-browser playground, no install needed. Work through a few exercises
  proving loop invariants and postconditions by hand, and notice how often you need to supply
  hints or lemmas rather than the solver figuring it out unaided.
- **[F\*](https://www.fstar-lang.org/)** — more powerful and more research-flavored, used in real
  verified systems (e.g. parts of HACL* and Project Everest/TLS verification). A skim is enough
  unless you want to go deeper; the goal is to see how a mature verification language handles
  effects and proof obligations.

**Relevance to Emet:** Dafny is close to "Emet, if it existed and shipped today." Working through
even basic Dafny exercises lets you evaluate Emet's correctness and efficiency claims against a
real, working comparison point.

---

## Phase 5: Termination Proving

- Read the sections on **structural recursion** and **well-founded recursion** in the Agda or
  Idris documentation (both cover this, since it underlies their totality checkers).
- Optional: ACL2's documentation on its measure/ranking-function approach — ACL2 is an
  industrially-used theorem prover (notably used at AMD for chip verification) built around
  exactly this technique.

**Relevance to Emet:** Maps directly onto the Recurrence Node / ranking function design. This
phase clarifies why "every loop needs a strictly decreasing rank" is a real and useful, but
genuinely hard-in-general, technique — and why Emet needs an explicit answer for loops that are
intentionally non-terminating (servers, event loops, daemons).

---

## Phase 6: Content-Addressed and Projectional Code Representations

- **[Unison](https://www.unison-lang.org/)** — install it and write a few functions. Pay
  attention to how renaming works (names as aliases over a content hash) and how the
  codebase/UCM tooling handles editing — this is the closest real-world analog to Emet's
  Projection Layer.
- **JetBrains MPS** (optional, more niche) — a projectional editor where the AST is the source of
  truth and text is rendered/edited through projections. Worth reading their docs and conference
  talks, which are candid about how hard round-tripping and tooling ergonomics are in practice.

**Relevance to Emet:** Directly informs the open question around reverse projection (mapping
arbitrary human text edits back to graph mutations) — you'll see shipped systems wrestling with
exactly this problem.

---

## Phase 7: Effects and Concurrency

- **[Koka](https://koka-lang.github.io/koka/doc/index.html)** — a research language with a
  first-class, well-designed effect system. Even a shallow pass shows how much machinery a real
  effect system needs, compared to a brief treatment of I/O and side effects.

**Relevance to Emet:** Effects, concurrency, and I/O are currently the least-specified part of
Emet's architecture (see `docs/concepts/emet-standard-library.md`, `std::io`). This phase gives
useful reference points for what a fuller design would need to cover.

---

## Suggested Pacing

A realistic path that builds naturally on itself:

1. **Z3 tutorial, hands-on** — ~1 week
2. **Dafny tutorial, hands-on** — ~1–2 weeks (this alone will let you re-read Emet's
   `docs/concepts/` with much sharper eyes)
3. **Idris/Agda totality + LiquidHaskell** — ~2 weeks
4. **Unison, hands-on** — 3–5 days
5. **Skim F\* and Koka** — a few days each; no need to go deep unless you're specifically working
   on the effects/verification-scope side of the project

Steps 1–2 alone (roughly 2–3 weeks of casual study) will let you evaluate most of Emet's core
design claims yourself, since Dafny is essentially a working, shipped version of what Emet is
trying to build. Everything after that deepens specific areas rather than being a strict
prerequisite.

---

## After This Guide

Once you've worked through Phases 1–4 at minimum, we'd suggest reading (or re-reading) the
`docs/concepts/` documents in this order, since later ones depend on ideas from earlier ones:

1. `semantic-nodes.md`
2. `the-projection-layer.md`
3. `architecture-and-compilation.md`
4. `logic-graphs-and-control-flow.md`
5. `emet-standard-library.md`
6. `candidate-realizations.md`
7. `semantic-graph-diffs.md`
8. `tokens-resources-waste.md`

If you have feedback, open questions, or disagreements after going through this material, the
`rfcs/` directory is the right place to propose changes — see `rfcs/README.md` for the process
and template.
