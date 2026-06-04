# Quint-Assisted Implementation

Your deliverable is a **code change** — and these tasks are overwhelmingly new
features and enhancements, occasionally bug fixes — not a spec. Quint is a
reasoning tool with two payoffs, both feeding the code you write:
1. **Planning (primary).** When you're building behavior that doesn't exist
   yet, modeling its states, transitions, and invariants forces you to pin
   down the design — what states are reachable, what must always hold — before
   you commit it to code.
2. **Checking.** Running the model validates your intended design against the
   property and surfaces flawed interleavings; for the rare genuine bug, it
   reproduces the bad trace directly.

You benefit from (1) even when (2) finds nothing. The `.qnt` file is a
scratchpad — it is never the thing you submit.

## What to model — and how much

This is **not** an all-or-nothing choice between "fully formalize the task" and
"skip Quint." Model the part(s) of the design that are subtle, to the extent it
helps — and no more. So the question isn't *whether* to use Quint, it's *which
slice* of the design is worth pinning down before you code it.

Look for the slice with enough **state and branching** that you could plausibly
miss an edge case reasoning in your head:

- **Concurrency & ordering** — interleaving of goroutines/threads/async tasks,
  channels, locks; properties over schedules such as deadlock-freedom, mutual
  exclusion, no lost/duplicated updates, exactly-once, progress. *If you're
  working out "who calls Done() when" or "what if these callbacks race" in
  prose, model it instead.*
- **Multi-step / stateful operations** — staged lifecycles, pipelines,
  pagination, streaming or incremental delivery, rollback.
- **Interacting options / combinatorial logic** — flags, modes, or config whose
  combinations interact: dependencies, conditional requirements, merge/override
  rules, precedence.
- **Data-structure invariants** — recursive or nested schemas, trees, graphs,
  cursors, caches; operations that must preserve well-formedness or
  determinism.
- **Explicit invariants** — any property that should hold across all reachable
  states or inputs, where the edge cases are easy to overlook.

Model the slice that fits one of these; leave the mechanical surface (plain API
additions, getters, formatting) to ordinary coding. A partial spec — one state
machine, one invariant, one race — is fine. Quint only fails to help when
nothing in the task has subtle state at all (a rename, a signature change, a
pure string transform).

## The pipeline

1. **Name the property.** For the slice you're modeling, state the safety
   invariant(s) it must preserve and the reachability/liveness goal(s) it
   should achieve. Locate the relevant state in the existing code: search for
   what you'll be extending (`state`, `phase`, `round`, `step`, `transition`,
   `match`/`switch` on a status, lock/channel usage).
2. **Model that slice.** Write a spec of the design you plan to implement —
   just the new states, transitions, and operations that make the slice subtle;
   don't transcribe the codebase or model the mechanical parts.
3. **Typecheck** and fix until it passes.
4. **Run the invariants and witnesses.** A violation means your intended design
   is flawed (or, for a bugfix, you reproduced the bug) — read the trace and
   revise the design. All-passing results validate the design you're about to
   implement.
5. **Implement it in code.** Write the feature/fix following the design the
   model validated; the spec tells you which transitions and guards the code
   needs.
6. *(optional)* Update the spec to match the final implementation and re-run
   the invariant to confirm it still holds.
7. **Submit the code change.**

Keep the spec small and the loop short. Its only job is to help you design and
validate the change.

---

## Detailed mechanics

The steps below expand the pipeline above (spec authoring, typecheck,
witnesses, invariants). They still contain interactive/Claude-Code-specific
assumptions that have not yet been ported to the autonomous bash-only
environment — treat them as reference for now.

---

### Writing the spec

**Before writing any Quint code**, fetch the language-constraints and core
syntax reference — both small, can be cat'd together in one turn:

```bash
cat /opt/quint-skill/guidelines/language-constraints.md /opt/quint-skill/guidelines/quint-syntax.md
```

Then fetch the appropriate template (one of these, not both):
```bash
# Standard systems (state machines, contracts, algorithms — the default):
cat /opt/quint-skill/guidelines/spec-template.md

# Distributed / consensus protocols (BFT, message-passing, multi-node):
cat /opt/quint-skill/guidelines/choreo-template.md
```

**Then read a relevant example spec** to ground your code in real working Quint.
The bundled examples happen to come from finance and distributed-systems
domains — that's just what was on hand. **Read them for the Quint *pattern*
(State type + pure functions + thin actions, or the message-passing structure),
not their subject matter.** Your task is almost certainly an unrelated domain;
match on the *shape of your design*, not the topic.

| If your design looks like… | Read (for the pattern, ignore its domain) |
|---|---|
| A single state record with operations guarded by preconditions — the default for most features | `/opt/quint-skill/guidelines/examples/coin.qnt` (best all-around demo of the core pattern) |
| State kept in maps/collections with invariants over them (per-key rules, conservation, lookups) | `/opt/quint-skill/guidelines/examples/bank.qnt` + `/opt/quint-skill/guidelines/examples/erc20.qnt` |
| A staged lifecycle / multi-phase operation (e.g. start → prepare → commit/abort) | `/opt/quint-skill/guidelines/examples/two_phase_commit.qnt` — read for the phase-transition structure |
| A turn-based or step-by-step state machine | `/opt/quint-skill/guidelines/examples/tictactoe.qnt` |
| Small self-contained logic with tricky constraints | `/opt/quint-skill/guidelines/examples/prisoners.qnt` |
| Multiple actors exchanging messages or taking interleaved steps | `/opt/quint-skill/guidelines/examples/consensus.qnt` (and `/opt/quint-skill/guidelines/examples/two_phase_commit_choreo.qnt` only if the Choreo framework is present) |

If you need a specific operator's signature, read `/opt/quint-skill/guidelines/builtins.md`.

#### Plan the spec

Output a brief plan:

```
Spec Plan: {ModuleName}
  Template:   standard | choreo
  Types:      {list of types to define}
  State vars: {list of state variables with types}
  Actions:    init, {list of domain actions}, step
  Invariants: {list of safety properties to check}
  Witnesses:  {list of reachability goals}
```

Put this plan in the THOUGHT of the same turn in which you write the spec. The agent loop already requires a THOUGHT before every command, so the plan needs no separate turn and no approval step — sketch it, then issue the spec-writing command.

#### Write the spec

Generate the `.qnt` file **outside the project working tree** so it never lands in your submitted diff — the spec is a scratchpad, not a deliverable. Write it under `/tmp`, e.g. `/tmp/quint-specs/{module_name_lowercase}.qnt` (`mkdir -p /tmp/quint-specs` first). Do not create a `specs/` directory inside the repo.

Follow the template from the guideline file you read. Apply patterns exactly — the State Type pattern (encapsulate state, pure functions, thin actions) is mandatory for standard specs.

---

### Typechecking

**Before debugging errors**, fetch the error-handling guide:
```bash
cat /opt/quint-skill/guidelines/error-handling.md
```

`quint` is preinstalled on PATH — invoke it directly (no `npx`). **Wrap `quint` invocations (and only `quint`) in `timeout 60`** so a pathological spec cannot hang the turn (e.g. `timeout 60 quint typecheck …`). Typecheck is sub-second; `quint run` on a real spec takes a few seconds to tens of seconds (the simulator is interpreted), so 60s gives headroom while still catching a genuine hang. If a `quint run` keeps hitting 60s, your spec or sample budget is too big — shrink it rather than raising the timeout. Do **not** apply this timeout to project build/test commands (`go build`, `cargo build`, `npm test`, etc.) — those legitimately take minutes and a short timeout will kill them; run them without a timeout or a generous one.

Run:
```bash
quint typecheck {spec_path}
```

If it fails:
1. Read the error carefully.
2. Check against language constraints and common error patterns from the error-handling guideline.
3. Edit the spec (heredoc or `sed -i`, per the harness's command examples) and re-typecheck.
4. Iterate up to 5 times.
5. If still failing after 5 attempts, stop debugging the spec and proceed to implement the code change with what you've learned — a stuck spec is a scratchpad, not a blocker.

If it passes, proceed.

---

### Witnesses

**Before checking witnesses or invariants**, fetch the verification guide:
```bash
cat /opt/quint-skill/guidelines/verification.md
```

A **witness** is a negated reachability goal written as an invariant: `val canX: bool = not(scenario)`. Under `quint run --invariant=canX`, a **VIOLATED** result is **good** — it means the scenario was reached. A satisfied witness means the scenario wasn't reached in the sampled traces (concerning only if you expected it to be reachable).

Put witnesses for the scenarios you actually care about — usually one or two, not an enumeration — into the spec file alongside the invariants. No separate witness file is needed.

---

### Invariants

Invariants are state predicates that should always hold (`val safe: bool = …`). Run as `quint run --invariant=name`; a **SATISFIED** result is good (no violation in the sampled traces). On a violation, capture the seed for reproduction (`--seed=0x…`).

Multiple invariants can be checked in a single run by combining them with `and`: define `val allSafe: bool = inv1 and inv2 and inv3` in the spec and check that. Useful when you have several cheap invariants and want one command; on failure, fall back to running individuals to attribute.

---

## Error Handling

- **quint not installed**: `quint` is expected to be preinstalled in the environment. The sandbox is offline, so do not attempt `npx`/`npm install`. If `quint` genuinely cannot be run, skip Quint and implement the code change directly.
- **All witnesses satisfied**: spec may be vacuous or over-constrained. Note it in a THOUGHT, revise the spec if warranted, and continue.
- **All invariants violated**: the design is likely wrong. Note it in a THOUGHT, revise the design, and continue (capped, as above).
