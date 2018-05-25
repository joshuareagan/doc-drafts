---
navhome: /docs/
sort: 3
title: Introduction to Cell Types
---

# Hoon Tutorial 1.3: Introduction to Cell Types

In the last lesson you learned about how Hoon's type system deals with atoms.  This lesson is an introduction to how Hoon's type system deals with cells.  We will use the dojo `?` operator and the cast rune `^-` to explore cell types.

## Don't Overthink This

Here's some good news: if you understood the last lesson, then this one will be easy.  You'll learn how to create simple cell types made up of various atom types.  There is (much) more to learn about Hoon's type system but we aren't going to cover it all in this lesson.

## Generic Cells

Use the `^` symbol to cast for any cell:

```
> ^-(^ [12 13])
[12 13]

> ^-(^ [[12 13] 14])
[[12 13] 14]

> ^-(^ [[12 13] [14 15 16]])
[[12 13] [14 15 16]]

> ^-(^ 123)
nest-fail

> ^-(^ 0x10)
nest-fail
```

If the expression to be evaluated produces a cell, the cast succeeds; if the expression evaluates produces an atom, the cast fails with a `nest-fail` crash.

The downside of using `^` for casts is that Hoon will infer only that the product of the expression is a cell; it won't know what kind of cell is produced.

```
> ? ^-(^ [12 13])
  {* *}
[12 13]

> ? ^-(^ [[12 13] 14])
  {* *}
[[12 13] 14]

> ? ^-(^ [[12 13] [14 15 16]])
  {* *}
[[12 13] [14 15 16]]
```

When we use the `?` operator to see the type inferred by Hoon for the expression, in all three of the above cases the same thing is returned: `{* *}`.  The `*` symbol indicates the type for any noun, and the curly braces `{ }` indicate a cell.  Every cell in Hoon is a cell of nouns; remember that cells are defined as pairs of nouns.

Yet the cell `[[12 13] [14 15 16]]` is a bit more complex than the cell `[12 13]`.  Can we use the type system to distinguish them?  Yes.

## Getting More Specific

What if you want to cast for a particular kind of cell?  You can use square brackets when casting for a specific cell type.  For example, if you want to cast for a cell in which the head and the tail must each be an atom, then simply cast using `[@ @]`:

```
> ^-([@ @] [12 13])
[12 13]

> ? ^-([@ @] [12 13])
  {@ @}
[12 13]

> ^-([@ @] 12)
nest-fail

> ^-([@ @] [[12 13] 14])
nest-fail
```

The `[@ @]` cast accepts any expression that evaluates to a cell with exactly two atoms, and crashes with a `nest-fail` for any expression that evaluates to something different.  The expression `12` doesn't evaluate to a cell; and while the expression `[[12 13] 14]` *does* evaluate to a cell, the left-hand side isn't an atom, but is instead another cell.

You can get even more specific about the kind of cell you want by using atom auras:

```
> ^-([@ud @ux] [12 0x10])
[12 0x10]

> ^-([@ub @ux] [0b11 0x10])
[0b11 0x10]

> ? ^-([@ub @ux] [0b11 0x10])
  {@ub @ux}
[0b11 0x10]

> ^-([@ub @ux] [12 13])
nest-fail
```

You are also free to embed more square brackets `[ ]` to indicate cells within cells:

```
> ^-([[@ud @sb] @ux] [[12 --0b1101] 0xdead.beef])
[[12 --0b1101] 0xdead.beef]

> ? ^-([[@ud @sb] @ux] [[12 --0b1101] 0xdead.beef])
  {{@ud @sb} @ux}
[[12 --0b1101] 0xdead.beef]

> ^-([[@ @] @] [12 13])
nest-fail
```

You can also specify just part of the type structure, leaving other parts more general.  Keep in mind that when you do this, Hoon's type system will infer the type from these more general casts.  Type information will be thrown away:

```
> ^-([^ @ux] [[12 --0b1101] 0xdead.beef])
[[12 26] 0xdead.beef]

> ? ^-([^ @ux] [[12 --0b1101] 0xdead.beef])
  {{* *} @ux}
[[12 26] 0xdead.beef]

> ^-(* [[12 --0b1101] 0xdead.beef])
[[12 26] 3.735.928.559]

> ? ^-(* [[12 --0b1101] 0xdead.beef])
  *
[[12 26] 3.735.928.559]
```

Because every piece of Hoon data is a noun, everything nests under `*`.  When you cast to `*` you can see the raw noun with cells as brackets and atoms as unsigned integers.

## Beyond Fixed Cells

We haven't exhausted the types you can build in Hoon, naturally.  So far you've only dealt with cells of a fixed length and structure.  In a later lesson we'll talk about how to define new types that are [recursive](https://en.wikipedia.org/wiki/Recursion) in nature, and hence of arbitrary length.  (The `*` type is recursive, in parallel with the recursive definition of a noun.)
