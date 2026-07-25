
---

# Emet Standard Library (`emet-std`)

## Overview

In traditional software development, standard libraries provide basic building blocks—such as string manipulation, primitive math functions, and file I/O—that trust developers to pass valid arguments. If a program divides an integer by zero, accesses an out-of-bounds array index, or overflows a buffer, execution crashes at runtime.

In **Project Emet**, computation must be mathematically proven safe before code can compile.

The **Emet Standard Library (`emet-std`)** is the foundational repository of **Axiomatic Semantic Nodes**. Rather than consisting of unverified code routines, `emet-std` provides pre-verified building blocks for math, logic, memory, and I/O. Every primitive in `emet-std` carries intrinsic mathematical constraints (**Refinement Types**) and formal correctness proofs that integrate natively into the **Emet-Validator (SMT Solver)** pass.

---

## 1. Core Architectural Modules

The standard library is structured into four primary axiomatic domains:

```
                                +-----------------------------------+
                                |     EMET STANDARD LIBRARY         |
                                |           (emet-std)              |
                                +-----------------------------------+
                                                  |
     +----------------------+---------------------+----------------------+----------------------+
     |                      |                                            |                      |
     v                      v                                            v                      v
+------------------+  +-------------------+                    +-------------------+  +-------------------+
|   std::math      |  |    std::logic     |                    |    std::memory    |  |     std::io       |
|  Axiomatic Math  |  | Predicates & Gates|                    | Slices & Collections| |  Monadic Effects  |
+------------------+  +-------------------+                    +-------------------+  +-------------------+

```

### A. `std::math` (Verified Arithmetic)

Provides axiomatic operations for integer, real, and floating-point math. Traditional unsafe arithmetic operations are replaced by nodes carrying intrinsic non-zero, overflow, and underflow constraints:

* `std::math::div`: Division node requiring a non-zero denominator constraint ($\text{denominator} \ne 0$).
* `std::math::add_bounded`: Addition node carrying explicit bit-vector bounds to prevent hardware overflow.
* `std::math::sqrt`: Square root node enforcing non-negative inputs ($\text{value} \ge 0$).

### B. `std::logic` (Predicate & Control Primitives)

Provides the axiomatic building blocks for **Predicate Gates** and boolean decision trees:

* `std::logic::gate`: Evaluates boolean predicates and enforces 100% path exhaustiveness.
* `std::logic::recurrence`: Base primitive for verified loops carrying ranking function interfaces ($R(S_{k+1}) < R(S_k)$).

### C. `std::memory` (Safe Collections & Indexing)

Replaces dangerous pointer arithmetic and unchecked array access with refined bounds-checked primitives:

* `std::memory::slice_at`: Array access node requiring index refinement constraints ($0 \le \text{index} < \text{length}$).
* `std::memory::alloc_bounded`: Buffer allocation node enforcing maximum size invariants to eliminate heap overflow risks.

### D. `std::io` (Monadic Side-Effect Capabilities)

External operations (file access, network requests, standard output) are inherently non-deterministic. `std::io` wraps external interaction in **Monadic Effect Nodes** that isolate side effects, enforce permission capabilities, and prevent unauthorized execution.

---

## 2. Axiomatic Nodes vs. Traditional Primitives

```
Traditional Code (Python / C++)               emet-std Axiomatic Node
+-------------------------------+             +-----------------------------------------+
|  result = a / b               |             | node_type: "std::math::div"             |
|                               |             | inputs: [a, b]                          |
|  Danger:                      |             | constraints: {                          |
|  If b == 0, program crashes   |             |   "b": "b != 0"  <-- Pre-Verified Proof |
|  at runtime!                  |             | }                                       |
+-------------------------------+             +-----------------------------------------+

```

| Dimension | Traditional Standard Library | Emet Standard Library (`emet-std`) |
| --- | --- | --- |
| **Safety Enforcement** | Developer-written checks / Runtime exceptions | Pre-verified mathematical refinement constraints |
| **Compiler Integration** | Passive type-checking only | Direct input into SMT Solver (Z3 / CVC5) |
| **Division Behavior** | Crashes or returns `NaN`/`Infinity` | Compilation fails unless $b \ne 0$ is proven |
| **Array Access** | Potential buffer overflow or runtime panic | Compile-time proof that $0 \le \text{index} < \text{length}$ |
| **AI Generation** | AI must repeatedly generate boilerplate safety checks | AI simply references pre-proven `emet-std` UUIDs |

---

## 3. Concrete Example: `std::math::div` in Action

### Traditional Primitive (Unsafe)

In C++ or Rust, writing `a / b` generates assembly directly. If $b = 0$, the CPU raises a floating-point exception or panics at runtime.

### The `emet-std` Axiomatic Node

In Project Emet, referencing `std::math::div` inserts a fully specified Semantic Node into the AIR graph:

```json
{
  "node_id": "node_std_math_div_001",
  "primitive": "std::math::div",
  "inputs": {
    "numerator": "node_total_revenue",
    "denominator": "node_active_users"
  },
  "axiomatic_constraints": {
    "denominator_safety": "node_active_users != 0"
  },
  "outputs": {
    "result": "node_average_revenue_per_user"
  }
}

```

### How the SMT Solver Uses `emet-std`

1. **The Proof Search:** When Stage 2 (Emet-Validator) encounters `std::math::div`, it inspects the `denominator_safety` rule.
2. **Path Audit:** The SMT solver tests: *"Can `node_active_users` evaluate to `0` under any possible graph state?"*
3. **Automatic Resolution:**
* If a **Predicate Gate** exists upstream checking `active_users > 0`, the solver returns **UNSAT** (Safe). The code compiles cleanly.
* If no check exists, the solver returns **SAT** (Crash Possible) and halts compilation:
> *"Truth Violation: Primitive `std::math::div` requires non-zero denominator. Node `node_active_users` is unconstrained."*





---

## 4. Key Benefits of `emet-std` for AI Code Generation

1. **Elimination of Boilerplate Code:** AI models do not need to re-generate safety code, try-catch blocks, or manual bounds checking. Referencing a pre-verified `emet-std` node automatically embeds the necessary mathematical proofs.
2. **Reduced Token Footprint:** AI agents emit short, atomic node references (e.g., `"primitive": "std::math::add_bounded"`) rather than writing verbose, multi-line safety routines in text.
3. **Composable Safety:** Because all `emet-std` nodes adhere to strict mathematical interfaces, complex software systems built from `emet-std` primitives inherit safety properties transitively across the entire AIR graph.

---

## Summary

The **Emet Standard Library (`emet-std`)** provides the verified bedrock for Project Emet. By replacing raw, unchecked programming language operations with **axiomatic nodes carrying built-in mathematical constraints**, `emet-std` ensures that every basic calculation, memory access, and I/O operation is proven safe before machine code generation ever begins.

---
