# External Critique: Project Emet Architecture

**Author(s):** External review (drafted with Claude, at the request of the project maintainer)
**Status:** Draft — for maintainer review and possible splitting into individual RFCs
**Scope:** README.md, docs/MANIFESTO.md, docs/concepts/*.md (all 9 concept documents)

---

## Summary

Project Emet proposes an AI-Native Intermediate Representation (AIR) — a multi-dimensional
semantic graph — as the canonical "source of truth" for programs, with human-readable source
code demoted to a generated, bi-directional projection. The design combines several real,
individually well-established ideas from formal methods and language design: content-addressed
graph representations, refinement types, SMT-backed verification, contract/implementation
separation, and well-founded recursion (termination proving via ranking functions).

This is a legitimate and often clever synthesis. The critique below is offered in that spirit:
the goal is to separate the parts of the vision that are buildable and defensible today from the
parts that either overstate what's achievable, or quietly assume away problems that are still
open research questions. Where possible, each issue below is written so it can become its own
RFC using the repo's existing RFC template.

The single highest-priority issue is **#1 (Termination-Only Model)**, because it currently makes
Emet unable to express an entire, large class of real software (anything designed to run
indefinitely), and no document in the repo acknowledges this as a deliberate scope decision.
Recommend resolving this first, since it affects how every other RFC should be scoped.

---

## Motivation

Project Emet's stated goals — reducing token waste in AI-assisted coding, catching bugs before
execution rather than after, decoupling logic from syntax, making refactors and diffs
semantically meaningful — are all worth pursuing, and the current documentation set makes a
compelling emotional and rhetorical case for them. But a few of the documents' central claims
depend on results that are unsolved or only partially solved in the formal methods literature
(general termination proving, robust bidirectional text/graph synchronization, tractable SMT
solving over realistic program constraints). Presenting these as solved architectural details
rather than open problems risks:

- Setting expectations (with contributors, users, or investors) that the current design cannot
  meet.
- Missing the opportunity to scope the project toward domains where it is *already* strong —
  safety-critical, embedded, and verification-heavy software — rather than "all software,
  including web servers and operating systems," which the current model cannot express at all.
- Under-investing in the two hardest and most novel parts of the system (reverse projection,
  effect/concurrency modeling), because the current docs treat them as solved corollaries of the
  graph model rather than as the primary research risk.

---

## Detailed Findings

### 1. The Termination-Only Model Silently Excludes an Entire Class of Real Software

**Where it appears:** `docs/concepts/logic-graphs-and-control-flow.md`

Every loop in Emet is represented as a Recurrence Node, which must carry a ranking function
`R(S)` proven by the SMT solver to strictly decrease (`R(S_{k+1}) < R(S_k)`) and remain bounded
below by zero. This is a real and useful technique — it's essentially *well-founded recursion* /
*total functional programming*, as used in Agda, Idris (in total mode), and ACL2. It genuinely
does rule out a class of infinite-loop bugs.

But taken as stated, with no exceptions, it also means **every loop in an Emet program must be
provably finite**. That excludes:

- Web/API servers (`while True: accept_connection()`)
- Event loops, message queues, daemons
- OS schedulers, REPLs, long-running game loops
- Any process whose entire purpose is to run until externally terminated

This is not an edge case — it's most of the software that currently gets built. None of the nine
concept documents mention this as a deliberate scope decision, nor propose an escape hatch (for
example, an explicit "unbounded" or "coinductive" node type, analogous to the `Data`/`Codata`
distinction in total functional languages, for subgraphs that are intentionally long-running and
exempted from the ranking-function requirement).

**Recommendation:** Decide explicitly whether Emet targets (a) only provably-terminating programs
(a legitimate, narrower scope matching safety-critical/embedded software), or (b) general-purpose
software including long-running processes, in which case a second node type for intentionally
unbounded execution needs to be designed — including how such subgraphs interact with the
verification guarantees the rest of the system depends on.

---

### 2. Even Within "Terminating" Programs, Realistic Ranking Functions Are Often Hard to Find

**Where it appears:** `docs/concepts/logic-graphs-and-control-flow.md`, worked example

The order-queue example works because each iteration removes exactly one item
(`length(queue_{k+1}) == length(queue_k) - 1`). Many ordinary, correct programs don't have such a
clean decreasing metric:

- Retry queues, where a failed item is re-enqueued (queue length can increase)
- Producer/consumer systems where the queue's size is not monotonic
- Work-stealing schedulers
- Iterative algorithms that converge (e.g., numerical methods) rather than shrink a discrete
  counter

Finding valid ranking functions for realistic recursive/iterative algorithms is a well-known hard
problem in the total-functional-programming literature — this is a large part of why total
languages haven't seen mainstream production adoption despite decades of academic work.

**Recommendation:** Either (a) invest in a richer termination-metric system beyond "one integer
strictly decreases" (lexicographic orders, well-founded relations over more complex domains,
solver-assisted synthesis of ranking functions), or (b) provide a documented fallback for loops
whose termination can't be automatically proven — e.g., accepted with a runtime bound/assertion
and flagged as "unverified" rather than rejected outright.

---

### 3. SMT Solvers Can Return "Unknown" or Time Out — the Pipeline Has No Path for This

**Where it appears:** `docs/concepts/architecture-and-compilation.md` (Stage 2),
`docs/concepts/emet-standard-library.md`, `docs/concepts/logic-graphs-and-control-flow.md`

Every worked example in the documentation resolves cleanly to SAT or UNSAT. In practice, SMT
solvers such as Z3 or CVC5 frequently return `unknown` or fail to terminate within a usable time
budget on nonlinear arithmetic, unbounded quantifiers, or complex data-structure theories. This is
not a rare failure mode — it's the standard reason production verification systems (Dafny, F*,
Why3) ship extensive tooling for manual proof hints, lemma libraries, and proof-assistance,
rather than relying on push-button solving alone.

**Recommendation:** Add an explicit third outcome to the Stage 2 pipeline diagram
(`SAT` / `UNSAT` / `UNKNOWN-OR-TIMEOUT`), and define what happens on that path — options include:
falling back to runtime assertions, requiring a human-supplied proof hint or lemma, or rejecting
compilation with a specific "unprovable, not disproven" diagnostic (distinct from "proven
unsafe").

---

### 4. Reverse Projection (Human Edit → Graph Mutation) Is the Riskiest Unsolved Problem, and Gets the Least Detail

**Where it appears:** `docs/concepts/the-projection-layer.md`

The forward direction (graph → human-readable code) is well-specified and plausible — this is
essentially a code-generation/pretty-printing problem, which is tractable. The reverse direction
(parsing an arbitrary human text edit and mapping it unambiguously back to a specific graph
mutation) is asserted via clean examples (changing a single literal value) but not addressed for
the general case: what happens when a human restructures a loop, introduces a new intermediate
variable, extracts a function, or writes something the projector has never encountered? Real
prior art in this space (Unison's syntax tooling, JetBrains MPS's projectional editing, notebook
environments with structured/text dual views) treats exactly this problem as one of the hardest
parts of the whole approach, and typically restricts the space of "legal" edits considerably
compared to free-form text editing.

**Recommendation:** Treat reverse projection as the primary research risk of the whole project and
prototype it early and narrowly — e.g., a single target language, a restricted edit grammar
(rename, change a literal, change a comparison operator), before attempting general-purpose
bidirectional sync. Document explicitly which classes of human edits are (and are not) expected
to map back to the graph in v1.

---

### 5. Effects, Concurrency, and I/O Are the Least-Specified Part of the System

**Where it appears:** `docs/concepts/emet-standard-library.md` (`std::io`),
`docs/concepts/architecture-and-compilation.md` ("Dimension 4: Effects & Security")

"Monadic Effect Nodes... isolate side effects, enforce permission capabilities, and prevent
unauthorized execution" is essentially one sentence describing what, in every comparable verified
or effect-typed system (Haskell's IO monad and effect systems generally, Koka, Unison's ability
system, capability security in seL4), is the single most heavily engineered part of the design.
Ordering guarantees, concurrency/interleaving, and non-determinism are where the majority of
real-world correctness bugs and the majority of formal-verification research effort actually live
— arguably more than in the pure/arithmetic domain the current documents focus on.

**Recommendation:** This deserves its own dedicated concept document and likely its own RFC,
specifically addressing: what effect algebra Emet uses, how concurrent/interleaved execution is
modeled and verified, and how the "100% exhaustiveness" and SMT-provability guarantees interact
with non-deterministic external systems (which cannot generally be fully modeled inside a closed
SMT theory).

---

### 6. "100% Path Exhaustiveness" Is in Tension With Open-Ended Real-World Inputs

**Where it appears:** `docs/concepts/architecture-and-compilation.md` (Exhaustiveness Check),
`docs/concepts/logic-graphs-and-control-flow.md` (Predicate Gates)

The weather-sensor example (`Clear`/`Rain`/`Snow`, but not `Fog`/`Hail`) illustrates the value of
exhaustiveness checking well for a *closed* enum of states. But real systems constantly handle
genuinely open-ended input spaces — arbitrary user text, external API responses, malformed
network payloads — where enumerating "every possible state" isn't meaningful. It's unclear from
the current docs whether there is a sanctioned "default/otherwise" catch-all gate, and if so, how
that's reconciled with the stated guarantee of "no unhandled state ever reaches execution."

**Recommendation:** Clarify whether a catch-all/default branch is a first-class, sanctioned Emet
construct, and if so, whether using one is treated as a (permitted) weakening of the
exhaustiveness guarantee, logged/flagged as such, or something else.

---

### 7. Token/Resource Efficiency Claims Are Stated as Facts Without a Measurement Methodology

**Where it appears:** `docs/concepts/tokens-resources-waste.md`,
`docs/concepts/semantic-nodes.md`, `docs/concepts/the-projection-layer.md`,
`docs/concepts/semantic-graph-diffs.md` (each contains a "Token Efficiency" table with figures
like 80–95%, 40–60%, and "100% of retry tokens saved")

The same unsupported percentage ranges recur across four separate documents. There's no
benchmark, prototype measurement, or methodology described (e.g., what workload was measured,
against which baseline model/tool, counted how). Repetition across documents reads as
reinforcement of a claim rather than independent evidence for it, which will likely be one of the
first things a technically literate reviewer flags.

The "100% of retry tokens saved" claim specifically overstates what SMT-based verification can
catch (see also #8 below) — it can eliminate retries caused by constraint/invariant violations,
but not retries caused by incorrect business logic, which is a large fraction of real debugging
loops.

**Recommendation:** Either replace these with hedged estimates ("we expect," "early prototyping
suggests"), or — ideally — build a small benchmark harness (even a toy language subset) and
report measured numbers with the methodology alongside them.

---

### 8. SMT Verification Is Being Implicitly Equated With "Correctness," Which Overstates Its Scope

**Where it appears:** Throughout, but most explicitly in `docs/concepts/architecture-and-compilation.md`
("mathematically proven to be bug-free") and `docs/concepts/tokens-resources-waste.md`

SMT solvers verify whatever can be expressed as a decidable formal property against the theories
they support — bounds, arithmetic invariants, type/shape constraints. They cannot verify that
code does what was *intended* — i.e., whether the specified contract itself matches the actual
requirement. The examples used throughout (non-negative balance, bounded discount percentage,
non-zero denominator) are exactly the class of bug SMT solvers are good at catching, and also a
relatively small fraction of the bugs that cost real engineering time in practice. Most expensive
real-world bugs are logic/requirements mismatches, not constraint violations.

**Recommendation:** State explicitly, probably in the Manifesto or a new "Verification Scope"
document, what classes of correctness Emet's SMT layer can and cannot address. This is not a
weakness to hide — naming the boundary accurately will make the project more credible to a formal
methods audience, not less.

---

### 9. The Manifesto's Rhetorical Framing Undercuts the Project's Own Technical Design

**Where it appears:** `README.md`, `docs/MANIFESTO.md`

Phrases like "human-readable code is a legacy artifact" and "the truth that animates the machine"
are stated as settled conclusions. But the architecture described in
`docs/concepts/the-projection-layer.md` has humans reading and *editing* Python/Rust projections
as the primary day-to-day interaction surface, with edits flowing back into the graph. That's not
what "legacy artifact" usually means — a legacy artifact is something being phased out, not
something still serving as the main interface for the people building the system.

**Recommendation:** The identical technical story — "the graph is canonical, human-readable code
is a synchronized view, not the source of truth" — can be told without the "legacy" framing, and
would likely draw a more substantive technical response rather than a reflexive one focused on the
provocative claim rather than the mechanism.

---

## Positive Notes (What's Working Well and Should Be Preserved)

- **`emet-std` (Axiomatic Standard Library)** is the strongest and most immediately buildable idea
  in the repo. Pre-verified primitives with built-in refinement constraints (non-zero
  denominators, bounds-checked array access) mirror real, proven approaches (SPARK Ada, Dafny's
  verified libraries, Rust's safety model pushed further). This doesn't require solving reverse
  projection or the full graph model to be useful — it could be prototyped as a refinement-type
  library or compiler plugin bolted onto an existing language today.
- **Recurrence Nodes / ranking functions**, while incomplete as described (see #1, #2), are a
  legitimate and well-chosen technique, not an invented one — grounding this explicitly in the
  well-founded recursion / total functional programming literature would strengthen the docs and
  make clear what's being built on.
- **Semantic Graph Diffs** (`docs/concepts/semantic-graph-diffs.md`) correctly identifies a real,
  underserved problem: line-based diffs are semantically blind to renames vs. logic changes. The
  "Impact Radius" framing is a genuinely useful mental model regardless of what happens with the
  rest of the architecture.
- **Contract/Realization separation** (`docs/concepts/candidate-realizations.md`) is a sound
  generalization of interface/implementation separation and multi-target compilation, and is one
  of the more directly implementable pieces.

---

## Alternatives Considered

1. **Scope Emet explicitly to safety-critical / embedded / verification-heavy software**, where
   the termination-only model, exhaustiveness checking, and SMT-backed stdlib are already
   established techniques (see SPARK Ada, seL4, Dafny) rather than positioning it as a
   general-purpose programming paradigm shift. This avoids issues #1 and #2 entirely by making
   them explicit, accepted scope boundaries rather than silent gaps.
2. **Ship the ideas as independent, incrementally adoptable layers** rather than one unified
   all-or-nothing architecture:
   - Phase 1: `emet-std` as a refinement-type library/linter plugin for an existing language.
   - Phase 2: Predicate Gate exhaustiveness checking as a static analysis tool.
   - Phase 3 (longest-horizon, highest-risk): the full graph/projection architecture, only after
     reverse projection and a richer termination model are separately de-risked.

---

## Unresolved Questions

- Does Emet target Turing-complete general-purpose computing, or a decidable/verifiable subset?
  This single decision reshapes nearly every other design choice and should probably be resolved
  and documented before any further RFCs are written.
- What is the fallback behavior when the SMT solver returns "unknown" rather than SAT/UNSAT?
- What is the actual grammar of human edits that reverse projection is expected to support in v1?
- Is there a sanctioned "default" branch for predicate gates handling open-ended input spaces, and
  if so, how is it reconciled with the "100% exhaustiveness" claim?
- What effect/concurrency model underlies `std::io`, and how does it interact with the
  SMT-provability guarantees claimed elsewhere?
- Are the token-efficiency figures based on any prototype or benchmark, or are they purely
  illustrative at this stage?

---

## Suggested Next Steps

1. Resolve the scope question (Unresolved Question #1) first — it determines how every other item
   here should be framed.
2. Split items #1, #3, #4, and #5 into their own individual RFCs, since each represents a
   substantial, independent design problem deserving focused discussion.
3. Add a short "Verification Scope and Limitations" document alongside the existing Manifesto,
   stating plainly what the SMT layer can and cannot guarantee (addresses #7 and #8).
4. Consider a lighter-weight v0 (the `emet-std` refinement-type library alone) as a way to get real
   usage data and a genuine efficiency benchmark before committing further to the full graph
   architecture.
