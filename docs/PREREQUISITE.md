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

Skip this phase if you already have a working knowledge of type systems. Both books below are
large — you don't need to read either cover to cover. The chapter-level breakdown is given so you
can target exactly the material that bears on Emet's design claims.

### *Types and Programming Languages* (TAPL) by Benjamin Pierce

**Must-read:**

- **Chapter 1 — Introduction.** Sets up why type systems exist and what safety guarantees they're
  for. Necessary framing for everything after.
- **Chapter 8 — Typed Arithmetic Expressions.** Introduces **Progress** ("well-typed terms aren't
  stuck") and **Preservation** ("evaluation preserves types") — the two theorems that formal
  "this program can't crash" claims are built on. This is the actual mathematical backbone behind
  Emet's "mathematically proven to be bug-free" language. Short chapter, high payoff.
- **Chapter 9 — Simply Typed Lambda-Calculus.** The minimal typed language with function types.
  You need this vocabulary (typing rules, type judgments) to read anything else, including
  Dafny/F* syntax later.
- **Chapter 12 — Normalization.** The single most directly relevant chapter for the
  Recurrence-Node/ranking-function critique. Proves that every well-typed term in the
  simply-typed lambda calculus terminates, using Tait's method of logical relations — a formal
  ancestor of exactly the "every loop must provably terminate" idea Emet leans on. Read this one
  closely.
- **Chapter 13 — References.** Introduces mutable state into a typed calculus — directly relevant
  to Emet's `std::io`/effects gap. Seeing how state gets typed at all is a prerequisite for
  evaluating whether "Monadic Effect Nodes" (one paragraph in the current docs) is remotely
  sufficient.
- **Chapter 14 — Exceptions.** Non-local control flow and typed error handling — relevant to how
  Emet would need to handle failure paths (division-by-zero rejection, "Truth Violation" errors)
  formally rather than just diagrammatically.

**Worth reading:**

- **Chapter 11 — Simple Extensions.** Covers `fix` (general recursion) and variants/sums. The
  `fix` operator is exactly the mechanism that breaks the guarantees proven in Chapter 12 — i.e.,
  this chapter shows you *why* unrestricted recursion and "provably terminates" are in tension,
  which is the crux of the termination-model critique.
- **Chapter 15 — Subtyping.** Useful background for Dafny/F*'s richer type systems, and for
  understanding Contract/Realization separation as a form of behavioral subtyping.
- **Chapter 20 — Recursive Types.** Needed to formally type self-referential structures like
  graphs/trees — relevant background for reasoning rigorously about the AIR graph itself as a
  typed data structure.
- **Chapter 24 — Existential Types.** The formal underpinning of "abstract data types /
  information hiding" — the closest classical analog to Emet's **Contract Node / Candidate
  Realization** split (a contract as an existential interface, realizations as its witnesses).

**Skip for now:** Chapters 2–7 (mechanical setup and de Bruijn indices — useful if you want to
*implement* a typechecker yourself, less so for evaluating Emet's claims); Chapters 16–19
(subtyping metatheory / OO case studies); Chapters 22–23, 25 (Hindley-Milner inference, System F);
Chapters 26–32 (bounded/higher-order polymorphism — genuinely advanced, not load-bearing here).

**If you only have time for four TAPL chapters:** 1, 8, 12, 13 — the minimum set that gets you
Progress/Preservation, termination proving, and typed state.

### Programming Language Foundations in Agda (PLFA)

PLFA covers similar ground to TAPL but with machine-checked proofs in Agda, and doubles as a
gentle, hands-on introduction to dependent types (see Phase 3). It's organized into three parts:
Logical Foundations, Programming Language Foundations, and Denotational Semantics.

**Must-read:**

- **Induction** (Part 1). Proof by induction, done properly and interactively. This is the
  foundation the ranking-function/ termination proofs in Recurrence Nodes ultimately rest on —
  seeing induction proved by hand in Agda makes "prove this loop terminates" concrete rather than
  hand-wavy.
- **Relations** (Part 1). Inductively-defined relations — directly relevant, since this is the
  formal machinery you'd use to define something like a Recurrence Node's step relation
  precisely.
- **Properties** (Part 2). Agda's own treatment of Progress and Preservation — a machine-checked
  counterpart to TAPL Chapter 8. Reading both versions back to back is genuinely clarifying.
- **Untyped: Untyped lambda calculus with full normalisation** (Part 2). Despite the name, this
  chapter is about proving normalization (termination) using strong computability/logical
  relations, in the same spirit as TAPL Chapter 12, but with the full machine-checked proof
  visible. This is the most direct, hands-on parallel to the termination-proving question at the
  heart of the Recurrence Node critique.

**Worth reading:**

- **Decidable** (Part 1). Booleans and decision procedures — a gentle, concrete introduction to
  what "decidable" means formally, which is directly relevant to why SMT solvers can return
  `unknown` on undecidable fragments.
- **Quantifiers** (Part 1). Universals and existentials — parallels TAPL Chapter 24 and is useful
  background for the Contract/Realization split.
- **Lambda** (Part 2). Introduction to the lambda calculus — largely redundant with TAPL Chapter 5
  if you've already read that, but useful as a refresher in Agda's notation.

**Skip for now:** Naturals, Equality, Isomorphism, Connectives, Negation, Lists (general logical/
Agda foundations, useful if you want fluency in Agda itself but not load-bearing for evaluating
Emet); DeBruijn, More, Bisimulation, Inference, Confluence, BigStep (implementation-level Part 2
material); all of Part 3, Denotational Semantics (advanced, not load-bearing for this critique).

**Relevance to Emet:** Emet's "refinement types," "invariants," and "constraints" are standard
type-system vocabulary under different names. This phase makes the rest of the material legible,
and the Normalization/Untyped chapters specifically are the clearest formal treatment of the
termination question that recurs throughout the critique.

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
