

---

# Logic Graphs & Control Flow

## Overview

In traditional programming languages, control flow is managed by linear text structures: `if/else` branching statements and `for/while` loops.

While these constructs are intuitive for humans, they introduce massive fragility into AI-generated code. A missing boundary check in an `if` statement causes runtime crashes, while a bad loop condition produces catastrophic **infinite loops** that burn CPU resources and hang execution.

In **Project Emet**, linear control flow is replaced by **Logic Graphs**. Instead of imperative text branches and loops, execution transitions are governed by **Predicate Gates** and **Recurrence Nodes** carrying embedded mathematical proofs of termination.

---

## 1. Predicate Gates (Replacing `if/else` Conditions)

### What is a Predicate Gate?

In traditional code, an `if` statement is a binary junction in text (`if condition: do_A() else: do_B()`).

A **Predicate Gate** is a structured node in the AIR graph that acts as a **mathematical filter**. Data cannot pass through the gate into down-stream operations unless its incoming **Semantic Node** satisfies the gate's explicit predicate function.

```
                   [ Input Semantic Node: User Balance ]
                                     │
                                     v
                  +-------------------------------------+
                  |   PREDICATE GATE: Balance >= Cost   |
                  +-------------------------------------+
                                  /       \
                     Passed (True) /         \ Failed (False)
                                 /           \
                                v             v
                    [ Complete Order ]   [ Reject Order ]

```

### The Difference: Text `if` vs. Predicate Gate

* **Traditional `if` Statement:**
```python
# Fragile: If the developer forgets to check for negative input, 
# an invalid state silently passes through.
if user_balance > item_cost:
    process_payment()

```


* **Emet Predicate Gate:**
* The gate explicitly consumes a Semantic Node (e.g., `user_balance` with constraint $\text{balance} \ge 0.00$).
* The gate demands **100% Path Exhaustiveness** (verified in Stage 1 of compilation). If any mathematical outcome of the predicate function is unhandled, the graph fails to compile.
* The SMT Solver (Stage 2) proves that the output states of *both* branches preserve all system invariants before code is emitted.



---

## 2. Recurrence Nodes (Replacing `for/while` Loops)

### What is a Recurrence Node?

A traditional `while` loop repeatedly executes a block of text until a boolean condition turns false. If the condition never turns false, the program enters an **infinite loop**.

A **Recurrence Node** models repetition as a mathematical recurrence relation over a graph state. Instead of arbitrarily jumping back to earlier code lines, it defines:

1. **Initial State ($S_0$):** The starting values fed into the iteration.
2. **Step Function ($f(S_k \rightarrow S_{k+1})$):** How the state transforms during each pass.
3. **Termination Ranking Function ($R(S)$):** A mathematical proof that guarantees the loop *must* finish.

---

## 3. How Recurrence Nodes Prove Termination (No Infinite Loops)

To prevent infinite loops, every Recurrence Node must carry a **Ranking Function** ($R$) verified by the **Emet-Validator** (SMT Solver).

### What is a Ranking Function?

A Ranking Function maps the loop's current state to a non-negative integer ($R: S \rightarrow \mathbb{N}$). The validator enforces two strict rules during Stage 2 compilation:

1. **Strict Monotonic Decrease:** Every single iteration of the loop *must* strictly decrease the rank value:

$$R(S_{k+1}) < R(S_k)$$


2. **Bounded Bottom:** The rank value can never drop below zero ($R(S) \ge 0$).

Because a non-negative integer cannot decrease infinitely, the SMT solver mathematically **proves that the loop MUST terminate in at most $R(S_0)$ steps**. If an AI agent attempts to create a loop where the rank function does not strictly decrease, the Emet-Validator rejects the code immediately before compilation.

---

## 4. Concrete Example: Processing an Order Queue

### Traditional Approach (Python `while` loop)

```python
# FRAGILE: If process_item() raises an exception or fails to pop 
# the item under certain conditions, this loop runs forever.
while len(order_queue) > 0:
    item = order_queue[0]
    if item.is_valid:
        process(item)
        order_queue.pop(0) # If skipped, INFINITE LOOP!

```

### Emet Approach (Recurrence Node)

```json
{
  "node_id": "node_recurrence_process_orders_091",
  "node_type": "RecurrenceNode",
  "initial_state": {
    "queue_node": "node_order_queue_ref"
  },
  "step_transition": {
    "pop_and_process": "node_process_single_order"
  },
  "ranking_function": {
    "expression": "length(queue_node)",
    "invariant": "length(queue_node) >= 0",
    "step_proof": "length(queue_node_{k+1}) == length(queue_node_k) - 1"
  },
  "termination_target": "node_orders_completed_effect"
}

```

* **How the Validator Proves It:**
1. Initial Rank: $R(S_0) = \text{length}(\text{queue})$.
2. Step Verification: $R(S_{k+1}) = R(S_k) - 1$.
3. The validator checks: Is $R(S_{k+1}) < R(S_k)$ guaranteed for all valid inputs? **YES**.
4. Result: **UNSAT** (Impossible to loop infinitely). Code is proven safe.



---

## 5. Summary: Traditional Control Flow vs. Emet Logic Graphs

| Feature | Traditional Control Flow (`if` / `while`) | Project Emet Logic Graphs |
| --- | --- | --- |
| **Representation** | 1D Text blocks (`if`, `else`, `while`) | Spatial Graph Nodes & Transition Edges |
| **Branching Checks** | Imperative conditions (susceptible to unhandled cases) | Predicate Gates with enforced 100% path exhaustiveness |
| **Loop Guarantee** | Hope/Testing (risk of infinite loops) | Mathematical Ranking Functions ($R(S_{k+1}) < R(S_k)$) |
| **Verification** | Runtime testing / QA | Pre-compilation SMT Proofs (Z3 / CVC5) |
| **AI Modification** | High risk of breaking syntax or loop boundaries | Atomic, structure-validated node mutations |

---

