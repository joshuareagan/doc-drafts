---
navhome: /docs/
sort: 5
title: Subject-oriented programming Pt. 2
---

# Hoon Tutorial 1.5: Subject-oriented Programming Pt. 2

In the previous lesson you learned about the subject, and how to return legs of the subject by various means.  But in addition to 'legs' the Hoon subject has 'arms'.  In this lesson you'll learn what an arm is.  But we can't fully understand arms without digging a little deeper.  We'll start with a brief digression and talk about Nock.

Hoon is a functional programming language, and as such it has functions.  Arms are critical for creating and evaluating functions in Hoon, so we're also going to build up to an explanation of what a Hoon function is.

This section is a little more challenging than previous sections.  Don't be discouraged.  You may need to read it more than once to understand everything being said.  That's okay.  The next lesson will expand on and illustrate this lesson.  So even if you don't fully understand this one the first time you work through it, it's perfectly reasonable to move on to the next lesson, and then circle back and reread this one if you still have questions.

## How Hoon Becomes a Computation

Hoon runs on your urbit, which is a virtual computer.  This virtual computer doesn't directly understand Hoon.  On the other hand, your urbit does include a Nock interpreter.  Every computation you want to make using Hoon code must first be compiled to Nock.  You certainly don't have to know Nock to understand Hoon, but a quick overview of some of its features will help.

### Nock

Nock is a [low-level](https://en.wikipedia.org/wiki/Low-level_programming_language), [functional programming](https://en.wikipedia.org/wiki/Functional_programming) language, and as such it can be understood as a function.

### What is a Function?

The word 'function' is used in various ways in other programming languages, but we mean it in basically its [mathematical sense](https://en.wikipedia.org/wiki/Function_(mathematics)).  Roughly put, a function takes one or more arguments (i.e., input values) and returns a value.  What the return value is depends solely on the argument(s), and nothing else.  For example, we could understand multiplication as a function: it takes two numbers and returns another number.  It doesn't matter where you ask, when you ask, or what kind of hat you're wearing when you ask.  If you pass the same two numbers (e.g., 3 and 4), you get the same answer returned every time (12).

### The Nock Function

The 'Nock function' takes two arguments: (1) a subject, and (2) a Nock formula.  The subject is a noun.  The formula is, loosely speaking, some instructions encoded as another noun.  The Nock function takes those two arguments and returns another noun called the 'product'.

The Nock function: `[subject formula] -> product`

Here's an example of some Nock run in the dojo, using `.*( )`:

```
> .*([15 17] [[4 0 2] [0 3]])
[16 17]
```

In this example Nock computation `[15 17]` is the subject, `[[4 0 2] [0 3]]` is the formula, and `[16 17]` is the product.  If you can't make sense of this bit of Nock, don't worry.  You don't need to know how to read Nock formulas to understand Hoon.  (Those who are curious about Nock anyway should head over to the Nock tutorial (link).)

### Nock Formulas Define Functions

It will be helpful to know one thing about Nock formulas, however.  As stated previously, a Nock formula is just a noun that represents a set of instructions.  When the Nock interpreter computes the Nock formula with a subject, the 'instructions' can manipulate and/or modify fragments of the subject, but the instructions have access to nothing else.  Accordingly, we may think of each Nock formula as defining a function with the subject itself as the argument.

Urbit community members have written Nock formulas for various functions: addition, multiplication, and more.  One obsessive individual took things a little too far and wrote a Nock formula that tests whether a given number is a counterexample to the [Goldbach conjecture](https://en.wikipedia.org/wiki/Goldbach%27s_conjecture).

Of course, programming interesting functions in Nock isn't especially practical.  That's why we have Hoon.

## Arms: An Incomplete Explanation

We're now ready for an initial, but incomplete, explanation of 'arms'.  An arm is a fragment of the subject that is a Nock formula.  But it's a lot more than just that.

### Arms Make Hoon Functions Possible

So far we haven't done much with Hoon functions.  But we have called two functions from the Hoon standard library: `add` and `mul`.

```
> (add 22 33)
55

> (mul 22 33)
726
```

Each Hoon function -- including not only predefined functions such as `add`, but also user-defined ones -- is implemented with an arm.  In order to make the function work, the arm must be evaluated.  This is one reason why arms are so important.  We'll build up to a more complete explanation of Hoon functions toward the end of this lesson.

### Arms Require an Organized Subject

Arms are Nock formulas.  But Nock formulas are primitive in many respects.  There is no basic Nock operation for looking up face values in the subject, for example.  So how do the arm 'instructions' find what they need in the subject when evaluated?

Up to this point in our dojo examples, we haven't organized the subject according to any particular arrangement.  The subject for most examples was so small that it wasn't worth the effort to bother with tidying it up.

On the other hand, if a subject becomes big enough to contain interesting functionality -- even having something as simple as a standard library of functions, as Hoon has -- organization of the subject becomes a pressing practical matter.  It's much better if the subject can be assumed to have a specific, standard structure.  Nock formulas that are created under that assumption can be more simple than would otherwise be the case.  Fortunately Hoon implements just this kind of subject organization.

With that in mind, we're ready to expand a little on the definition of an arm: an arm isn't just any Nock formula.  It's a formula that expects the subject to be structured as a *core*.  

## Cores

A solid grasp of cores is important for understanding Hoon semantics.  Fortunately, they aren't particularly complicated.  A core is a cell of a *battery* and a *payload*.

```
A core:  [battery payload]
```

Very simple, except we don't yet know anything about either batteries or payloads!  To put it simply, the battery is code and the payload is data.

### Battery

A battery is one or more arm(s).  That is, a battery is one or more Nock formula(s) that, as they are evaluated, expect the subject to be ordered as a core.  If there are multiple arms they are arranged in a cell as follows:

```
[arm-1 arm-2 arm-3 ...]
```

### Payload

As we said earlier, the payload is data.  In principle, the payload of a core can have data arranged in any arbitrary configuration.  In practice, the payload usually has a predictable structure of its own.  We'll say a little more about payloads shortly.

## Arms Explained Further

We're now ready to give a nearly-complete definition of an 'arm'.  We can't yet give a definitive description, but we're getting close.

An arm is (1) a Nock formula in the head of a core (i.e., in the battery), and it's (2) designed to be evaluated with the core it's contained in as the subject.  Let's call the core each arm is contained in its *parent core*.

### One More Quirk

There is still one thing that isn't quite clear yet.  It concerns whether the arm's subject must be *exactly* its parent core, or whether the arm may be evaluated with a modified version of the parent core as the subject.  For example, let's say arm *A* is in the battery of core *C*.  The question here is: must *A* be evaluated with exactly *C* as the subject, or should variations of *C* be allowed too?

You might be wondering why this matters.  Arms are Nock formulas, and Nock formulas are functions that take a subject as the argument.  Let's assume that each arm must take *exactly* its parent core as the subject, and hence as its 'argument'.  If we evaluate an arm with exactly its parent core as the subject, it'll return the same product every time.  That's how functions work.  If you pass the same argument(s) to a function, you always get the same result.

We said earlier that all Hoon functions depend on arms.  But we don't want to be limited to just one argument for each Hoon function.  We want Hoon functions to take various possible input values and return the appropriate value in each case.

```
> (add 22 33)
55

> (add 23 33)
56
```

There is a single arm that implements the `add` function.  How is it that we're able to use arms to implement functions such as `add`, which can give different answers when we pass different values to them?  

## Anatomy of a Hoon Function

The solution is to allow the arm to be evaluated with a modified version of its parent core as the subject.  Variations are permitted in a specific part of the payload called the *sample*.  The change in the payload is for storing the arguments passed to the Hoon function (e.g., `22` and `33` for `add`).  The arm in question therefore has access to those arguments.

### Gates: Cores that Define Hoon Functions

Hoon functions such as `add` are defined by a special kind of core called a *gate*.  A gate is (1) a core whose battery contains only one arm, and (2) whose payload is a cell of a *sample* and a *context*.  

```
A gate:  [arm [sample context]]
```

The sample is for storing the argument(s) passed to the Hoon function, and the context is for other data needed by the function.

You can look at a gate yourself by entering its name into the dojo:

```
> add
<1.vng {{a/@ b/@} <31.ohr 1.jmk $143>}>
```

So the gate is `<1.vng {{a/@ b/@} <31.ohr 1.jmk $143>}>`.  But what does this all mean?  We'll take it one step at a time.

#### The Gate Arm

The arm of a gate specifies (in Nock) the instructions for the Hoon function in question.  The gate's arm is always located at `+2` of the gate.  But if we look at `+2` of the `add` gate above, we see `1.vng`.  What does this mean?  It's the Hoon pretty-printer's way of letting you know that there is a battery with `1` arm in it, without actually showing you the noun directly.  The `vng` is a [hash](https://en.wikipedia.org/wiki/Hash_function) of the battery noun.

To see the actual noun of the arm for `add`, enter `+2:add` into the dojo:

```
> +2:add
[ 6
  [5 [1 0] 0 12]
  [0 13]
  9
  2
  [0 2]
  [[8 [9 2.540 0 7] 9 2 [0 4] [0 28] 0 11] 4 0 13]
  0
  7
]
```

Unless you know Nock this won't mean much to you.  Still, for what it's worth, these are the instructions for adding two numbers in Nock.

It's worth pointing out that the arm of a gate has a special name: `$`.  For reasons we'll explain in the next lesson, using this name in Hoon not only finds the arm in the subject; it also evaluates that arm with its parent core as the subject.  We can evaluate `add`s arm with `$:add` in the dojo:

```
> $:add
0
```

This result may seem a bit strange.  We didn't call `add` or in any other way pass it numbers to add together.  Yet using `$` to evaluate `add`'s arm seems to work -- sort of, anyway.  Why is it giving us `0` as the return value?  We can answer this question after we understand gate samples a little better.

#### The Sample

The sample of a gate is the address reserved for storing the argument(s) to the Hoon function.  The sample is always at the head of the gate's tail, i.e., `+6`.

Let's look at the gate for `add` again, paying particular attention to its sample:

```
> add
<1.vng {{a/@ b/@} <31.ohr 1.jmk $143>}>
```

We see `{a/@ b/@}`.  This probably isn't totally clear, but it should make *some* sense.  This is the pretty-printer's way of indicating a pair of atoms with the faces `a` and `b`.  Calling `add` gives us the sum of two atoms, after all.  Let's take a closer look:

```
> +6:add
[a=0 b=0]

> ^-(* +6:add)
[0 0]
```

We see now that the sample of `add` is a cell with two atoms, each `0`.  These are placeholder values for the function arguments.  If you evaluate the arm of `add` without passing it any arguments the placeholder values will be used, and the return will be sum of `0 + 0`:

```
> $:add
0

> (add)
0
```

When you call a function with arguments, the default value of the sample is replaced with the value of those arguments.  The arm is then evaluated with this modified version of its parent core as the subject; the sample of that core is the part that has changed.

```
> (add 21 33)
54

> (add 22 33)
55

> (add 23 33)
56
```

What are the faces for in the sample, `a` and `b`?

```
> +6:add
[a=0 b=0]
```

These faces are used in the Hoon code defining the `add` gate.  The adventurous are invited to look at this code in `hoon.hoon` ([here](https://github.com/urbit/arvo/blob/master/sys/hoon.hoon)).  The code defining the `add` gate is near the top of this file.  (Yes, the Hoon programming language is implemented in Hoon.)

#### The Context

The context of a gate contains other data that may be necessary for the arm to evaluate correctly.  The context always located at the tail of the tail of the gate, i.e., `+7` of the gate.

There is no requirement that the context have any particular structure.  Often, however, the context contains another core.  Let's look at `add`'s core once more:

```
> add
<1.vng {{a/@ b/@} <31.ohr 1.jmk $143>}>
```

The context of `add` is `<31.ohr 1.jmk $143>`.  This is, in fact, another core.  The pretty-printer uses the `< >` characters to indicate a core.  I won't explain everything you see here, but you can see that the head of this core is a battery with 31 arms.

Notice that you can (and usually do) modify the sample when you call a gate, but there is no way to modify the context.

```
> (add 12 14)
26

> (mul 12 14)
168
```

We'll have more to say about contexts in the next lesson.

## Arms: A Definition, Finally

An arm is (1) a Nock formula in the battery of a core, and it's (2) designed to be evaluated with a subject of either (a) its parent core or (b) a core that's just like its parent core except with a replaced sample.

And now you have a definition of 'arm'!

### Gates Define Functions of the Sample

In an ordinary Hoon gate call, neither the arm nor the context is modified before the arm is evaluated.  That means that the only part of the gate that changes before the arm evaluation is the sample.  Hence, we may understand each gate as defining a function whose argument is the sample.  If you call a gate with the same sample, you'll get the same value returned to you every time.

In the next lesson we'll learn more about how limb and wing expressions resolve to arms, and how this makes makes Hoon function calls possible.  We'll also deepen our understanding of cores.
