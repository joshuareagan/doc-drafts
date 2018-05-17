---
navhome: /docs/
sort: 2
next: true
title: Atoms and Auras
---

# Hoon Tutorial 1.2: Atoms and Their Types

In the last lesson you learned what an atom is: any unsigned integer.  In this lesson we'll learn about how atoms can represent other, more complicated types of data.  In order to do this, we'll first need to learn a little about Hoon's type system, particularly as it pertains to atoms.

## Hoon's Type System

Hoon's type system infers the type of an expression using syntactic clues from that expression.  In the most straightforward case of type inference, the expression is simply data as a [literal](https://en.wikipedia.org/wiki/Literal_(computer_programming)).  In the previous lesson you entered various atoms and other nouns into the dojo; these are all noun literals in Hoon.  You can use the `?` dojo operator to see both the product and the inferred type of an expression.  Restart your comet if you haven't already done so and try it out:

```
$ urbit mycomet
[...]
ames: on localhost, UDP 31337.
http: live (insecure, public) on 8080
http: live ("secure", public) on 8443
http: live (insecure, loopback) on 12321
~palnul_nocser:dojo>
```

Let's try `?` on `15`:

```
> 15
15

> ? 15
  @ud
15
```

The '`@ud`' is the inferred type of `15` (and of course `15` is the product).  The `@` is for 'atom' and the `ud` is for 'unsigned decimal'.  The latter part is the 'aura' of the atom---don't worry, we'll explain auras soon.

Keep in mind that the `?` operator isn't a part of Hoon itself; it's just a (useful) dojo tool.

One important role played by the type system is to make sure that the output of an expression is of the intended data type.  If the output is of the wrong type then the programmer did something wrong.  But how does Hoon know what the intended data type is?  The programmer must specify this explicitly by using a 'cast'.  To cast for an unsigned decimal atom, you'll use the `^-` rune along with the `@ud` from above.

What exactly does the `^-` rune do?  It compares the inferred type of some expression with the desired cast type.  If the expression's inferred type fits under the desired type, then the product of the expression is returned.

Let's try one in the dojo.  For the expression to be assessed we'll use `15` again:

```
> ^-(@ud 15)
15
```

Because `@ud` is the inferred type of `15`, the cast succeeds.  What about when the inferred type doesn't fit under the cast type?  You get a `nest-fail` crash:

```
> ^-(@ud [13 14])
nest-fail
[crash message]
```

Why 'nest-fail'?  The inferred type of `[13 14]` doesn't 'nest' under the cast type `@ud`.  It's a cell, not an atom.  But if we use the symbol for nouns, `*`, then the cast succeeds:

```
> ^-(* [13 14])
[13 14]
```

A cell of atoms is a noun, so the inferred type of `[13 14]` nests under `*`.  Every product of a Hoon expression nests under `*` because every product is a noun.

These applications of casting are all fairly trivial because the expressions under consideration are all literals.  We'll consider more interesting casts in later lessons.  For now, our goal is simply to understand the various ways that atoms can represent other types of data than unsigned integers.

## What's an Aura?

In the most straightforward sense, atoms represent unsigned integers.  But they can also be understood as representing signed integers, ASCII symbols, floating-point values, dates, binary numbers, hexadecimal numbers, and more.  Every atom is, in and of itself, just an unsigned integer; but Hoon keeps track of type information about each atom, and this info tells Hoon how to interpret the atom in question.

The piece of type information that determines how Hoon interprets an atom is called an *aura*.  An aura is indicated with `@` followed by some letters, e.g., `@ud` for unsigned decimal.

Hoon has a wide (but not extensible) variety of atom literal syntaxes.  Each literal syntax indicates to the Hoon type checker which predefined aura is intended.  Hoon can also pretty-print any aura literal it can parse.  Because atoms make great path nodes and paths make great URLs, all regular atom literal syntaxes use only URL-safe characters.

Here's a non-exhaustive list of auras, along with examples of corresponding literal syntax:

```
Aura         Meaning                        Example Literal Syntax
-------------------------------------------------------------------------
@d           date
  @da        absolute date                  ~2018.5.14..22.31.46..1435
  @dr        relative date (ie, timespan)   ~h5.m30.s12
@n           nil                            ~
@p           phonemic base (ship name)      ~sorreg-namtyv
@r           IEEE floating-point
  @rd        double precision  (64 bits)    .~6.02214085774e23
  @rh        half precision (16 bits)       .~~3.14
  @rq        quad precision (128 bits)      .~~~6.02214085774e23
  @rs        single precision (32 bits)     .6.022141e23
@s           signed integer, sign bit low
  @sb        signed binary                  --0b11.1000
  @sd        signed decimal                 --1.000.056
  @sv        signed base32                  -0v1df64.49beg
  @sw        signed base64                  --0wbnC.8haTg
  @sx        signed hexadecimal             -0x5f5.e138
@t           UTF-8 text (cord)              'howdy'
  @ta        ASCII text (knot)              ~.howdy
    @tas     ASCII text symbol (term)       %howdy
@u              unsigned integer
  @ub           unsigned binary             0b11.1000
  @ud           unsigned decimal            1.000.056
  @uv           unsigned base32             0v1df64.49beg
  @uw           unsigned base64             0wbnC.8haTg
  @ux           unsigned hexadecimal        0x5f5.e138
```

You'll notice that some of these auras are subsets of others.  For example, `@u` is for all unsigned auras.  But there are other, more specific auras; `@ub` for unsigned binary numbers, `@ux` for unsigned hexadecimal numbers, etc.

## Aura Inference in Hoon

Let's start with some examples in the Dojo using the `?` operator.  We'll focus on just the unsigned auras for now:

```
> 15
15

> ? 15
  @ud
15

> 0x15
0x15

> ? 0x15
  @ux
0x15
```

When you enter just `15`, the Hoon type checker infers from the syntax that its aura is `@ud` because you typed an unsigned integer in decimal notation.  Hence, when you use `?` to check the aura, you get `@ud`.

And when you enter `0x15` the type checker infers that its aura is `@ux`, because you used `0x` before the number to indicate the unsigned hexadecimal literal syntax.  In both cases, Hoon pretty-prints the appropriate literal syntax by using inferred type information from the input expression; the Dojo isn't (just) echoing what you enter.

More generally: for each atom expression in Hoon, use the literal syntax of an aura to force Hoon to interpret the atom as having that aura type.  For example, when you type `~sorreg-namtyv` Hoon will interpret it as an atom with aura `@p` and treat it accordingly.

Here's another example of type inference at work:

```
> (add 15 15)
30

> ? (add 15 15)
  @
30

> (add 0x15 15)
36

> ? (add 0x15 15)
  @
36
```

The `add` function in the Hoon standard library operates on all atoms, regardless of aura, and returns atoms with no aura specified.  Hoon isn't able to infer anything more specific than `@` for the product of `add`.  This is by design, however.  Notice that when you `add` a decimal and a hexadecimal above, the correct answer is returned (pretty-printed as a decimal).  This works for all of the unsigned auras:

```
> (add 100 0b101)
105

> (add 100 0xf)
115

> (add 0b1101 0x11)
30
```

The reason these add up correctly is that unsigned auras all map directly to the 'correct' atom underneath.  That is, `16`, `0b1.000`, and `0x10` are all the exact same atom, just with different literal syntax.  (This doesn't hold for signed versions of the auras!)

## Auras as 'Soft' Types

It's important to understand that Hoon's type system doesn't enforce auras as strictly as it does other types.  Auras are 'soft' types.  To see how this works, we'll take you through the process of converting the aura of an atom to another aura.

Hoon makes *some* effort to enforce that the correct aura is produced by an expression:

```
> ^-(@ud 0x10)
nest-fail

> ^-(@ud 0b10)
nest-fail

> ^-(@ux 100)
nest-fail
```

But there are ways around this.  First, you can cast to a more general aura, as long as the current aura is a subset of the cast aura.  E.g., `@ub` to `@u`, `@ux` to `@u`, `@u` to `@`, etc.  By doing this you're telling Hoon to throw away type information:

```
> ^-(@u 0x10)
16

> ? ^-(@u 0x10)
  @u
16

> ^-(@u 0b10)
2

> ? ^-(@u 0b10)
  @u
2
```

In fact, you can cast any atom all the way to the most general case `@`:

```
> ^-(@ 0x10)
16

> ? ^-(@ 0x10)
  @
16

> ^-(@ 0b10)
2

> ? ^-(@ 0b10)
  @
2
```

Anything of the general aura `@` can, in turn, be cast to more specific auras.  We can show this by embedding a cast expression inside another cast:

```
> ^-(@ud ^-(@ 0x10))
16

> ^-(@ub ^-(@ 0x10))
0b1.0000

> ^-(@ux ^-(@ 10))
0xa
```

Hoon uses the outermost cast to infer the type:

```
> ? ^-(@ub ^-(@ 0x10))
  @ub
0b1.0000
```

As you can see, an atom with one aura can always be converted to another aura.  For a convenient shorthand, you can do this conversion with irregular cast syntax, e.g. `` `@ud` ``, rather than using the `^-` rune twice:

```
> `@ud`0x10
16

> `@ub`0x10
0b1.0000

> `@ux`10
0xa
```

This is what we mean when we call auras 'soft' types.  The above examples show that the programmer can get around the type system for auras by casting up to `@` and then back down to the specific aura, say `@ub`; or by casting with `` `@ub` `` for short.

## Examples

So far you've only used the auras for unsigned integers.  Let's try some others.

### Signed integers

```
> -7
-7

> ? -7
  @sd

> --7
--7

> ? --7
  @sd
--7
```

`--7` means "positive 7".  `+7` might have been better but `+` is not URL-safe.

Hoon needs `--` to distinguish positive signed numbers such as `--7` from the unsigned `7`.  The latter is always understood by Hoon as an unsigned literal.  If you want Hoon to infer that a literal is signed you must explicitly include the `--`.

Whereas we could use the `add` function on different unsigned auras and still get the correct answer, this doesn't work for signed auras:

```
> (add -7 --7)
27
```

Why not?  `add` is happy to operate on any atom, regardless of aura, but it's primarily intended for auras whose literals map directly to numerically equivalent general atoms `@`.  For example, the `@ud` `7`, the `@ub` `0b111`, and the `@ux` `0x7` all map to the same `@` `7`.  Signed literals such as `--7` are different, because some underlying atoms are used for representing negative numbers.  We can see how this works by forcibly converting `@sd` atoms to `@ud`:

```
> `@ud`-1
1

> `@ud`--1
2

> `@ud`-7
13

> `@ud`--7
14
```

If you want to add signed atoms use the function `sum:si` instead of `add`:

```
> (sum:si -7 --7)
--0

> ? (sum:si -7 --7)
  @s
--0

> (sum:si --7 --0b10)
--9

> (sum:si --7 --0x10)
--23
```

### Exercise 1.2.1

Does Hoon's `@s` aura make a distinction between positive and negative zero, e.g., `-0` and `--0`?  Use what you've learned in this lesson so far to come up with a principled answer.  (As always, exercise solutions are at the bottom of the lesson.)

### Text and Symbols

Atoms can also represent text.  The `@t` aura is for UTF-8 text.  We call `@t` strings 'cords' (to distinguish them from another type of Hoon string we'll cover in a later lesson):

```
> ? 'Ürbit'
  @t
'Ürbit'

> `@ud`'Ürbit'
127.995.972.066.499

> `@ux`'Ürbit'
0x7469.6272.9cc3

> `@ux`'Ürbitt'
0x74.7469.6272.9cc3

> `@t`0x74
't'
```

As you can see by casting a cord to `@ux`, the first byte (here the `74` of `0x74`) represents the last character of the cord.  The next byte is the next to last character, and so on.

Hoon also has `@ta`, which is intended for ASCII text.  In practice, the Hoon parser rejects `@ta` literals that aren't in so-called 'kebab case'.  That is, lower-case letters, numerals, and hyphens.  Each `@ta` atom is called a 'knot'.

```
> ? ~.urbit
  @ta
~.urbit

> `@ux`~.this-is-kebab-case-123
0x3332.312d.6573.6163.2d62.6162.656b.2d73.692d.7369.6874
```

The Hoon type system doesn't enforce the `@ta` aura intentions (because it doesn't enforce auras in general):

```
> `@ta`'The quick brown fox jumped over the lazy dog.'
~.The quick brown fox jumped over the lazy dog.
```

However, the Hoon *parser* does prevent you from typing the literal `~.The quick brown fox jumped over the lazy dog.` into the dojo.  Try it!

The `@tas` aura is intended for 'symbols', i.e., kebab case strings following a `%`:

```
> %hello
%hello

> ^-(@tas %hello)
%hello

> ^-(@ %hello)
478.560.413.032

> ^-(@ 'hello')
478.560.413.032
```

### Dates

You can think of `now` as being an environment variable for the dojo that tells you what time it is:

```
> now
~2018.5.16..23.42.06..5da3

> ? now
  @da
~2018.5.16..23.42.08..3455
```

The `@da` aura is for 'absolute dates' represented with 128-bit atoms; 64 bits for the year, month, day, hour, minute, and second, 64 bits for fractions of a second.  There is also `@dr` for relative date:

```
> ~h6
~h6

> ? ~h6
  @dr
~h6

> ~h6.m32.s11
~h6.m32.s11

> `@ux`~h6
  @ux
0x5460.0000.0000.0000.0000
```

The literal `~h6.m32.s11` is for 6 hours, 32 minutes, and 11 seconds.  You can add `@dr` atoms together, or add a `@dr` atom to a `@da` atom.  Remember that the `add` function produces atoms with no aura, so you'll have to cast the the aura you want the dojo to pretty-print to:

```
> (add ~h2 ~h3)
332.041.393.326.771.929.088.000

> `@dr`(add ~h2 ~h3)
~h5

> `@dr`(add ~d1.m33 ~h7.m15.s12)
~d1.h7.m48.s12

> now
~2018.5.17..00.15.06..a0d0

> `@da`(add ~d1 now)
~2018.5.18..00.15.31..2df6
```

### Floats

Here are the float auras from the aura chart above:

```
Aura         Meaning                        Example Literal
-------------------------------------------------------------------------
@r           IEEE floating-point
  @rd        double precision  (64 bits)    .~6.02214085774e23
  @rh        half precision (16 bits)       .~~3.14
  @rq        quad precision (128 bits)      .~~~6.02214085774e23
  @rs        single precision (32 bits)     .6.022141e23
```

By this point you should be comfortable playing around with auras in the dojo.  One thing needs to be pointed out for float auras, however.  Atoms are fundamentally integers, and only when interpreted differently can they be understood as floats.  Floats obviously can't all map to numerically equivalent atoms:

```
> .6.022
.6.022

> ? .6.022
  @rs
.6.022

> `@`.6.022
1.086.370.873
```

As a result, the standard mathematical functions for atoms won't work correctly for floats:

```
> (add .6.022 .1.000)
2.151.724.089

> `@rs`(add .6.022 .1.000)
.-5.942123e-39
```

Instead, you'll have to call the variant function for that type of float: `add:rs` for adding `@rs` floats, `add:rd` for `@rd` floats, `add:rq` for `@rq` floats, and `add:rh` for `@rh` floats.

```
> (add:rs .6.022 .1.000)
.7.022

> (add:rd .~6.022 .~1.000)
.~7.022
```

## Custom Auras

Programmers can use their own auras if desired, keeping in mind however that Hoon won't support custom aura literals or pretty-printing.  The set of literals Hoon knows how to parse and/or print is fixed.  But there are plenty of good reasons to use your own user-defined auras.

If your program uses both fortnights and furlongs, Hoon will not parse a user-defined fortnight syntax. It will not print furlongs in the dojo.  But the type system can prevent you from accidentally passing a furlong to a function that expects a fortnight.

Remember also that Hoon interprets the aura string in one way: auras specialize to the right.  For example, `@u` atoms are interpreted as unsigned integers; `@ux` atoms are interpreted as unsigned *hexadecimal* integers.

```
> `@madeupaura`123
0x7b

> ^-(@madeupaura `@madeupaura`123)
0x7b

> ^-(@madeupaura `@ux`123)
nest-fail
```

## General and Constant Atoms

There's more to the type of an atom than its aura.  Another distinction is necessary.  Some atoms are *general* in type, meaning that the atom type in question includes all atoms.  For example, the type of the literal `17` is `@ud`; every atom can be of that type.  Up to this point of this lesson every atom has been general.

There are also atoms that are *constant* in type.  Constant atom types contain just one atom.  But that's not all.  Constant atom types contain just one atom *with just one aura*.  

For atomic constant literal syntax simply put `%` in front of the atom literal:

```
> ? 15
  @ud
15

> ? %15
  $15
%15

> ? %~h6
  $~h6
%~h6

> ? %hello
  $hello
%hello
```

The `$` on the type indicates the constant.  You can use the same literal syntax to cast:

```
> ^-(%15 %15)
%15
```

Keep in mind that, underneath, `15` and `%15` are the same atom:

```
> ^-(@ 15)
15

> ^-(@ %15)
15
```

But because they have different auras, `15` doesn't nest under the type for `%15`:

```
> ^-(%15 15)
nest-fail
```

When you cast using a constant as your type, you're asking for something very specific: one particular atom with one particular aura.  If both conditions aren't met, the result of the cast will be a `nest-fail`.

```
> `@`%0xf
15

> `@`%15
15

> ^-(%15 %0xf)
nest-fail

> ^-(%0xf %15)
nest-fail

> ^-(%0xf %0xf)
%0xf
```

## Common irregulars

There are a few special forms worth learning early.  Null `@n`, a special constant type for `0`:

```
> ~
~

> `@`~
0

> ^-(@n ~)
~
```

Flag `?`, which is for boolean values:

```
> |
%.n

> &
%.y

> ^-(? |)
%.n

> ^-(? &)
%.y
```

And ship names use `@p`:

```
> ~zod
~zod

> ~sorreg-namtyv
~sorreg-namtyv

> `@`~zod
0

> `@`~taglux-nidsep
6.095.360

> ^-(@p ~taglux-nidsep)
~taglux-nidsep
```

That's it for atoms!  Use `ctrl-d` to exit your comet, or move on to the next lesson.

## Exercise Answer

### 1.2.1

There is no distinction between positive and negative zero for `@s`.  To see this, let's cast each of `--0` and `-0` to `@s`:

```
> ^-(@s --0)
--0

> ^-(@s -0)
--0
```

Hoon pretty-prints them the same way, but can we be sure that they are the same thing?  Let's check the atom underneath:

```
> `@`--0
0

> `@`-0
0
```

They are indeed the same.
