---
navhome: /docs/
sort: 6
next: true
title: Cores
---

# Cores

A core is a particularly useful kind of Hoon data: a cell whose head is a battery and whose tail is a payload:

```
          [core]
         /      \
  [battery]    [payload]
```

The battery is a series of arms, each of which is intended to be evaluated with the parent core as the subject.  Some cores have samples and some don't.  Those that do have a payload structured as a cell whose head is the sample and whose tail is the context:

```
          [core]
         /      \
  [battery]    [payload]
               /       \
         [sample]     [context]
```

The context is all the relevant data needed to evaluate the arms in the battery correctly, apart from the sample.  In cores with no sample, there is no distinction between 'payload' and 'context'.

In this lesson you'll learn how to produce various kinds of cores.  You've already been using one kind of core, produced with the `|=` rune: gates.  A gates is a one-armed core with a sample.  But in this lesson you'll learn how to produce many-armed cores, with and without samples.  You'll also learn how to make use of these cores in your own programs.

## Project Euler 1

Let's start with an example program illustrating the use of a many-armed core.

[Problem 1 of Project Euler](https://projecteuler.net/problem=1) is as follows:

```
If we list all the natural numbers below 10 that are multiples of
3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.

Find the sum of all the multiples of 3 or 5 below 1000.
```

The program below provides a solution:

```
|=  a=@
=<  (euler-sum a)
::
|%
++  threes
  |=  a/@
  =|  b/@
  |-  ^-  @
  ?:  (lth a b)
    0
  (add b $(b (add 3 b)))
::
++  fives
  |=  a/@
  =|  b/@
  |-  ^-  @
  ?:  (lte a b)
    0
  ?:  =((mod b 3) 0)
    $(b (add b 5))
  (add b $(b (add b 5)))
::
++  euler-sum
  |=  a=@
  (add (threes a) (fives a))
--
```

Save this as `euler1.hoon` in the `/gen` directory of your urbit's pier and run it in the dojo:

```
> +euler1 1.000
233.168
```

Success!  (We could use the same program to find the sum of the multiples of 3 and 5 for numbers other than `1.000` as well.)

Let's take a closer look at the code (with part of it elided).

```
|=  a=@
=<  (euler-sum a)
::
|%
      ...
--
```

This uses `|=` to produce a gate whose sample is an atom labeled `a`.  The `=<` rune creates a new subject using `|%` and evaluates `(euler-sum a)` against that subject.  `euler-sum` isn't a function in the Hoon standard library---it's a user-defined gate.  This gate is only defined in the core created by the `|%` expression.

### `|%` Produce a Core with No Sample

The `|%` rune produces a core with no sample; nearly always it's used to create a multi-arm core.  The `|%` rune takes a series of arm definitions.  The expression is terminated with the `--` rune.  From the above example:

```
|%
++  threes
  |=  a/@
  =|  b/@
  |-  ^-  @
  ?:  (lth a b)
    0
  (add b $(b (add 3 b)))
::
++  fives
  |=  a/@
  =|  b/@
  |-  ^-  @
  ?:  (lte a b)
    0
  ?:  =((mod b 3) 0)
    $(b (add b 5))
  (add b $(b (add b 5)))
::
++  euler-sum
  |=  a=@
  (add (threes a) (fives a))
--
```

As you can see, the expression begins with `|%`, ends with `--`, and in between it has three arm definitions.  Each arm definition begins with `++` followed by an arm name.  After each name is the Hoon expression for that arm.  In this example, each arm defines a gate.  Once this core is in the subject these gates can be called like any other standard library gate.

The `|%` rune produces a core with no sample.  The composition of the head of this core, i.e., the battery, is determined by the arm definitions of the `|%` expression.  The tail of the core, i.e., the payload, is the subject against which the `|%` expression was evaluated.  This is important -- the arm definitions can use any data from the subject, and that data is guaranteed to be available to the arm regardless of the arm evaluation context.  Each arm can make calls to any of the other arms in that core as well, because arms are evaluated with the entire parent core as the subject.

## Using `|%` to Define Types

The `++` rune is used for most arm definitions in `|%`, but there are other runes for special kinds of arms.  

...[skipping this material for now, pending Hoon updates]

## Cores with Samples (Doors)

Cores without samples are often useful, but sometimes more is desired.  Sometimes you'll want to use a core with a sample.  Such cores are called *doors*.

One way they can be used is to produce different versions of a function.  There is a door in the Hoon standard library, `++  rs`, for producing single-precision float functions.  The various arms of this door (e.g., `add`, `sub`, `mul`) are designed for use with atoms whose aura is `@rs`:

```
> (add:rs .3.14 .2.72)
.5.86

> (sub:rs .3.14 .2.72)
.4.2000008e-1

> (mul:rs .3.14 .2.72)
.8.5408
```

But there are different versions of these gates for different rounding rules.  The default rule is to round to zero.  Accordingly, when `add` is evaluated with `rs` as the subject, a gate is produced that adds single-precision floats with that rounding rule assumed.

You can change the rounding rule by passing an argument to the `++  rs` core:

```
> ~(add rs %u)
<1.dfv {{a/@rs b/@rs} <21.vij {r/?($n $u $d $z) <54.tyv 119.wim 31.ohr 1.jmk $143>}>}>

> ~(add rs %d)
<1.dfv {{a/@rs b/@rs} <21.vij {r/?($n $u $d $z) <54.tyv 119.wim 31.ohr 1.jmk $143>}>}>
```

Each of these expressions produces a version of the `add` gate from `rs`.  By using `~( )` to pass `%u` to `rs` before producing `add`, you're creating an `add` gate for floats that rounds up.  By passing `%d`, you're producing an `add` gate for floats that rounds down:

```
> (~(add rs %u) .3.14159265 .1.11111111)
.4.252704

> (~(add rs %d) .3.14159265 .1.11111111)
.4.2527037
```

There are other uses for doors.  Often you'll want a door to serve as a 'state machine'.  That is, you'll want a core with data inside it representing various 'states' of the core.  This mutable core data is stored in the core's sample.  Usually the arms of such cores are designed to make use of the sample data, either by using the data to produce a value or by modifying the sample data.

Doors can be used a lot like objects in object-oriented programming languages.  The arms of doors are like computed attributes of objects.

### `|_` Produce a Door, i.e., a Core with a Sample

The `|_` rune is for producing a door.  Syntactically it's like the `|%` rune, but it takes an extra subexpression to define the sample.  The first subexpression following the `|_` defines the sample; then there is a series of arm definitions.  This series is terminated with the `--` rune.

For an example of a door 'in the wild', you can look at `++  rs` in the `hoon.hoon` standard library [here](https://github.com/urbit/arvo/blob/master/sys/hoon.hoon#L2752).

Let's look at a simpler door, one that can be used as a state machine:

```
|_  a/@
  ++  this  .
  ::
  ++  inc
    this(a +(a))
  ::
  ++  plus
    |=  b=@
    this(a (add a b))
  ::
  ++  reset
    this(a 0)
--
```

The door produced by this code has four arms and a sample `a` of the type `@`.

But what is each arm doing?  To put it briefly, the `inc` arm adds `1` to the atom in the sample, the `plus` arm adds `b` to the sample, and the `reset` arm resets the sample value to `0`.

But in order to make the door behave like a state machine, these arms can't simply evaluate to an atom.  Each of the arms above evaluates to a core.  Specifically, they each evaluate to a version of the parent core.

Remember, the subject of an arm is the core in which that arm is defined.  The `this` arm evaluates simply to `.`, i.e., the unmodified parent core.  `inc` evaluates to `this`, i.e., the parent core, but with the sample `a` modified to be `+(a)`.  `plus` evaluates to `this` except with the sample `a` modified to be `(add a b)`.  And `reset` evaluates to `this` but with the sample `a` set to `0`.

To use a door as a state machine, put the door in the subject and modify that door using its arms.  Let's do this in the dojo first, then we'll use a generator.  Set the dojo face `num` to the `|_` expression above.  You can paste the multi-line code above directly into the dojo if you like:

```
> =num |_  a/@
    ++  this  .
    ::
    ++  inc
      this(a +(a))
    ::
    ++  plus
      |=  b=@
      this(a (add a b))
    ::
    ++  reset
      this(a 0)
  --

> num
< 4.uzk
  { a/@
    { {rng/<4.wmp {a/@ud <54.tyv 119.wim 31.ohr 1.jmk $143>}> b/nlr(@) l/it(@)}
      our/@p
      now/@da
      eny/@uvJ
    }
    $~
    <16.cvy 19.rrs 41.gpy 112.lkm 224.nab 54.nhx 119.wim 31.ohr 1.jmk $143>
  }
>

> a.num
0

> =num inc.num

> a.num
1

> =num inc.num

> a.num
2

> =num (plus:num 100)

> a.num
102

> =num reset.num

> a.num
0
```

As you can see, you never actually 'change' the `num` core literally; you actually product a variant of that core and set that as the new value of `num`.  You can see the sample value of `num` using `a.num`.

Let's do the same sort of thing in a generator.  Save the following as `numcore.hoon` in the `/gen` directory of your urbit's pier:

```
|=  c=@
=/  num                              ::  define the `num` core
  |_  a/@
    ++  this  .
    ::
    ++  inc
      this(a +(a))
    ::
    ++  plus
      |=  b=@
      this(a (add a b))
    ::
    ++  reset
      this(a 0)
  --
=.  num  inc.num                     ::  increment the sample
=.  num  inc.num                     ::  do it again
=.  num  (plus:num c)                ::  add `c` to the sample
=.  num  (plus:num c)                ::  do it again
a.num                                ::  produce the sample value
```

Now run it in the dojo:

```
> +numcore 10
22

> +numcore 100
202

> +numcore 500
1.002
```

Each of the `=.` expressions 'modifies' `num` by replacing the old `num` core with a different version of it.  In this program we're 'modifying' `num` four times.

For more on how to use cores as state machines see the next lesson.
