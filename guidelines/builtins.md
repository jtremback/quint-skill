# Quint builtins — index

Operator reference is split across four files (each ≤ 10 KB so a single read
returns the whole file, not a head/tail-elided slice). Find your operator
below, then read the file containing it.

## `builtins-core.md` — types, logic, quantifiers
`Nat`, `Int`, `Bool`, `eq` (`==`), `neq` (`!=`), `iff`, `implies`, `not`,
`exists`, `forall`, `in`

## `builtins-sets-maps.md` — sets and maps
`contains`, `union`, `intersect`, `exclude`, `subseteq`, `filter`, `map`,
`fold`, `powerset`, `flatten`, `allLists`, `allListsUpTo`, `getOnlyElement`,
`chooseSome`, `oneOf`, `isFinite`, `size`, `get`, `keys`, `mapBy`, `setToMap`,
`setOfMaps`, `set`, `setBy`, `put`

## `builtins-lists-arith.md` — lists and integer arithmetic
`append`, `concat`, `head`, `tail`, `length`, `nth`, `indices`, `replaceAt`,
`slice`, `range`, `select`, `foldl`, `iadd`, `isub`, `imul`, `idiv`, `imod`,
`ipow`, `ilt`, `igt`, `ilte`, `igte`, `iuminus`, `to`

## `builtins-actions.md` — actions and debug
`assign`, `ite`, `then`, `expect`, `reps`, `fail`, `assert`, `q::debug`,
`apalache::generate` (warning: do not use under `quint run`)

## Removed (verify-only / unusable under `quint run`)
The temporal and fairness operators (`always`, `eventually`, `next`, `orKeep`,
`mustChange`, `enabled`, `weakFair`, `strongFair`) are dropped — they fail to
typecheck or error at runtime under the randomized simulator. Use these
alternatives instead:
- `always(P)` → use `P` as a `--invariant` directly (checked at every state)
- `eventually(P)` → use a witness: `--invariant=not(P)` and look for a violation
- `enabled(A)` → guard inside the action body with `if cond then ... else unchanged_all`
