# Project Emet — Closed

> **So you have chosen AI...**
> **This is the Truth that animates the machine.**

**This project is closed.** See [docs/CLOSURE.md](docs/CLOSURE.md) for why, including a
retrospective on the reasoning that led to closure.

In short: Emet started from a solution — an AI-Native Intermediate Representation — rather than
from a clearly defined problem, and the documentation went on to overclaim what that
architecture could actually deliver (general correctness, seamless bidirectional projection,
universal termination guarantees) well beyond what was established. `docs/CLOSURE.md` names this
pattern explicitly and is the canonical record of what happened and why.

This repository is kept public and unarchived-in-content (though it may be archived at the
platform level) as a research record: the ideas, the critique, and the lessons learned are real
and may inform future work, even though this specific project is not continuing.

---

## What This Repository Contains (Historical)

### An AI-Native Intermediate Representation

Project Emet was a proposed new approach to programming language design where human-readable code
was treated as a generated projection rather than the primary representation, with an AI-Native
Intermediate Representation (AIR) proposed as the "source of truth."

**Original core philosophy** (kept for historical accuracy, not as a current claim):

- **AIR as Foundation**: the AI-Native Intermediate Representation as the primary form of code
- **Human Code as Projection**: traditional source code generated from AIR when needed
- **Verification First**: formal correctness intended to be intrinsic, not an afterthought
- **Machine Intelligence**: optimized for AI reasoning, manipulation, and verification

**Original framing of what made this different:**

```
Traditional: Human Code → IR → Machine Code
Emet:        AIR ⇄ AI Systems → (Projections | Machine Code)
```

An external critique of this framing — including why several of these claims outran what was
actually established (general termination proving, robust bidirectional projection, and full
formal effects verification remain open problems, not solved engineering) — is preserved in the
project history and referenced in `docs/CHARTER.md`.

### The Mid-Course Correction (Also Superseded)

Partway through, the project attempted to correct course by reframing the objective around a
narrower, measurable claim — reducing token usage in AI-assisted software engineering without
regressing time or quality — documented in `docs/CHARTER.md`. This was a genuine improvement in
rigor, but the project was ultimately closed rather than continued under that reframing, since the
original solution-first sequencing was judged to be the deeper issue. `docs/CHARTER.md`'s method
(the metric/guardrail structure, the staged-gate model) is expected to carry forward into future,
problem-first work, independent of Emet's specific architecture.

---

## Documentation (Historical Record)

- **[Manifesto](docs/MANIFESTO.md)**: the original vision and principles — kept as a historical
  record of the project's original framing, not as a current goal statement.
- **[Charter](docs/CHARTER.md)**: the mid-course reframing toward a narrower, measurable
  objective, and the staged-gate decision process developed alongside it.
- **[Closure](docs/CLOSURE.md)**: why the project was closed, and the lessons carried forward.
- **[Core Concepts](docs/concepts)**: technical concepts and architecture, kept as a library of
  candidate techniques and prior art, not as validated conclusions.
- **[RFCs](rfcs/)**: technical proposals and specifications from the project's active period.
- **[Prerequisites](docs/PREREQUISITES.md)**: background reading on the foundational concepts
  (type systems, SMT solving, dependent types, content-addressed code, effect systems) that inform
  both the original design and the critique of it. Still useful independent of this project's
  status.
- **[Original README](docs/archive/README.original.md)**: the project's README exactly as it
  stood before closure, preserved in full for anyone who wants the primary source rather than the
  historical summary above.

## Why "Emet"?

Emet (אמת) means "truth" in Hebrew. The name was originally chosen to reflect a claim about
computational truth living in the AIR rather than in human-readable source. Read in hindsight, it
also fits the project's closure: naming the actual reason for closing plainly, rather than quietly
letting the repository go stale, is its own kind of truth-telling.

## Status

**Closed.** See [docs/CLOSURE.md](docs/CLOSURE.md).

## License

This project is licensed under the terms described in the [LICENSE](LICENSE) file.
