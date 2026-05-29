# Quint builtins — types, logic, quantifiers


## Nat

Signature: `pure val Nat: Set[int]`

The infinite set of all natural numbers.

Infinite sets cannot be enumerated. Hence some operators
that require iteration may be unsupported.

## Int

Signature: `pure val Int: Set[int]`

The infinite set of all integers.

Infinite sets cannot be enumerated. Hence some operators
that require iteration may be unsupported.

## Bool

Signature: `pure val Bool: Set[bool]`

The set of all booleans

That is, Set(false, true)

## eq

Signature: `pure def eq: (t, t) => bool`

`a.eq(b)` is `true` when `a` and `b` are equal values of the same type.

It can be used in the infix form as `==` or as a named operator `eq`.

## neq

Signature: `pure def neq: (t, t) => bool`

`a.neq(b)` is `true` when `a` and `b` are not equal values of the same type.

It can be used in the infix form as `!=` or as a named operator `neq`.

## iff

Signature: `pure def iff: (bool, bool) => bool`

`p.iff(q)` is `true` when `p` and `q` are equal values of the bool type.

This is the logical equivalence operator.

### Examples

```quint
assert(iff(true, true))
assert(iff(false, false))
assert(not(iff(true, false)))
assert(not(iff(false, true)))
```

## implies

Signature: `pure def implies: (bool, bool) => bool`

`p.implies(q)` is true when `not(p) or q` is true.

This is the material implication operator.

### Examples

```quint
assert(true.implies(true))
assert(false.implies(false))
assert(not(true.implies(false)))
assert(not(false.implies(true)))
```

## not

Signature: `pure def not: (bool) => bool`

`not(p)` is `true` when `p` is `false`.

This is the negation opearator.

## exists

Signature: `pure def exists: (Set[a], (a) => bool) => bool`

`s.exists(p)` is true when there is an element in `s` that satisfies `p`.

This is the existential quantifier.

### Examples

```quint
assert(Set(1, 2, 3).exists(n => n == 2))
assert(not(Set(1, 2, 3).exists(n => n == 4)))
```

## forall

Signature: `pure def forall: (Set[a], (a) => bool) => bool`

`s.forall(p)` is true when all elements in `s` satisfy `p`.

This is the universal quantifier.

### Examples

```quint
assert(Set(1, 2, 3).forall(n => n > 0))
assert(not(Set(1, 2, 3).forall(n => n > 1)))
```

## in

Signature: `pure def in: (a, Set[a]) => bool`

`e.in(s)` is true when the element `e` is in the set `s`.

This is the set membership relation.

See also: `contains`

### Examples

```quint
assert(1.in(Set(1, 2, 3)))
assert(not(4.in(Set(1, 2, 3))))
```

