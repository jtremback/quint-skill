# Quint builtins — lists and integer arithmetic

## append

Signature: `pure def append: (List[a], a) => List[a]`

`l.append(e)` is the list `l` with the element `e` appended.

### Examples

```quint
assert(List(1, 2, 3).append(4) == List(1, 2, 3, 4))
```

## concat

Signature: `pure def concat: (List[a], List[a]) => List[a]`

`l1.concat(l2)` is the list `l1` with `l2` concatenated at the end.

### Examples

```quint
assert(List(1, 2, 3).append(List(4, 5, 6)) == List(1, 2, 3, 4, 5, 6))
```

## head

Signature: `pure def head: (List[a]) => a`

`l.head()` is the first element of `l`.

If `l` is empty, the behavior is undefined.

### Examples

```quint
assert(List(1, 2, 3).head() == 1)
```

## tail

Signature: `pure def tail: (List[a]) => List[a]`

`l.tail()` is the list `l` without the first element.

If `l` is empty, the behavior is undefined.

### Examples

```quint
assert(List(1, 2, 3).tail() == List(2, 3))
```

## length

Signature: `pure def length: (List[a]) => int`

`l.length()` is the length of the list `l`.

### Examples

```quint
assert(List(1, 2, 3).length() == 3)
```

## nth

Signature: `pure def nth: (List[a], int) => a`

`l.nth(i)` is the `i+1`th element of the list `l`.

If `i` is negative or greater than or equal to `l.length()`, the behavior is undefined.

### Examples

```quint
assert(List(1, 2, 3).nth(1) == 2)
```

## indices

Signature: `pure def indices: (List[a]) => Set[int]`

`l.indices()` is the set of indices of `l`.

### Examples

```quint
assert(List(1, 2, 3).indices() == Set(0, 1, 2))
```

## replaceAt

Signature: `pure def replaceAt: (List[a], int, a) => List[a]`

`l.replaceAt(i, e)` is the list `l` with the `i+1`th element replaced by `e`.

If `i` is negative or greater than or equal to `l.length()`, the behavior is undefined.

### Examples

```quint
assert(List(1, 2, 3).replaceAt(1, 4) == List(1, 4, 3))
```

## slice

Signature: `pure def slice: (List[a], int, int) => List[a]`

`l.slice(i, j)` is the list of elements of `l` between indices `i` and `j`.

`i` is inclusive and `j` is exclusive.

The behavior is undefined when:

- `i` is negative or greater than or equal to `l.length()`.
- `j` is negative or greater than `l.length()`.
- `i` is greater than `j`.

### Examples

```quint
assert(List(1, 2, 3, 4, 5).slice(1, 3) == List(2, 3))
```

## range

Signature: `pure def range: (int, int) => List[int]`

`range(i, j)` is the list of integers between `i` and `j`.

`i` is inclusive and `j` is exclusive.

The behavior is undefined when `i` is greater than `j`.

### Examples

```quint
assert(range(1, 3) == List(1, 2))
```

## select

Signature: `pure def select: (List[a], (a) => bool) => List[a]`

`l.select(p)` is the list of elements of `l` that satisfy the predicate `p`.

Preserves the order of the elements.

### Examples

```quint
assert(List(1, 2, 3).select(x -> x % 2 == 0) == List(2))
```

## foldl

Signature: `pure def foldl: (List[a], b, (b, a) => b) => b`

`l.foldl(z, f)` reduces the elements in `l` using `f`,
starting with `z` from the left.

I.e., `f(f(f(z, x0), x1)..., xn)`.

### Examples

```quint
pure val sum = List(1, 2, 3, 4).foldl(0, (x, y) => x + y)
assert(sum == 10)
pure val l = List(1, 2, 3, 4).foldl(List(), (l, e) => l.append(e))
assert(l == List(1, 2, 3, 4))
```

## iadd

Signature: `pure def iadd: (int, int) => int`

`a.iadd(b)` is the integer addition of `a` and `b`.

It can be used in the infix form as `+` or as a named operator `iadd`.

## isub

Signature: `pure def isub: (int, int) => int`

`a.isub(b)` is the integer subtraction of `b` from `a`.

It can be used in the infix form as `-` or as a named operator `isub`.

## imul

Signature: `pure def imul: (int, int) => int`

`a.imul(b)` is the integer multiplication of `a` and `b`.

It can be used in the infix form as `*` or as a named operator `imul`.

## idiv

Signature: `pure def idiv: (int, int) => int`

`a.idiv(b)` is the integer division of `a` by `b`.

It can be used in the infix form as `/` or as a named operator `idiv`.

## imod

Signature: `pure def imod: (int, int) => int`

`a.imod(b)` is the integer modulus of `a` and `b`.

It can be used in the infix form as `%` or as a named operator `imod`.

## ipow

Signature: `pure def ipow: (int, int) => int`

`a.ipow(b)` is the integer exponentiation of `a` by `b`.

It can be used in the infix form as `^` or as a named operator `ipow`.

## ilt

Signature: `pure def ilt: (int, int) => bool`

`a.ilt(b)` is the integer less than comparison of `a` and `b`.

It can be used in the infix form as `<` or as a named operator `ilt`.

## igt

Signature: `pure def igt: (int, int) => bool`

`a.igt(b)` is the integer greater than comparison of `a` and `b`.

It can be used in the infix form as `>` or as a named operator `igt`.

## ilte

Signature: `pure def ilte: (int, int) => bool`

`a.ilte(b)` is the integer less than or equal to comparison of `a` and `b`.

It can be used in the infix form as `<=` or as a named operator `ilte`.

## igte

Signature: `pure def igte: (int, int) => bool`

`a.igte(b)` is the integer greater than or equal to comparison of `a` and `b`.

It can be used in the infix form as `>=` or as a named operator `igte`.

## iuminus

Signature: `pure def iuminus: (int) => int`

`iuminus(a)` is `-1 * a`.

This is the unary minus operator

## to

Signature: `pure def to: (int, int) => Set[int]`

`i.to(j)` is the set of integers between `i` and `j`.

`i` is inclusive and `j` is inclusive.

The behavior is undefined when `i` is greater than `j`.

### Examples

```quint
assert(1.to(3) == Set(1, 2, 3))
```

