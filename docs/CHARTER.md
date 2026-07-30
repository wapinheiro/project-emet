# Project Emet — Charter

- **Status:** Draft v1
- **Purpose:** This document defines the project's core objective, method, and decision process
going forward. It is intended to be the reference point for evaluating whether any given piece of
work is progress, and for deciding when to advance, pause, or abandon a direction.

---

## 1. Relationship to Existing Documentation

This charter **narrows and reframes** the objective stated in `docs/MANIFESTO.md`. The original
framing — an AI-Native Intermediate Representation as a wholesale replacement for human-readable
source code — is not being pursued directly. Instead, this charter defines a narrower, evidence-first
first milestone under the same project, built around a claim that is smaller, more defensible, and
directly measurable.

`README.md`, `docs/MANIFESTO.md`, and the documents in `docs/concepts/` are expected to be revised
to align with this charter. Some concept documents (for example, the standard library and semantic
graph diff ideas) largely survive under the narrower framing; others (particularly the termination
model and the projection layer) will need to be scoped down or explicitly marked as longer-horizon
research rather than near-term deliverables. That revision is tracked as follow-up work, not done
in this document.

---

## 2. Objective

**Reduce token usage in AI-assisted software engineering, without regressing wall-clock time or
code quality relative to current practice.**

This is a constrained optimization, not a single number to minimize:

- **Primary metric (minimize):** tokens consumed per completed task, where "completed" means the
  task reaches a defined done-state (e.g., tests passing), not a self-reported "done" from the
  agent.
- **Guardrail A — Time:** wall-clock time to completion must not regress beyond an agreed
  tolerance relative to baseline (proposed default: within 10%; to be confirmed — see Open
  Questions).
- **Guardrail B — Quality:** code quality must not regress relative to baseline. Quality is
  measured via at least one automated proxy (e.g., pass rate on a held-out test suite not written
  by the system under test) and, at later gates, a blind human reviewer rating. Quality is treated
  as a hard constraint, not an assumption — it must be checked explicitly at every gate, not
  inferred from the absence of complaints.

Token reduction that comes at the cost of more time, or at the cost of quality, does not count as
progress under this charter, even if the token number looks good in isolation.

---

## 3. Baseline

Comparisons are made against **current best-practice AI-assisted software engineering**, not
against unassisted human coding. A specific baseline configuration should be fixed and documented
before running any experiment (e.g., a named agentic coding tool, on an unmodified repository,
default settings), so that future measurements are comparable to each other rather than to a
moving target.

*(To be finalized: the specific baseline tool/configuration — see Open Questions.)*

**Controlled variable — requirements/prompt quality.** Requirements-based programming (writing
clear, complete, unambiguous task specifications) genuinely reduces token usage, but it does so by
improving the human's input, not by changing the system's architecture. It is a discipline applied
by whoever writes the task prompt, not a mechanism the project builds or ships — and so it is not
a candidate lever under Section 4. However, it is a real confound: if prompt/requirement quality
differs across conditions in an experiment, measured differences will reflect prompt quality
rather than the mechanism under test. **The same requirements/prompt text must be used, unchanged,
across every condition for a given task**, so that requirements quality is held constant and does
not leak into the primary metric. (Note: if requirements are ever formalized into structured,
machine-checkable contracts, that becomes an instance of Lever 3 — "contracts as compressed
context" — not a separate lever.)

---

## 4. First Attempt Method: Context Management

The first mechanism to test is **improving context management** — reducing the amount of
re-reading, re-explaining, and re-deriving context that an AI agent needs to do across turns of a
coding task. This is chosen first because context re-reading was identified as one of the largest
token cost buckets in typical agentic coding sessions, larger than code generation itself.

Four candidate levers were identified, not treated as equally weighted:

1. **Persistent semantic index** *(primary candidate for the first prototype)* — a lightweight,
   incrementally-updated model of the codebase (function signatures, types, call graph, contracts)
   that the agent queries instead of re-reading whole files from scratch each turn.
2. **Delta-based context** — after the first turn, supply only what changed (diffs, new errors,
   new test results) rather than re-supplying full surrounding context each time.
3. **Contracts as compressed context** — sufficiently rich type signatures and pre/post-conditions
   let an agent safely reason about a function without re-reading its full body.
4. **Verified-primitive caching** — logic proven correct once (e.g., a bounds check) doesn't need
   to be re-derived or re-explained on subsequent turns that touch nearby code.

Lever 1 is the proposed starting bet. Levers 2–4 are documented backups if lever 1 does not clear
Gate 0 (see below) — the intent is to try one thing at a time, fail cheaply, and move to the next
candidate rather than build all four simultaneously.

---

## 5. Explicit Non-Goals (For Now)

This objective deliberately does **not** require solving the following, which were identified as
the hardest and least-resolved parts of the original Emet architecture:

- General termination proving for arbitrary recursive/iterative programs (Recurrence
  Nodes/ranking functions as a universal requirement)
- Full bidirectional reverse projection (mapping arbitrary human text edits back to a canonical
  graph representation)
- A complete formal effects/concurrency verification system

These may remain valid longer-horizon research directions, but they are out of scope for this
charter's objective and should not be assumed to be secretly required by any work done under it.
Work items should be evaluated against the metric in Section 2, not against these unresolved
architectural ambitions.

---

## 6. Strategy: Staged-Gate Model

Confidence required, and cost of evidence gathered, scale together — cheap-to-reverse decisions
get a quick gut check; decisions that cost other people's time get real measurement. Each gate is a
conditional pass/fail: advancing to the next gate requires clearing the one before it. Failing a
gate is an explicit trigger to pivot the mechanism (try the next candidate lever) or, if repeated
across mechanisms, to reconsider the objective itself (see Section 7).

### Gate 0 — "This could work" (personal conviction)
- **Cost tolerance:** hours to ~2 days.
- **Evidence required:** a hand-built, minimal version of the chosen mechanism (lever), tried on
  one real task with a known-good expected outcome. Subjective assessment is acceptable here.
- **Pivot/abandon trigger:** the mechanism doesn't noticeably reduce the amount of re-explaining or
  re-reading needed → drop it, try the next candidate lever from Section 4.

### Gate 1 — "Good enough to invite collaborators"
- **Cost tolerance:** ~1–2 weeks.
- **Evidence required:** a small, automatically-logged before/after comparison (tokens, rough
  wall-clock) on a handful of real tasks, runnable by a second person without your direct help. No
  formal control group required yet.
- **Non-negotiable guardrail check:** output must still pass whatever tests it passed before —
  checked explicitly, not assumed from the absence of visible breakage.
- **Pivot/abandon trigger:** no measurable token reduction, or any regression in the guardrail
  check → return to Gate 0 with the next candidate lever.

### Gate 2 — "Good enough for external/academic scrutiny" (e.g., a professor, department demo)
- **Cost tolerance:** a few weeks.
- **Evidence required:** a scoped-down controlled comparison — baseline vs. best mechanism, a
  modest task set (~8–10 tasks), a real (if small) held-out test suite for the quality guardrail.
  Methodology must be able to withstand being questioned by a skeptical outside audience.
- **Pivot/abandon trigger:** results don't hold up under a real methodological question, or the
  measured gain doesn't clear both guardrails → revisit mechanism design before proceeding.

### Gate 3 — "Good enough for early release / testers"
- **Cost tolerance:** real investment now justified, since real users' time is at stake.
- **Evidence required:** ship the minimal mechanism to real users on real projects, with automatic
  token/time logging on genuine usage. Quality is tracked as an active listening post (explicitly
  solicited feedback on regressions, not just passive absence of complaints), not inferred solely
  from adoption.
- **Pivot/abandon trigger:** users report quality regressions even if adoption/enthusiasm is
  positive → the guardrail takes precedence over adoption as a signal.

---

## 7. Reconsideration Trigger

If Gate 1 is failed by two or more different candidate levers from Section 4, that is a signal to
pause and reconsider whether context management is the right first mechanism at all — not just to
keep trying further levers indefinitely. At that point, revisit this charter's Section 4 (and, if
needed, Section 2) explicitly, rather than continuing to iterate without updating the plan.

---

## 8. Open Questions

- What is the exact tolerance for the time guardrail (proposed default: 10%)?
- Which quality proxy (or combination) is authoritative: automated test pass rate, blind human
  review, static complexity measures, or some combination? What weighting?
- What is the specific baseline tool/configuration to compare against?
- What size and composition of task set is used at each gate (toy snippets vs. real, moderately
  complex codebases)? Given that context-reading costs barely show up on trivial tasks, task
  realism matters more than task count.
- Who makes the pivot/abandon call at each gate, and is it a single-person decision or does it
  require external input (e.g., a collaborator's agreement) once collaborators are involved?

---

## 9. Next Steps

1. Resolve the Open Questions above (or explicitly mark them as "decide at the relevant gate").
2. Revise `docs/MANIFESTO.md` and `README.md` to reflect this narrower framing.
3. Review each file in `docs/concepts/` against this charter — mark each as (a) still accurate as
   written, (b) needs revision to scope down claims, or (c) reclassify as longer-horizon research,
   out of scope for the current milestone.
4. Begin Gate 0 work on the persistent semantic index (Section 4, lever 1).
