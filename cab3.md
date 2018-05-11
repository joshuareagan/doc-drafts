---
navhome: /docs/
next: true
sort: 3
title: |_ "barcab"
---

# `|_ "barcab"`

form a *door*, a many-armed core with a sample.

## Syntax

Regular: *1-fixed*, then *battery*.

```
|_  \\toro\\
  ++  \\term\\  \\hoon\\
  ++  \\term\\  \\hoon\\
  ... 
  --
```

Example:

```
++  hi
  |_  a=@ud                         :: the
  ++  hug  (add a 2)                :: 'hi'
  ++  mug  (sub a 2)                :: door
  --                                ::
```


## Expands to

```
=|  p
|%  q
==
```

## Discussion

A door is the general case of a gate (function).  A gate is a door with only one arm, the empty name `$`.

Other languages have no real equivalent of a door, but we often see the pattern of multiple functions with the same argument list, or with shared argument structure.  In Hoon, this shared structure is a door.

Calling a door is like calling a gate, `(gate sample)`, but the caller also needs to specify the arm:

```
~(arm door sample)
```

For example, to call `hug` in the `++hi` door with the sample `3`, we would write `~(hug hi 3)`.  This is the irregular form of `%~(hug hi 3)`, ["cen-sig"](../../cen/sig).

## Examples

A trivial door:

```
/~zod:dojo> =mol  |_  a=@ud
                  ++  succ  +(a)
                  ++  prev  (dec a)
                  --
/~zod:dojo> ~(succ mol 1)
2
/~zod:dojo> ~(succ mol ~(succ mol ~(prev mol 5)))
6
```

A more interesting door, from the kernel library:

```
++  ne
  |_  tig=@
  ++  d  (add tig '0')
  ++  x  ?:((gte tig 10) (add tig 87) d)
  ++  v  ?:((gte tig 10) (add tig 87) d)
  ++  w  ?:(=(tig 63) '~' ?:(=(tig 62) '-' ?:((gte tig 36) (add tig 29) x)))
  --
```

The `ne` door prints a digit in base 10, 16, 32 or 64:

```
~zod:dojo> `@t`~(x ne 12)
'c'
```
