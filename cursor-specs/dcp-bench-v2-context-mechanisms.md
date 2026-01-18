# DCP-Bench v2: Context-Dependent Learning Mechanisms

## Overview

This document defines foundational context mechanisms that ensure tasks cannot be solved correctly without accumulated experience across phases.

These mechanisms model real-world software development, where correctness depends not only on code, but on remembered constraints, prior explanations, and historical decisions.

**The following mechanisms are mandatory primitives for DCP-Bench v2.**

---

## 1. Mechanism 1: One-Time Explanation → Long-Term Expectation

### Motivation (Real-World Analogy)

In real software projects:

- Architectural constraints are often explained once, during onboarding or early design discussions
- Later tasks, bug reports, or code reviews do not restate these explanations
- Developers are expected to remember the reasoning behind constraints and act accordingly

Failure to remember prior explanations leads to:

- "Reasonable" changes that violate hidden assumptions
- Reintroduction of previously eliminated bugs
- Repeated mistakes by inexperienced developers

**This mechanism models accumulated professional experience.**

### Core Idea

- A rule is **explicitly explained only once**
- In later phases, the rule is **enforced without explanation**
- The agent must retain and internalize the original explanation
- The code alone is not sufficient to recover the reasoning

### Formal Definition

In an early phase P₀, a rule R is introduced with:

- a detailed explanation
- rationale ("why this rule exists")
- consequences of violation

In later phases P₁…Pₙ:

- the same rule R is enforced
- feedback references only the rule identifier
- no explanation is repeated

The agent is expected to:

- remember the original explanation
- apply it correctly in new contexts
- avoid violating the rule even when the current task encourages it

### Required Agent Capability

The agent must retain **semantic memory**, not just code state:

- what the rule means
- why it exists
- what kinds of changes may violate it

### Benchmark-Level Guarantees

This mechanism ensures that:

- Restarting "from scratch" at each phase is suboptimal
- Agents without long-term context retention degrade over phases
- Experience accumulation directly affects performance

### Example (Normalize Dictionary Task)

**Phase 1 — Explicit Explanation**

Feedback returned on failure:

```json
{
  "rule_id": "no_input_mutation",
  "message": "The input dictionary must not be modified. The same input object is reused elsewhere in the system, and mutating it causes downstream logic to break."
}
```

Agent learns:

- Mutating input is forbidden
- The reason is external reuse, not stylistic preference

**Phase 6 — Implicit Expectation**

Feedback returned on failure:

```json
{
  "rule_id": "no_input_mutation"
}
```

No explanation is provided.

- An agent that remembers Phase 1: immediately understands what went wrong
- An agent that does not: sees a vague rule name, must guess, often applies incorrect or partial fixes

### Why Code Alone Is Insufficient

The code may show:

- copying
- deep copying
- defensive checks

But it does not encode **why** mutation is forbidden, nor in which future scenarios it becomes dangerous.

That knowledge exists only in accumulated context.

### Implementation Notes (Automatic Generation)

This mechanism can be generated automatically:

- Assign each rule an `explanation_phase`
- Emit detailed feedback only in that phase
- Suppress explanations in all later phases

```yaml
rule:
  id: no_input_mutation
  explanation_phase: 1
  explanation_required_after: false
```

No manual task writing is required.

---

## 2. Mechanism 2: Multi-Constraint Preservation Under Incremental Change

### Motivation (Real-World Analogy)

In real development work, a common request is:

> "Add this feature, but do not break existing behavior."

This requires developers to:

- remember multiple previously introduced constraints
- understand which parts of the system enforce which constraints
- modify behavior without violating earlier agreements

Inexperienced developers often:

- fix the new problem
- unintentionally break older guarantees

Experienced developers succeed because they carry a **mental model of accumulated constraints**.

### Core Idea

- Later phases require changing behavior while **preserving all previously introduced constraints simultaneously**
- Constraints are not restated
- They are assumed to be already known

### Formal Definition

- Each phase may introduce a new constraint Cᵢ
- All previously introduced constraints {C₁…Cᵢ₋₁} remain active
- A later phase requires a behavioral change that:
  - satisfies the new constraint
  - must not violate any prior constraints

The agent must reason about **constraint interaction**, not isolated fixes.

### Required Agent Capability

The agent must maintain:

- a set of known constraints
- an understanding of which code decisions enforce which constraints
- awareness of trade-offs and fragile areas

### Benchmark-Level Guarantees

This mechanism ensures that:

- "Local fixes" are insufficient
- Regression avoidance becomes a primary skill
- Performance reflects depth of accumulated understanding

### Example (Dependency Sorting Task)

**Phase 1 — Constraint Introduced: Valid Order**

Rule: Dependencies must appear before dependents.

Agent implements topological sorting.

**Phase 2 — Constraint Introduced: Deterministic Output**

Rule: If multiple valid orders exist, the output must be deterministic.

Agent adds deterministic tie-breaking.

**Phase 4 — Constraint Introduced: Performance on Large Inputs**

Rule: The solution must handle large graphs efficiently.

Now the agent must:

- preserve valid ordering
- preserve determinism
- remove or limit expensive operations

A naive agent:

- removes determinism → breaks Phase 2
- changes ordering logic → breaks Phase 1

A competent agent:

- refines where determinism is applied
- preserves all constraints simultaneously

### Why Code Alone Is Insufficient

The final code shows what works now, but not:

- which parts are essential
- which parts were compromises
- which constraints are fragile

Without historical context, an agent cannot safely modify behavior.

### Implementation Notes (Automatic Generation)

This mechanism is trivially automatable:

- Each phase adds a constraint
- The evaluator always checks all prior constraints
- No constraint explanations are repeated

```yaml
phases:
  - introduces: valid_order
  - introduces: deterministic
  - introduces: performance
  - requires: [valid_order, deterministic, performance]
```

---

## 3. Summary: Context-Dependent Learning

These two mechanisms together ensure that:

1. **Correct behavior depends on remembered explanations**
2. **Progress depends on preserving accumulated constraints**
3. **Restarting reasoning at each phase is penalized**
4. **Long-term context becomes a first-class requirement**

They form the foundation upon which more advanced context mechanisms can be layered.
