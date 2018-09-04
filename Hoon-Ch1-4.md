---
navhome: /docs/tutorials/ch1-hoon/
sort: 4
title: Arms and Cores
---

# 1.4 Arms and Cores

The Hoon subject is made up of arms and legs.  In the previous lesson you learned what a leg is, and how to return legs of the subject by various means.  In this lesson you'll learn what an arm is.  Arms are a bit more complex than legs -- a full understanding of them requires a bit more background knowledge.  Accordingly, in this lesson we also (1) give a brief overview on Nock, a low-level functional programming language to which Hoon is compiled, and (2) define an important Hoon data structure: 'cores'.

After arms are defined and explained we review wing expressions, this time focusing on wing resolution to arms rather than legs.

## Arms: A Preliminary Explanation

Arms and legs are limbs of the subject, i.e., noun fragments of the subject.  In the last lesson we said that legs are for data and arms are for computations.  But what _specifically_ is an arm, and how is it used for computation?

Let's begin with a preliminary definition that will be refined later in the lesson.  An *arm* is a Nock formula in the battery of a core.

Now you just need to know what a 'Nock formula' is, what a 'battery' is, and what a 'core' is!  Read on:

## Computing Hoon

Hoon runs on your urbit, which is a virtual computer.  However, this virtual computer doesn't directly understand Hoon; it has a Nock interpreter instead.  Any Hoon code you want to run must first be compiled to Nock.  You certainly don't have to know Nock to understand Hoon, but it helps to be aware of a few of Nock's general features.

Nock is a [low-level](https://en.wikipedia.org/wiki/Low-level_programming_language), [functional programming](https://en.wikipedia.org/wiki/Functional_programming) language.  As a consequence Nock itself can be understood as a function, called the 'Nock function'.  Nock is officially defined by the Nock spec (link).

### What is a Function?

The word 'function' is used in various ways in other programming languages, but we mean it in basically its [mathematical sense](https://en.wikipedia.org/wiki/Function_(mathematics)).  Roughly put, a function takes one or more arguments (i.e., input values) and returns a value.  What the return value is depends solely on the argument(s), and nothing else.  For example, we can understand multiplication as a function: it takes two numbers and returns another number.  It doesn't matter where you ask, when you ask, or what kind of hat you're wearing when you ask.  If you pass the same two numbers (e.g., `3` and `4`), you get the same answer returned every time (`12`).

### The Nock Function

The 'Nock function' takes two arguments: (1) a subject, and (2) a Nock formula.  The subject is a noun.  The formula is, loosely speaking, some instructions encoded as another noun.  The Nock function takes those two arguments and returns another noun called the 'product'.

The Nock function: `[subject formula] -> product`

Here's an example of some Nock run in the dojo, using `.*( )`:

```
> .*([15 17] [[4 0 2] [0 3]])
[16 17]
```

In this example `[15 17]` is the subject, `[[4 0 2] [0 3]]` is the formula, and `[16 17]` is the product.

This tutorial series is designed to teach you Hoon.  For that purpose it's not important that you know how to read and understand Nock formulas; you just need to know that Nock takes a subject and a formula and returns a product.  (Those who who want to learn more about Nock should head over to the Nock tutorial (link).)

### Nock Formulas Define Functions

Each Nock formula defines a function of the subject.

As stated previously, a Nock formula is just a noun that represents a set of instructions.  When the Nock interpreter computes the Nock formula against a subject, the 'instructions' can manipulate and/or modify fragments of the subject, but the instructions have access to no other data.  Accordingly, we may think of each Nock formula as defining a function with the subject itself as the argument.

Urbit community members have written Nock formulas for various functions: addition, multiplication, and more.  One obsessive individual went overboard and wrote a Nock formula that tests whether a given number is a counterexample to the [Goldbach conjecture](https://en.wikipedia.org/wiki/Goldbach%27s_conjecture).

Keep in mind that Nock is a low-level language.  Programming interesting functions in it isn't especially practical.  That's why you have Hoon.

### Arms are Nock formulas

An arm is a _certain kind_ of Nock formula in the battery of a core.  Each arm expects the subject to have a certain arrangement so that it can find the data it needs to operate correctly.  (This point will be discussed in more detail shortly.)  The subject must be a core.  What's a core?

## Cores

A *core* is a cell of a *battery* and a *payload*.

```
A core:  [battery payload]
```

Simply put, the battery is code and the payload is data.  The payload contains all data necessary for running the code in the battery correctly.

### Battery

A battery is a cell containing one or more arm(s).  That is, a battery is one or more Nock formula(s) that, when computed, expect the subject to be organized as a core.  For Hoon programming it's not important to know just how those arms are arranged in the battery.  Just keep in mind that this is where the core's computations live.

### Payload

As stated earlier, the payload is data.  In principle, the payload of a core can have data arranged in any arbitrary configuration.  In practice, the payload often has a predictable structure of its own.

The payload of a core *C* contains whatever data is necessary to run the arms of *C* correctly.  This data may include other cores.  Consequently, *C*'s payload data can include other 'code' -- cores in the payload have their own arms.

### Each Arm Computes with its Parent Core as the Subject

Arms are Nock formulas.  But Nock formulas are primitive in many respects.  There is no basic Nock operation for looking up face values in the subject, for example.  So how do arm 'instructions' find what they need in the subject?  The answer is: by assuming that the subject is organized a particular way.

#### Subject Organization for Arm Computation

Up to this point in our dojo examples we haven't organized the subject according to any intentional pattern.  The subject for most examples was so small that it wasn't worth the effort to bother with tidying it up.

However, if a subject is sufficiently large and complex -- e.g., it has a standard library of Hoon functions, as the dojo subject does -- then organization becomes a pressing practical matter.  It's much better if the subject can be assumed to have a specific, standard arrangement.  This is one reason Hoon enforces subject organization for arm computations.

The subject must be organized as a core.  But not just any core!  For an arm to compute correctly, the subject must be the arm's *parent core*.

#### The Parent Core of an Arm

Each arm is contained in the battery of a core.  That core is the parent core of the arm.  In other words: the *parent core* of an arm *A* is the core whose battery contains *A*.

Why must an arm have its parent core as the subject, when it's computed?  As stated previously, the payload of a core contains all the data needed for computing the arms of that core.  Remember that arms can only access data in the subject.  By requiring that the parent core be the subject we guarantee that each arm has the appropriate data available to it, in the payload.

An arm can also make use of the other arms in its parent core.  This is a useful feature of arms that will be discussed in more detail later.

## Final Definition of an Arm

Now that you have the relevant background knowledge about cores, we can finally give a complete definition of what an arm is.

An arm is a Nock formula in the battery of a core, and it's to be computed with its parent core as the subject.

## Wings that Resolve to Arms

To reinforce your newfound understanding of arms and cores, let's go over various dojo examples illustrating their properties.  These examples use and illustrate how wings resolve to arms.  Face resolution to a leg of the subject produces the leg value unchanged, but resolution to an arm produces the _computed_ value of that arm.

To show how this works, we'll look at how the various wing expressions behave when they resolve to arms instead of legs.

### Building a Core of Your Own

Before getting to the arm examples we need to produce a simple core.  We'll do this using a multi-line Hoon expression that is more complex than the other examples you've seen up to this point.  The dojo can be used to input multi-line Hoon expressions; just type each line, hitting 'enter' or 'return' at the end.  The expression will be evaluated at the appropriate line break, i.e., when the dojo recognizes it as complete.

First let's use the dojo to bind the face `a` to the value `12`, and the face `b` to the value `[22 24]`.  These values will be used in the arms of the core we're going to make.  Afterward use `.` to see the value of the subject:

```
> =a 12

> =b [22 24]

> .
[ [ [a=12 b=[22 24]]
    our=~bitnus-holleb-rocmeg-fotfer--migsum-nidsym-hilput-patmus
    now=~2018.8.17..23.53.43..c6b5
      eny
    \/0vqg.quub6.1vjms.es43r.rogsb.91okf.bcvtp.eeoih.or9ge.18ere.jtk15.3afb4.5d\/
      qp2.f7cqj.tj4bh.m7ja9.r1i9m.5scdh.ih7qa.dqm18.q5b4r
    \/                                                                         \/
  ]
  ~
  <16.igq 19.rrs 41.xjd 112.hmp 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
]
```

You can see that `a` and `b` have the desired values in the dojo subject.

#### Using `|%`

Now use the following multi-line expression to bind `c` to a core, then evaluate `c` so you can look at the product.  Take note of the expression spacing -- Hoon uses significant whitespace.  Feel free to cut and paste the following expression into the dojo:

```
> =c |%
  ++  two  2
  ++  inc  (add 1 a)
  ++  double  (mul 2 a)
  ++  sum  (add -.b +.b)
  --

> c
< 4.oep
  { {{a/@ud b/{@ud @ud}} our/@p now/@da eny/@uvJ}
    $~
    <16.igq 19.rrs 41.xjd 112.hmp 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>
```

Note: binding `c` to the core created by the `|%` expression will fail unless you have already bound `a` and `b` to the relevant values, as we did above.  Our having already bound them is what lets us use them in that expression.

#### Explaining `|%` Expressions

We'll cover Hoon syntax thoroughly in lesson 2.2.  For now, take for granted that `|%` is a 'rune' used to produce a core.  The `++` runes are used to define the arms of the core's battery, and is followed by both an arm name and a computation.  The `--` rune is used to indicate that there are no more arms to be defined, and indicates the end of the expression.

The `|%` expression above creates a core with four arms.  The first, named `two`, evaluates to the constant `2`.  The second arm, `inc`, adds `1` to `a`.  `double` returns double the value of `a`, and `sum` returns the sum of the two atoms in `b`.  The computations defined by these arms are pretty simple (and in the case of `two`, trivial), but a good starting point for learning about cores.

#### Payload and Subject

That's enough to specify the battery of the core.  How is the payload defined?  The payload of a core produced by a `|%` expression is the subject of that expression.  Look at the subject before `c` was bound to the core value, produced by `.` above, and compare that with the value of `c` once we bound it to the core.  `c`, like all cores, is a cell of battery and payload and it's pretty printed inside a set of angled brackets, `< >`.  The battery of four arms is represented by the pretty-printer as `4.oep`.  The `4` represents the number of arms in the battery, and the `oep` is a [hash](https://en.wikipedia.org/wiki/Hash_function) of the battery.  The tail is the exact same as the subject of the `|%` expression (but it's pretty-printed in a slightly different way).

(You'll notice that there are several other cores in the payload.  For example, `16.igq` is a battery of `16` arms, `19.rrs` is a battery of `19` arms, etc.)

### Address-Based Wings

In the last lesson, you saw how the following expressions return legs based on an address in the subject: `+n`, `.`, `-`, `+`, `+>`, `+<`, `->`, `-<`, `&`, `|` etc.  When these resolve to the part of the subject containing an arm, they *don't* evaluate the arm.  They simply return that fragment of the subject, as if it were a leg.

Let's use `-.c` to look at the head of `c`, i.e., the battery of the core:

```
> -.c
[ [1 2]
  [ [8 [9 318 0 2.047] 9 2 [0 4] [[7 [0 3] 1 2] 0 56] 0 11]
    8
    [9 8 0 2.047]
    9
    2
    [0 4]
    [[7 [0 3] 1 1] 0 56]
    0
    11
  ]
  8
  [9 8 0 2.047]
  9
  2
  [0 4]
  [0 57]
  0
  11
]
```

The results is a series of uncomputed Nock formulas.  (Don't worry about what they mean -- you don't need to know Nock to become a Hoon expert.)  Let's use `-<.c` to look at the head of the head of `c`, i.e., the first arm:

```
> -<.c
[1 2]
```

The wing `-<.c` correctly resolves to an arm, but returns the uncomputed Nock formula.

### Name-based Wings

To get the arm of a core to compute you must use its name.  The arm names of `c` are in the expression used to create `c`:

```
> =c |%
  ++  two  2
  ++  inc  (add 1 a)
  ++  double  (mul 2 a)
  ++  sum  (add -.b +.b)
  --
```

Remember that `a` is `12` and `b` is `[22 24]`.  Let's evaluate some arms:

```
> two.c
2

> inc.c
13

> double.c
24

> sum.c
46
```

You can also evaluate the arms of `c` as follows:

```
> two:c
2

> inc:c
13

> double:c
24

> sum:c
46
```

The difference between `two.c` and `two:c` is as follows.  `two.c` is an instruction for finding `two` in `c` in the subject.  `two:c` is an instruction for setting `c` as the subject, and then finding for `two`.  In the examples above both versions amount to the same thing, and so return the same product.

What if we unbind `a` and `b` in the dojo subject?  Will the arms of `c` still run correctly?  Yes.

```
> =a

> =b

> a
-find.a

> inc.c
13

> sum.c
46

> a.c
12

> b.c
[22 24]
```

Notice that `a.c` and `b.c` still evaluate to the values we originally set for `a` and `b`.  That's because when we defined `c` those `a` and `b` were stored as part of `c`'s payload.  The payload stores all the data needed to compute the arms correctly.  That also includes `add` and `mul`, which themselves are arms in a core of the Hoon standard library.

### Name Searches and Collisions

It's possible to have 'name collisions' with faces and arm names.  Nothing prevents one from using the name of some arm as a face too.  For example:

```
> double:c
24

> double:[double=123 c]
123
```

When `[double=123 c]` is the subject, the result is a cell of: (1) `double=123` and (2) the core `c`:

```
> [double=123 c]
[ double=123
  < 4.oep
    { {{a/@ud b/{@ud @ud}} our/@p now/@da eny/@uvJ}
      $~
      <16.igq 19.rrs 41.xjd 112.hmp 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
    }
  >
]
```

Hoon doesn't automatically know whether `double` is a face or an arm name until it conducts a search looking for name matches.  If it finds a face first, the value of the face is returned.  If it finds an arm first, the arm will be evaluated and the product returned.  You may use `^` to indicate that you want to skip the first match, and multiple `^`s to indicate multiple skips:

```
> double:[double=123 c]
123

> ^double:[double=123 c]
24

> ^double:[double=123 double=456 c]
456

> ^^double:[double=123 double=456 c]
24
```

### Modifying a Core

We can produce a modified version of the core `c` in which `a` and `b` have different values.  A core is just a noun in the subject, so we can modify it in the way we learned to modify legs in the last lesson.  To change `a` to `99`, use `c(a 99)`:

```
> c(a 99)
< 4.oep
  { {{a/@ud b/{@ud @ud}} our/@p now/@da eny/@uvJ}
    $~
    <16.igq 19.rrs 41.xjd 112.hmp 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>

> a.c
12
```

The expression `c(a 99)` produces a core exactly like `c` except that the value of `a` in the payload is `99` instead of `12`.  But when we evaluate `a.c` we still get the original value, `12`.  Why?  The value of `c` in the dojo is bound to the original core value, and will stay that way until we unbind `c` or bind it to something else.  We can ask for a modified copy of `c` but that value doesn't automatically persist.  It must be put into the subject if we're to find it there.  So how do we know that `c(a 99)` successfully modified the value of `a`?  We can check by setting the new version of the core as the subject and checking `a`:

```
> a:c(a 99)
99

> double:c(a 99)
198
```

To make the modified core persist as `c`, we can rebind `c` to the new value:

```
> =c c(a 99)

> a:c
99

> double:c
198
```

We can make multiple changes to `c` at once:

```
> =c c(a 123, b [44 55])

> a:c
123

> b:c
[44 55]

> two:c
2

> inc:c
124

> double:c
246

> sum:c
99
```

### Arms on the Search Path

A wing is a search path into the subject.  We've looked at some examples of wings that resolve to arms; e.g., `double.c`, which resolves to `double` in `c` in the subject.  In the latter example the arm `double` is the final limb in the search path.  What if an arm name in a wing isn't the final limb?  What if it's elsewhere on the search path?

Normally we might read a wing expression like `two.double.c` as '`two` in `double` in `c`'.  Does that make sense when `double` is itself an arm?  Try it:

```
> two.double.c
2
```

It produces the value of `two` of `c`.  This is a fact in need of explanation.

Arms are raw Nock formulas, and there isn't much reason to follow a search path into those.  There are no faces or other names in arm Nock formulas!  For this reason, when arm names are included in the search path the search behavior is a little different.  Instead of indicating that the search should continue in the arm itself, an arm name indicates that the search should continue in the parent core of the arm.

So the meaning of `two.double.c` is, roughly, '`two` in the parent core of `double` in `c`'.  Of course, Hoon doesn't know that `double` is an arm until the search for it ends; but once the `double` arm is found, Hoon continues the search from _the parent core of_ `double`, not `double` itself.  It turns out in this case that this is a redundant step.  `c` is the parent core and was already on the search path.  We can illustrate redundancy more dramatically:

```
> double.two.sum.two.double.inc.c
246

> two.double.two.sum.two.double.inc.c
2

> sum.two.double.two.sum.two.double.inc.c
99
```

In each of the following examples, the only wings that matter are `c` and whichever arm name is left-most in the expression.  The other arm names in the path simply resolve to their parent core, which is just `c`.

### Evaluating an Arm Against a Modified Core

Assume `this.is.a.wing` is a wing that resolves to an arm.  You can use the `this.is.a.wing(face new-value)` syntax to compute the arm against a modified version of the parent core of `this.is.a.wing`.

```
> double.c(a 55)
110

> inc.c(a 55)
56
```

## Summary

At this point you should have a pretty good understanding of what an arm is, and how wing resolution to an arm works.  But so far the arms you've created have been fairly simple.  In the next lesson you'll learn about Hoon functions, and how to create your own.

You can now unbind `c` in the dojo -- this will help to keep your dojo subject tidy:

```
> =c

> c
-find.c
```
