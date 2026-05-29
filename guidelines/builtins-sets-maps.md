# Quint builtins — sets and maps

## contains

Signature: `pure def contains: (Set[a], a) => bool`

`s.contains(e)` is true when the element `e` is in the set `s`.

This is the set membership relation.

See also: `in`

### Examples

```quint
assert(Set(1, 2, 3).contains(1))
assert(not(Set(1, 2, 3).contains(4)))
```

## union

Signature: `pure def union: (Set[a], Set[a]) => Set[a]`

`s1.union(s2)` is the set of elements that are in `s1` or in `s2`.

This is the set union operator.

### Examples

```quint
assert(Set(1, 2, 3).union(Set(2, 3, 4)) == Set(1, 2, 3, 4))
```

## intersect

Signature: `pure def intersect: (Set[a], Set[a]) => Set[a]`

`s1.intersect(s2)` is the set of elements that are in both sets `s1` and `s2`.

This is the set intersection operator.

### Examples

```quint
assert(Set(1, 2, 3).intersect(Set(2, 3, 4)) == Set(2, 3))
```

## exclude

Signature: `pure def exclude: (Set[a], Set[a]) => Set[a]`

`s1.exclude(s2)` is the set of elements in `s1` that are not in `s2`.

This is the set difference operator.

### Examples

```quint
assert(Set(1, 2, 3).exclude(Set(2, 3, 4)) == Set(1))
```

## subseteq

Signature: `pure def subseteq: (Set[a], Set[a]) => bool`

`s1.subseteq(s2)` is true when all elements in `s1` are also in `s2`.

### Examples

```quint
assert(Set(1, 2, 3).subseteq(Set(1, 2, 3, 4)))
assert(not(Set(1, 2, 3).subseteq(Set(1, 2))))
```

## filter

Signature: `pure def filter: (Set[a], (a) => bool) => Set[a]`

`s.filter(p)` is the set of elements in `s` that satisfy `p`.

### Examples

```quint
assert(Set(1, 2, 3).filter(n => n > 1) == Set(2, 3))
```

## map

Signature: `pure def map: (Set[a], (a) => b) => Set[b]`

`s.map(f)` is the set of elements obtained by applying `f` to

to each element in `s`. I.e., `{ f(x) : x \in s}`.

### Examples

```quint
assert(Set(1, 2, 3).map(n => n > 1) == Set(false, true, true))
```

## fold

Signature: `pure def fold: (Set[a], b, (b, a) => b) => b`

`l.fold(z, f)` reduces the elements in `s` using `f`, starting with `z`.

I.e., `f(...f(f(z, x0), x1)..., xn)`.

The order in which the elements are combined is unspecified, so
the operator must be associative and commutative or else it has undefined behavior.

### Examples

```quint
val sum = Set(1, 2, 3, 4).fold(0, (x, y) => x + y)
assert(sum == 10)
val mul = Set(1, 2, 3, 4).fold(1, (x, y) => x * y)
assert(mul == 24)
```

## powerset

Signature: `pure def powerset: (Set[a]) => Set[Set[a]]`

`s.powerset()` is the set of all subsets of `s`,
including the empty set and the set itself.

### Examples

```quint
assert(Set(1, 2).powerset() == Set(
  Set(),
  Set(1),
  Set(2),
  Set(1, 2)
))
```

## flatten

Signature: `pure def flatten: (Set[Set[a]]) => Set[a]`

`s.flatten()` is the set of all elements in the sets in `s`.

### Examples

```quint
assert(Set(Set(1, 2), Set(3, 4)).flatten() == Set(1, 2, 3, 4))
```

## allLists

Signature: `pure def allLists: (Set[a]) => Set[List[a]]`

`s.allLists()` is the set of all lists containing elements in `s`.
This is an infinite set unless `s` is the empty set.

Like other inifite sets, this is not supported in any execution/verification form.

### Examples

```quint
assert(Set(1, 2).allLists().contains([]))
assert(Set(1, 2).allLists().contains([1, 1, 1, 1, 2, 1]))
```

## allListsUpTo

Signature: `pure def allListsUpTo: (Set[a], int) => Set[List[a]]`

`s.allListsUpTo(l)` is the set of all lists of elements in `s` with length <= `l`

```quint
assert(Set(1, 2).allListsUpTo(1) == Set([], [1], [2]))
assert(Set(1).allListsUpTo(2) == Set([], [1], [1, 1]))
```

## getOnlyElement

Signature: `pure def getOnlyElement: (Set[a]) => a`

`s.getOnlyElement()` is, deterministically, the only element of `s`.
If the size of `s` is not 1, this operator has undefined behavior.

### Examples

```quint
assert(Set(5).getOnlyElement() == 5)
```

## chooseSome

Signature: `pure def chooseSome: (Set[a]) => a`

`s.chooseSome()` is, deterministically, one element of `s`.

### Examples

```quint
assert(Set(1, 2, 3).chooseSome() == 1)
assert(Set(1, 2, 3).filter(x => x > 2).chooseSome() == 3)
```

## oneOf

Signature: `pure def oneOf: (Set[a]) => a`

`s.oneOf()` is, non-deterministically, one element of `s`.

### Examples

```quint
nondet x = oneOf(Set(1, 2, 3))
assert(x.in(Set(1, 2, 3)))
```

## isFinite

Signature: `pure def isFinite: (Set[a]) => bool`

`s.isFinite()` is true when `s` is a finite set

### Examples

```quint
assert(Set(1, 2, 3).isFinite())
assert(!Nat.isFinite())
```

## size

Signature: `pure def size: (Set[a]) => int`

`s.size()` is the cardinality of `s`.

### Examples

```quint
assert(Set(1, 2, 3).size() == 3)
```

## get

Signature: `pure def get: ((a -> b), a) => b`

`m.get(k)` is the value for `k` in `m`.

If `k` is not in `m` then the behavior is undefined.

### Examples

```quint
pure val m = Map(1 -> true, 2 -> false)
assert(m.get(1) == true)
```

## keys

Signature: `pure def keys: ((a -> b)) => Set[a]`

`m.keys()` is the set of keys in the map `m`.

### Examples

```quint
pure val m = Map(1 -> true, 2 -> false)
assert(m.keys() == Set(1, 2))
```

## mapBy

Signature: `pure def mapBy: (Set[a], (a) => b) => (a -> b)`

`s.mapBy(f)` is the map from `x` to `f(x)` for each element `x` in `s`.

### Examples

```quint
pure val m = Set(1, 2, 3).mapBy(x => x * x)
assert(m == Map(1 -> 1, 2 -> 4, 3 -> 9))
```

## setToMap

Signature: `pure def setToMap: (Set[(a, b)]) => (a -> b)`

`s.setToMap()` for a set of pairs `s` is the map
from the first element of each pair to the second.

### Examples

```quint
pure val m = Set((1, true), (2, false)).setToMap()
assert(m == Map(1 -> true, 2 -> false))
```

## setOfMaps

Signature: `pure def setOfMaps: (Set[a], Set[b]) => Set[(a -> b)]`

`keys.setOfMaps(values)` is the set of all maps from `keys` to `values`.

### Examples

```quint
pure val s = Set(1, 2).setOfMaps(set(true, false))
assert(s == Set(
  Map(1 -> true, 2 -> true),
  Map(1 -> true, 2 -> false),
  Map(1 -> false, 2 -> true),
  Map(1 -> false, 2 -> false),
))
```

## set

Signature: `pure def set: ((a -> b), a, b) => (a -> b)`

`m.set(k, v)` is the map `m` but with the key `k` mapped to `v` if `k.in(keys(m))`

If `k` is not a key in `m`, this operator has undefined behavior.

### Examples

```quint
pure val m = Map(1 -> true, 2 -> false)
pure val m2 = m.set(2, true)
assert(m == Map(1 -> true, 2 -> false))
assert(m2 == Map(1 -> true, 2 -> true))
```

## setBy

Signature: `pure def setBy: ((a -> b), a, (b) => b) => (a -> b)`

`m.setBy(k, f)` is a map with the same keys as `m` but with `k` set to `f(m.get(k))`.

If `k` is not present in `m`, this operator has undefined behavior.

### Examples

```quint
pure val m = Map(1 -> true, 2 -> false)
pure val m2 = m.setBy(2, x => !x)
assert(m == Map(1 -> true, 2 -> false))
assert(m2 == Map(1 -> true, 2 -> true))
```

## put

Signature: `pure def put: ((a -> b), a, b) => (a -> b)`

`m.put(k, v)` is the map `m` but with the key `k` mapped to `v`.

### Examples

```quint
pure val m = Map(1 -> true, 2 -> false)
pure val m2 = m.put(2, true)
pure val m3 = m.put(3, true)
assert(m == Map(1 -> true, 2 -> false))
assert(m2 == Map(1 -> true, 2 -> true))
assert(m3 == Map(1 -> true, 2 -> false, 3 -> true))
```

