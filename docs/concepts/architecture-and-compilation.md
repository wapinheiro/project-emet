---

# Architecture & Compilation Model

## Overview

In traditional software development, compilers take flat, human-written text files and translate them sequentially into machine instructions. In **Project Emet**, source code as a text file is a legacy concept.

Project Emet establishes the **AI-Native Intermediate Representation (AIR)** as the primary **Source of Truth**. Program logic exists as a multi-dimensional graph of intent, and traditional human-readable source code (Python, Rust, C++) is merely a temporary, bi-directional **projection**.

---

## The Core Philosophy: Graph of Intent vs. Text File

```
+-----------------------------------------------------------------+
|                        AIR GRAPH ENGINE                         |
|                   (The True Source of Truth)                    |
|                                                                 |
|   [ Semantic Node ] <---> [ Predicate Gate ] <---> [ Effect ]   |
+-----------------------------------------------------------------+
       ^                                             |
       | Bi-Directional                              | One-Way
       | Projection Lens                             v Compilation
+-----------------------+                    +--------------------+
|  Human Projections    |                    |   Emet-Validator   |
| (Python, C++, Rust)   |                    |    (SMT Solver)    |
+-----------------------+                    +--------------------+
                                                     |
                                                     v
                                             +--------------------+
                                             |      LLVM IR       |
                                             +--------------------+
                                                     |
                                                     v
                                             +--------------------+
                                             |    Machine Code    |
                                             +--------------------+

```

### 1. AIR as the Source of Truth

* **Traditional Model:** `Human Code (Text) -> Compiler -> Machine Code`
* **Emet Model:** `AIR Graph <-> AI Systems / Projections -> Machine Code`

In the Emet paradigm, computation is modeled as a verified network of **Semantic Nodes** (data) and **Control/Transition Nodes** (logic). AI agents interact with and mutate this graph directly rather than writing text syntax.

### 2. Human Code as a Projection

Human-readable syntax is rendered on demand through a **Projection Lens**. If a developer edits logic in a Python projection, a bi-directional transformation updates the underlying AIR graph. If the edit violates an intrinsic mathematical constraint in the AIR, the projection lens rejects the edit immediately.

---

## The Emet Compilation Pipeline

Unlike traditional compilers that perform lexing, parsing, and syntax-tree generation on raw text, the Emet compiler operates on a pre-structured semantic graph.

### Stage 1: Structural & Topological Analysis

The compiler ingests the AIR JSON/Protobuf graph and performs initial graph traversal:

* **Orphan Detection:** Ensures no isolated effect or transition nodes exist without valid input dependencies.
* **Exhaustiveness Check:** Verifies that all conditional branching paths account for 100% of potential mathematical outcomes.

### Stage 2: The Emet-Validator (SMT Solver Pass)

Before machine instructions are generated, the graph passes through the **Emet-Validator**. Using SMT (Satisfiability Modulo Theories) solvers (such as Z3 or CVC5), the validator proves that the graph preserves computational truth:

* **Refinement Type Verification:** Asserts that node value constraints (e.g., $0 \le x \le 120$) cannot be violated under any execution state.
* **Loop Termination Proofs:** Checks Recurrence Nodes for valid ranking functions to mathematically guarantee termination.
* **Proof Object Audit:** Re-verifies formal mathematical proofs attached to state transitions.

### Stage 3: Lowering to LLVM IR

Once the graph is proven valid by the Emet-Validator, the compiler "lowers" high-level semantic nodes and logic gates into **LLVM Intermediate Representation (LLVM IR)**.

* By targeting LLVM IR, Project Emet inherits battle-tested, world-class optimizations (vectorization, dead-code elimination, memory allocation) and target support across all major architectures (x86_64, ARM, Apple Silicon, WebAssembly).

### Stage 4: Machine Code Emission & JIT Runtime

The LLVM backend compiles the IR into native assembly or machine code. Because AIR tracks state side-effects explicitly, the Emet runtime can also perform Just-In-Time (JIT) re-verification—hot-swapping optimized machine code in real time if execution patterns change.

---

## Key Differences: Traditional Compilers vs. Emet Pipeline

| Pipeline Stage | Traditional Compiler (e.g., GCC, Clang) | Project Emet Compiler Pipeline |
| --- | --- | --- |
| **Input Material** | Unstructured `.c` / `.py` text files | Structured AIR Semantic Graph |
| **Parsing** | Lexical analysis, Tokenization, AST build | N/A (Graph is inherently structured) |
| **Validation** | Syntax errors, basic type checking | SMT Formal Verification, Invariant Proofs |
| **Optimization Goal** | Execution speed, binary size heuristics | Mathematical correctness, AI-native manipulation |
| **Errors Reported** | Missing semicolons, mismatched types | "Truth Violations", unhandled edge states |

---

