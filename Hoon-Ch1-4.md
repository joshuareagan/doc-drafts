---
navhome: /docs/
sort: 4
title: Subject-oriented programming
---

# Hoon Tutorial 1.4: Subject-oriented Programming

Hoon isn't an object-oriented programming language; it's a "subject-oriented" programming language.  But what's this "subject"?

## A Start

For now we can say three things about the subject: (1) every Hoon expression is evaluated relative to some subject; (2) roughly, the subject defines the environment in which a Hoon expression is evaluated; and (3) the subject is a noun.

In fact, you already created some subjects in lesson 1.1 when you used the `+` operator to return a fragment of a noun:

```
> +1:[[11 22] 33]
[[11 22] 33]

> +2:[[11 22] 33]
[11 22]

> +3:[[11 22] 33]
33
```

The `:` operator does two things.  First, it evaluates the expression on the right-hand side; and second, it evaluates the expression on the left-hand side, using the product of the right-hand side as its subject.

In the examples above, the expression on the right-hand, `[[11 22] 33]`, evaluates to itself:

```
> [[11 22] 33]
[[11 22] 33]
```

...so the subject for the left-hand expression is simply `[[11 22] 33]`.  When we use the `+` operator on the left the appropriate fragment of the subject is returned.  E.g., `+2`, returns whatever is at address `2` of the subject.  Hence, we may read `+2:[[11 22] 33]` as "address `2` of the subject produced by `[[11 22] 33]`".

Let's create a subject with some computations:

```
> +1:[[(add 22 33) (mul 2 6)] 23]
[[55 12] 23]

> +2:[[(add 22 33) (mul 2 6)] 23]
[55 12]

> +3:[[(add 22 33) (mul 2 6)] 23]
23

> +4:[[(add 22 33) (mul 2 6)] 23]
55
```

You've already seen `add`.  In Hoon we use `mul` to multiply atoms.

## Legs of the Subject

For now, we will consider only simple subjects and simple ways of looking at pieces of the subject.

The subject is a noun, just like any other piece of Hoon data.  We've previously discussed how any noun can be understood as a binary tree.  E.g., `[[4 5] [6 [14 15]]]`:

```
     [[4 5] [6 [14 15]]]
             .
            / \
          .     .
         / \   / \
        4   5 6    .
                  / \
                 14 15
```

Each fragment of a noun is itself a noun, and hence can be understood as a binary tree as well.  Each fragment or 'subtree' sticks out of the original tree, sort of like a *leg*.  A 'leg' is just a subtree of the subject.

There are various ways to produce a leg.  First we'll show you how to produce a leg using *limb* expressions.  (There are two kinds of limbs: (1) arms, and (2) legs.  We'll talk about 'arms' a little later.)

### Limb Expressions

Let's cover some of the kinds of limb expressions available to you in Hoon.  First we'll explain the ones that return a leg according to its subject address.

#### `+` operator

We've already used this.  For any unsigned integer `n`, `+n` returns the limb of the subject at address `n`.  If there is no such limb, the result is a crash.

#### `.` expression

Using `.` as an expression returns the entire subject.  It's equivalent to `+1`.

```
> .:[[4 5] [6 [14 15]]]
[[4 5] 6 14 15]

> .:[4 5]
[4 5]

> .:4
4
```

#### 'lark' expressions (`+`, `-`, `+>`, `+<`, `->`, `-<`, etc.)

Using `-` by itself returns the head of the subject, and using `+` by itself returns the tail:

```
> -:[[4 5] [6 [14 15]]]
[4 5]

> +:[[4 5] [6 [14 15]]]
[6 14 15]
```

These only work if the subject is a cell, naturally.  An atom doesn't have a head or a tail.

You can combine `+` or `-` with either of `>` or `<` to get a more specific limb of the subject.  `-<` returns the head of the head; `->` returns the tail of the head; `+<` returns the head of the tail; and `+>` returns the tail of the tail.

```
> -<:[[4 5] [6 [14 15]]]
4

> ->:[[4 5] [6 [14 15]]]
5

> +<:[[4 5] [6 [14 15]]]
6

> +>:[[4 5] [6 [14 15]]]
[14 15]
```

By alternating the `+`/`-` symbols with `<`/`>` symbols, you can grab an even more specific limb of the subject:

```
> +>-:[[4 5] [6 [14 15]]]
14

> +>+:[[4 5] [6 [14 15]]]
15
```

#### `&` and `|` operators

Let's say you have a cell of nouns arranged as a list, like this:

```
> ['first' 'second' 'third' 'fourth' 'fifth']
['first' 'second' 'third' 'fourth' 'fifth']
```

The first noun is at `+2`, the second is at `+6`, the third is at `+14`, and so on.  (Try it!)  That's because the above noun is really the following:

```
> ['first' ['second' ['third' ['fourth' 'fifth']]]]
['first' 'second' 'third' 'fourth' 'fifth']
```

...with the superfluous brackets removed.  If you don't want to work out the address of each noun in the list, however, you can just use the `&` operator.

`&n` returns the `n`th noun of a cell that has at least `n + 1` nouns:

```
> &1:['first' 'second' 'third' 'fourth' 'fifth']
'first'

> &2:['first' 'second' 'third' 'fourth' 'fifth']
'second'

> &3:['first' 'second' 'third' 'fourth' 'fifth']
'third'

> &4:['first' 'second' 'third' 'fourth' 'fifth']
'fourth'
```

`|n` returns everything after `&n`:

```
> |1:['first' 'second' 'third' 'fourth' 'fifth']
['second' 'third' 'fourth' 'fifth']

> |2:['first' 'second' 'third' 'fourth' 'fifth']
['third' 'fourth' 'fifth']

> |3:['first' 'second' 'third' 'fourth' 'fifth']
['fourth' 'fifth']

> |4:['first' 'second' 'third' 'fourth' 'fifth']
'fifth'
```

### Other Limb Expressions: Names

Dealing with specific addresses of the subject can be cumbersome even when the subject is small.  When the subject is a really large noun -- as is often the case -- it's downright impractical.  Thankfully there's a more convenient method for getting a limb out of the subject: using names.

There are two kinds of names for the two kinds of limbs: arm names and leg names.  We aren't yet ready to talk about arms.  For now, let's try to understand names of legs.

#### Faces

Hoon doesn't have variables like other programming languages do; it has 'faces'.  Faces are like variables in certain respects, but not in others.  Faces play various roles in Hoon, but most frequently faces are used simply as labels for legs.

For a face name you may use a combination of lowercase letters, numbers, and the '`-`' symbol.  Some examples: `b`, `c3`, `var`, `this-is-kebab-case123`.  Faces may not begin with numbers.

There are various ways to affix a face to a limb of the subject, but for now we'll use the simplest method: face literal syntax.

```
> b=5
b=5

> [b=5 c=6]
[b=5 c=6]

> -:[b=5 c=6]
b=5

> b:[b=5 c=6]
5

> b:[[4 b=5] [c=6 d=[14 15]]]
5

> d:[[4 b=5] [c=6 d=[14 15]]]
[14 15]
```

To be clear, putting a face on a limb doesn't add anything to or change the underlying noun.  Faces are understood by Hoon as a kind of metadata.  Faces are tracked and managed by Hoon, but they aren't part of the actual noun in question:

```
> ? b=5
  b/@ud
b=5

> ^-(* b=5)
5

> ? ^-(* b=5)
  *
5

> ? [[4 b=5] [c=6 d=[14 15]]]
  {{@ud b/@ud} c/@ud d/{@ud @ud}}
[[4 b=5] c=6 d=[14 15]]

> ^-(* [[4 b=5] [c=6 d=[14 15]]])
[[4 5] 6 14 15]

> ? ^-(* [[4 b=5] [c=6 d=[14 15]]])
  *
[[4 5] 6 14 15]
```

Notice that when we cast a noun with faces to a generic noun, `*`, the face gets thrown away.  Faces are tracked by Hoon's type system.  That means when type data is thrown away, face data is too.

If you use a face that isn't in the subject you'll get a `find.[face]` crash:

```
> a:[b=12 c=14]
-find.a
[crash message]
```

You can even give your faces faces:

```
> b:[b=c=123 d=456]
c=123
```

#### Duplicate Faces

There is no restriction against using the same face name for multiple limbs of the subject:

```
> [[4 b=5] [b=6 b=[14 15]]]
[[4 b=5] b=6 b=[14 15]]

> b:[[4 b=5] [b=6 b=[14 15]]]
5
```

Why does this return `5` rather than `6` or `[14 15]`?  When a face is evaluated against a subject, a head-first binary tree search occurs starting at address `1` of the subject.  If there is no matching face at address `n`, first the head of `n` is searched and then `n`'s tail.

But what is being searched?  There is no face information in the noun!  The search happens in the inferred type of the noun, and on the basis of that search Hoon decides which leg of the subject is intended:

```
> [[4 b=5] b=6 b=[14 15]]
[[4 b=5] b=6 b=[14 15]]

> ^-(* [[4 b=5] b=6 b=[14 15]])
[[4 5] 6 14 15]

> ? [[4 b=5] b=6 b=[14 15]]
  {{@ud b/@ud} b/@ud b/{@ud @ud}}
[[4 b=5] b=6 b=[14 15]]
```

There are cases when you don't want the limb of the first matching face.  You can 'skip' a face using the `^` operator before the face.  Upon discovery of the first match at address `n`, the search skips `n` (as well as its children) and continues the search elsewhere:

```
> ^b:[[4 b=5] [b=6 b=[14 15]]]
6
```

You can stack `^` symbols to skip more than one matching face:

```
> a:[[[a=1 a=2] a=3] a=4]
1

> ^a:[[[a=1 a=2] a=3] a=4]
2

> ^^a:[[[a=1 a=2] a=3] a=4]
3

> ^^^a:[[[a=1 a=2] a=3] a=4]
4
```

It's important to realize that when a face is skipped at some address `n`, neither the head nor the tail of `n` will be searched:

```
> b:[b=[a=1 b=2 c=3] a=11]
[a=1 b=2 c=3]

> ^b:[b=[a=1 b=2 c=3] a=11]
-find.^b
```

The first `b`, `b=[a=1 b=2 c=3]`, is skipped; so the entire head of the subject is skipped.  The tail has no `b`; so `^b` doesn't resolve to a limb when the subject is `[b=[a=1 b=2 c=3] a=11]`.

How do we get to that `b=2`??  We use a wing.

## Wings

A wing expression indicates a search path into the subject, and it's made up of a series of limbs.  To indicate a wing, connect limb expressions with the `.` symbol:

```
[limb1].[limb2].[limb3].[and so on....]
```

You can read this as `limb1` in `limb2` in `limb3`, etc.  You can use a wing to get the value of `b` inside the head of `[b=[a=1 b=2 c=3] a=11]`: `b.b`.

```
> b.b:[b=[a=1 b=2 c=3] a=11]
2

> a.b:[b=[a=1 b=2 c=3] a=11]
1

> c.b:[b=[a=1 b=2 c=3] a=11]
3

> a:[b=[a=1 b=2 c=3] a=11]
11

> b.a:[b=[a=1 b=2 c=3] a=11]
-find.b.a
```

Here are some other wing examples:

```
> g.s:[s=[c=[d=12 e='hello'] g=[h=0xff i=0b11]] r='howdy']
[h=0xff i=0b11]

> c.s:[s=[c=[d=12 e='hello'] g=[h=0xff i=0b11]] r='howdy']
[d=12 e='hello']

> e.c.s:[s=[c=[d=12 e='hello'] g=[h=0xff i=0b11]] r='howdy']
'hello'

> +3:[s=[c=[d=12 e='hello'] g=[h=0xff i=0b11]] r='howdy']
r='howdy'

> r.+3:[s=[c=[d=12 e='hello'] g=[h=0xff i=0b11]] r='howdy']
'howdy'
```

## Exploring the Subject

Earlier we said that every Hoon expression is evaluated relative to a subject.  We then showed how the `:` operator uses a Hoon expression on the right-hand side in order to set the subject for the expression on the left.

```
> b:[b='Hello world!' c='This is the tail of the LHS subject']
'Hello world!'
```

Clearly the left-hand Hoon expression has a subject.  But what about the right?  What's its subject?  In fact, you already know how to find the answer.  The subject is a noun.  To see that noun, use a limb: `+1`.  You'll see something like the following:

```
> +1
[ [ our=~mipfyl-lodsel-halrul-dilbyt--balluc-havfes-wordeb-pagrem
    now=~2018.5.24..23.51.31..f5f2
      eny
    \/0vfq.npac6.eklcb.19rir.el97m.1jaq4.m3ths.ejfa5.cpjiu.fk1uh.kqf9r.jncko.\/
      7eorl.ciggn.4hi3g.gna8v.qrpvc.rd32q.t60pj.8v7ev.27g2c
    \/                                                                       \/
  ]
  ~
  <16.swm 19.mtp 41.lur 112.twd 224.bkh 54.tyv 119.wpb 31.ohr 1.jmk $143>
]
```

We can't explain what all of this stuff is just yet, but you should be able to recognize some of it.  There is a face, `our`, serving as a label for an `@p`, a comet name.  Another face, `now`, is a label for a `@da`, an absolute date.

```
> our
~mipfyl-lodsel-halrul-dilbyt--balluc-havfes-wordeb-pagrem

> now
~2018.5.24..23.55.34..1ef0
```

Also embedded in that subject is the Hoon 'standard library', a library of commonly-used functions.  We've already used two of them: `add` and `mul`.  Have you noticed that these function names look like faces?  They aren't faces, but Hoon searches for them like it searches for faces.  When we call `add`,

```
> (add 22 33)
55
```

...it only works because Hoon finds `add` somewhere in the subject.  We can make a subject that doesn't have `add` in it and try to use it:

```
> (add 22 33):[123 456]
-find.add

> (add 22 33):['This subject' 'is austere.']
-find.add
```

It doesn't work because, as far as Hoon is concerned, `add` doesn't exist for the expression on the left.  We can put `add` back into the subject manually, however:

```
> (add 22 33):[123 456 add]
55
```

And it works!  The subject of the right-hand side expression of the `:` is the same implicit subject we have in the dojo.

## Dojo Faces

We're playing in the Urbit `:dojo`.  The dojo is sort of like a cross between a Lisp REPL and the Unix shell.  (If you know the Unix shell, the Urbit command line has the same history model and Emacs editing commands.)

You can use the dojo to add a noun with a face to the subject.  Type

```
> =a 37
```

You've set the face `a` to the value `37`.  The dojo puts this binding in the Hoon subject for expressions to use. So you can write:

```
> a
37

> [4 a]
[4 37]

> ? [4 a]
  {@ud @ud}
[4 37]
```

To unbind `a`, just omit the value:

```
> =a
```

You removed `a` from the subject.  (We can bind it again later, or bind over it without removing it.)

```
> [4 a]
-find.a
```

### Dojo Fun

We can use dojo faces to get used to how wings work.  

```
> =a [g=37 b=[%hi c=.6.28 d=~m45] h=0xdead.beef]

> a
[g=37 b=[%hi c=.6.28 d=~m45] h=0xdead.beef]

> ? a
  {g/@ud b/{$hi c/@rs d/@dr} h/@ux}
[g=37 b=[%hi c=.6.28 d=~m45] h=0xdead.beef]

> g.a
37

> b.a
[%hi c=.6.28 d=~m45]

> c.b.a
.6.28

> +2.a
g=37

> +3.a
[b=[%hi c=.6.28 d=~m45] h=0xdead.beef]
```

### Creating a Modified Noun

Let's create a mutant version of the noun we have stored in `a`.  To do this, type in the face `a` followed by the desired modifications in parentheses:

```
> a
[g=37 b=[%hi c=.6.28 d=~m45] h=0xdead.beef]

> a(g 44)
[g=44 b=[%hi c=.6.28 d=~m45] h=0xdead.beef]

> a(b 'hello world!')
[g=37 b='hello world!' h=0xdead.beef]

> a(b 'hello world!', h 0xfeed.beef)
[g=37 b='hello world!' h=0xfeed.beef]

> a(b 'hello world!', h 0xfeed.beef, g 123)
[g=123 b='hello world!' h=0xfeed.beef]

> a(+2 r=457.842)
[r=457.842 b=[%hi c=.6.28 d=~m45] h=0xdead.beef]
```

At this point you should have at least a rough idea of what the subject is, and a fair understanding of how to get legs of the subject using limbs and wings.  But legs are only one kind of limb -- we'll talk about arms in the next lesson.
