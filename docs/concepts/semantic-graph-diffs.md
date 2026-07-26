

---


# Semantic Graph Diffs & Impact Analysis

## Overview

Traditional version control systems (like Git) treat software changes as **1D line-based text edits** (`+ line 42, - line 42`). Line diffs cannot distinguish between a cosmetic formatting change, a variable rename, or a breaking logical modification.

In **Project Emet**, code changes are managed as **Semantic Graph Diffs**. 

Because the source of truth is a multi-dimensional AIR graph, a change is recorded as a structural transformation over graph nodes and edges. The Projection Layer evaluates edits by calculating their exact **Impact Radius** across data flow, control logic, and system invariants.

---

## Line Diffs vs. Semantic Graph Diffs


TRADITIONAL LINE DIFF (Text-Based)

* const max_users = 100;

* const max_users = 200;

Problem: Git has no idea what functions or security invariants
are affected by changing "100" to "200".

PROJECT EMET SEMANTIC GRAPH DIFF
Mutation Payload:
Node ID: node_max_users_881
Property: constraint_value
Old State: <= 100
New State: <= 200

```
Calculated Impact Radius:
├── Modified Nodes: 1 (node_max_users_881)
├── Dependent Gates Evaluated: 4 (All verified SAT)
└── Broken System Invariants: 0

```

---

## The Three Components of a Semantic Graph Diff

### 1. Atomic Node Deltas
Instead of replacing entire text files or functions, a semantic diff represents an atomic mutation payload targeting specific node UUIDs (`node_id`):
* `NodeAdded`: Introduces a new Semantic Node, Predicate Gate, or Effect Node.
* `NodeMutated`: Updates an existing node's value, type constraint, or formula.
* `EdgeRerouted`: Changes a dependency link between two nodes.
* `NodeRemoved`: Safely detaches a node (rejected by Stage 1 if orphans are created).

### 2. Calculated Impact Radius
When an AI agent or developer submits a semantic delta, the compiler traverses the graph's dependency edges to compute the exact **Impact Radius**:
* **Direct Dependents:** Nodes that consume data directly from the modified node.
* **Indirect Dependents:** Downstream logic gates and effects that rely on direct dependents.
* **Invariant Scope:** Specific mathematical rules ($x \ge 0$) that must be re-audited by the SMT solver.

### 3. Pre-Execution Validation Status
Unlike Git, which allows broken code to be committed as long as syntax is valid, a Semantic Graph Diff includes an embedded **Validation Certificate**:
* **Preserved:** The graph mutation maintains all structural and SMT invariant rules.
* **Violated:** The mutation breaks an invariant downstream. The diff is rejected before it can be merged into the primary graph trunk.

---

## Key Benefits

| Feature | Line-Based Diffs (Git) | Semantic Graph Diffs (Emet) |
| :--- | :--- | :--- |
| **Granularity** | Text lines & line numbers | Atomic Node UUIDs & Properties |
| **Noise Sensitivity** | Formatting, whitespace, and renames create noisy diffs | Renames and formatting create **zero** graph diff noise |
| **Causality Tracking** | Unclear which line edits caused a bug | Exact dependency edges mapped from source node to effect |
| **Conflict Resolution** | Merge conflicts occur when two lines overlap | Conflicts occur only when two mutations contradict node invariants |
| **Token Efficiency** | AI must re-read full text diffs | AI queries exact, compact node delta payloads |

---

