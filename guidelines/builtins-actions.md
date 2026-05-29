# Quint builtins — actions and debug

## assign

Signature: `action assign: (a, a) => bool`

`assign(n, v)` is true when the state variable named `n` has the value `v`
in the next state of a transition.

This operator is used to define transitions by specifying how the state variables change

Can be written as `n' = v`.

### Examples

```quint
var x: int
action Init = x' = 0
// Next defines all transitions from a number to its successor.
action Next = x' = x + 1
```

## ite

Signature: `action ite: (bool, a, a) => a`

`ite(c, t, e)` is `t` when `c` is true and `e` when `c` is false.

Can be written as `if (c) t else e`.

Can be used with actions.

### Examples

```quint
pure val m = if (1 == 2) 3 else 4
assert(m == 4)
```

```quint
var x: int
action a = if (x == 0) x' = 3 else x' = 4
run test = (x' = 0).then(a).then(assert(x == 3))
```

## then

Signature: `action then: (bool, bool) => bool`

`a.then(b)` is true for a step from `s1` to `s2` if there is a state `t` such that

`a` is true for a step from `s1` to `t` and `b` is true for a step from `t` to `s2`.

This is the action composition operator. If `a` evaluates to `false`, then
`a.then(b)` reports an error. If `b` evaluates to `false` after `a`, then
`a.then(b)` returns `false`.

### Examples

```quint
var x: int
run test = (x' = 1).then(x' = 2).then(x' = 3).then(assert(x == 3))
```

## expect

Signature: `action expect: (bool, bool) => bool`

`a.expect(b)` is true for a step from `s1` to `s2` if

 - `a` is true for a step from `s1` to `s2`, and
 - `b` holds true in `s2`.

If `a` evaluates to `false`, evaluation of `a.expect(b)`
fails with an error message. If `b` evaluates to `false`,
evaluation of `a.expect(b)` fails with an error message.

### Examples

```quint
var n: int
run expectConditionOkTest = (n' = 0).then(n' = 3).expect(n == 3)
run expectConditionFailsTest = (n' = 0).then(n' = 3).expect(n == 4)
run expectRunFailsTest = (n' = 0).then(all { n == 2, n' = 3 }).expect(n == 4)
```

## reps

Signature: `action reps: (int, (int) => bool) => bool`

`n.reps(i => A(i))` or `n.reps(A)` the action `A`, `n` times.
The iteration number, starting with 0, is passed as an argument of `A`.
As actions are usually not parameterized by the iteration number,
the most common form looks like: `n.reps(i => A)`.

The semantics of this operator is as follows:

- When `n <= 0`, this operator does not change the state.
- When `n = 1`, `n.reps(A)` is equivalent to `A(0)`.
- When `n > 1`, `n.reps(A)`, is equivalent to
  `A(0).then((n - 1).reps(i => A(1 + i)))`.

### Examples

```quint
var x: int
run test = (x' = 0).then(3.reps(i => x' = x + 1)).then(assert(x == 3))
```

## fail

Signature: `action fail: (bool) => bool`

`a.fail()` evaluates to `true` if and only if action `a` evaluates to `false`.

This operator is good for writing tests that expect an action to fail.

## assert

Signature: `action assert: (bool) => bool`

`assert(p)` is an action that is true when `p` is true.

It does not change the state.

### Examples

```quint
var x: int
run test = (x' = 0).then(3.reps(x' = x + 1)).then(assert(x == 3))
```

```quint
var x: int
action Init = x' = 0
action Next = x' = x + 1

run test = Init.then(all { Next, assert(x > 0) })
```

## q::debug

Signature: `pure def q::debug: (str, a) => a`

`q::debug(msg, value)` prints the given message and value to the console,
separated by a space.

It also returns the given value unchanged,
so that it can be used directly within expressions.

When called with a single argument as `q::debug(expr)`, it prints the
expression itself as the message, followed by its value.

### Examples

```quint
var x: int
>>> (x' = 0).then(3.reps(i => x' = q::debug("new x:", x + 1)))
> new x: 1
> new x: 2
> new x: 3
true
```

```quint
var x: int
>>> (x' = 0).then(x' = q::debug(x + 1))
> x + 1: 1
true
```

## apalache::generate — DO NOT USE

`apalache::generate` is a symbolic-only operator for the Apalache model checker
(`quint verify`). It **hangs the randomized simulator** used by `quint run`
(verified: a spec using it never terminates, vs. ~0.4s for the same spec with
`oneOf`). This workflow uses `quint run`, so never use `apalache::generate`.

To pick a value nondeterministically, use `nondet x = S.oneOf()` over a
concrete, bounded set instead.
