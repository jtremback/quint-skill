# Verification Guide

## Core Concepts

### Witnesses vs Invariants

| Aspect | Witnesses | Invariants |
|--------|-----------|------------|
| Purpose | Reachability / liveness | Safety |
| Formulation | Negated goals: `not(reached_state)` | Properties: `always_holds` |
| Expected result | **VIOLATED** = GOOD | **SATISFIED** = GOOD |
| Bad result | SATISFIED = concern | VIOLATED = BUG |

### Commands

**Prefix every `quint` command — and only `quint` commands — with `timeout
30`.** (Never wrap project build/test commands like `go build` or `npm test`
in this short timeout; they take minutes and it will kill them.) A well-formed
spec typechecks and simulates in well under a second, so 30s only ever fires on
a pathological spec that would otherwise hang (see `apalache::generate` in
builtins.md) — killing it so you get an error to react to instead of a dead
turn. The examples below omit the `timeout` prefix for brevity — always add it
in practice.

```bash
# Witness check (expect VIOLATION):
quint run spec.qnt --main=Module --invariant=witnessName --max-steps=100 --max-samples=100
# Invariant check (expect NO violation):
quint run spec.qnt --main=Module --invariant=invariantName --max-steps=50 --max-samples=100
# Reproduce a violation with seed:
quint run spec.qnt --main=Module --invariant=name --seed=0x1234 --verbosity=3
# Run deterministic tests:
quint test spec_test.qnt --main=TestModule --match="testName"
```

## Witnesses

A witness is a negated reachability goal: `val canX: bool = not(scenario)`. Put
the one or two witnesses you actually care about into the spec file alongside
the invariants — no separate `_witnesses.qnt` file. Run with
`quint run --invariant=canX`.

### Interpreting witness results

| Output | Result | Meaning | Action |
|--------|--------|---------|--------|
| "An example execution" | VIOLATED | Scenario reachable | Good — record the success |
| "No trace found" / "No violation found" | SATISFIED | Scenario NOT reached | If you expected it reachable, raise `--max-steps`/`--max-samples` selectively or check whether the witness is too strong. If you didn't expect reachability, ignore. |

Don't escalate budgets blindly — only when you have a specific reason to think a
particular scenario should have been hit. Common reasons a witness stays
satisfied when it shouldn't: init conditions block progress, an action's guards
are unsatisfiable, the witness predicate is stronger than intended.

## Running Invariants

### Basic Run

```bash
quint run {file} --main={module} --invariant={name} --max-steps=50 --max-samples=100```

### When Invariant is Violated (BUG)

1. **Capture seed** from output
2. **Get detailed trace:**
   ```bash
   quint run {file} --main={module} --invariant={name} --seed={seed} --verbosity=3```
3. **Analyze trace:**
   - Find the step where invariant became false
   - Identify the action that caused the transition
   - Extract relevant state variables at that step
4. **Determine root cause:**
   - Bug in spec logic? → Fix the spec
   - Invariant too strong? → Weaken the invariant
5. **Record the reproduction command** (the seed) in a THOUGHT for your own reference

## Running Tests

```bash
quint test {test_file} --main={module} --match="testName"
quint test {test_file} --main={module} --match=".*"  # all tests
```

### Debugging Failed Tests

**Critical**: Quint reports errors at the test chain START (`init`), NOT where `.expect()` failed.

1. Run with `--verbosity=3`
2. Count frames — map to test code to find actual failure point
3. Check state values in last frame vs expected
4. Classify: spec bug or test bug

## Before moving on to implement the code

Briefly reflect in a THOUGHT on what verification gave you, then act on one of:

- **Invariants held under sampling** → the design is validated for the cases the simulator explored; proceed to write the code change.
- **An invariant violated** → the model found a flaw. Read the trace (`--verbosity=3`), capture the seed for reproduction, then revise the design (or the spec, if it was the spec that was wrong) and re-run.
- **A targeted witness you expected to fire didn't** → either raise `--max-steps`/`--max-samples` once (if you have a specific reason to think the scenario should be reachable), weaken the witness, or accept that the scenario isn't reachable in your design and update your mental model accordingly.

The spec is a scratchpad — the goal is enough confidence in the design to write the code, not to certify the spec itself.
