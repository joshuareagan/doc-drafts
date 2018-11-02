# Polymorphism

Hoon supports type [polymorphism](https://en.wikipedia.org/wiki/Polymorphism_(computer_science)) at the core level.  That is, each core has certain polymorphic properties which are tracked and maintained as metadata by Hoon's type system.  They determine the extent to which a core can be used as an interface for values of various types.  ("Polymorphism" = "many forms", so "type polymorphism" is about dealing with a variety of data types.)

This is rather vaguely put so far.  We can clarify by talking more specifically about the different kinds of polymorphism supported in Hoon.  There are two kinds of type polymorphism for cores: [variance](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)) and [genericity](https://en.wikipedia.org/wiki/Generic_programming).

Correspondingly, there are two broad categories of cores, one for each kind of polymorphism: wet and dry.  *Dry* cores use variance and *wet* cores use genericity.  (Note to writer: there can also be wet cores with different variance values, so this is misleading.)

## Dry Cores

Before discussing variance, we should first talk about the typical kinds of type-checking that occurs for dry cores.  Understanding the latter will help us to discuss the former topic more clearly.  There are two kinds of type check that occur for dry cores, each with a distinct set of rules: (i) type checks associated with arm evaluation, intended to verify that the arm's parent core has a payload of the appropriate type; and (ii) the type checks that determine whether one core type nests under another.  Let's discuss (i) first.

### Arm Evaluation

To review briefly, a *core* is a pair of `[battery payload]`.  The *battery* contains the various arms of the core (code), and the *payload* contains all the data needed to evaluate those arms correctly (data).  A *gate* (i.e., a Hoon function) is a core with exactly one arm named `$` and with a payload of the form: `[sample context]`.  The *sample* is reserved for the function's argument value, and the *context* is any other data needed to evaluate `$` correctly.

When an arm is evaluated it takes its parent core as the subject.  But often certain values in the payload of a core are modified before the evaluation occurs.  In the most typical case, a dry gate's default sample is replaced with some argument value before the gate's `$` arm is computed.  But this change is intended to be conservative; the new sample value must be of the same type as the default sample value (or possibly a subtype).  Thus when the `$` arm is evaluated, it should always have a subject of the same type.

For example, consider the gate created by `add`, a function in the Hoon standard library.  The structure of the `add` gate is as follows:

```
       add
      /   \
     $     .
          / \
    sample   context
```

The `add` code depends on the fact that there are two atoms in the sample, the numbers to be added together.  Hence, it is appropriate that only a cell of atoms is permitted for the sample.  We can look at the default sample of `add` at address `+6`:

```
> +6:add
[a=0 b=0]
```

If we call `add` with no arguments, we get the sum of these two values:

```
> (add)
0
```

Any other type of value in the sample results in a `nest-fail` crash:

```
> (add 12 "hello")
nest-fail
```

This function call creates a modified copy of the `add` gate, replacing the original sample `[0 0]` with `[12 "hello"]`.  (We ignore the `a` and `b` faces for now.)  But evaluating the `$` arm of this modified core didn't work; the type system knew to reject it.

How does it know?  The type system keeps track of two payload types for each core: (i) the original payload type, i.e., the type of the payload when the core was first created; and (ii) the current payload type, i.e., the type of the payload after any substitutions or other changes have been made.

The type check at compile time compares these two types, and if the current payload type doesn't nest under the original payload, the compile fails with a `nest-fail` crash.  If the former nests under the latter, the original payload type is used for evaluation.  So the arm is guaranteed to run with exactly the same subject type each time it is evaluated. 

The type check for arm evaluation can be understood as implementing a version of the ["Liskov substitution principle"](https://en.wikipedia.org/wiki/Liskov_substitution_principle).  The arm works for some (originally defined) payload type `P`.  Payload type `Q` nests under `P`.  Therefore the arm works for `Q` as well, though for evaluation the payload is treated as a `P`.

### When One Core Type Nests Under Another

The other sort of type check relevant for dry core polymorphism involves comparing two distinct cores.  When does one core type nest under another?

A couple of example use-cases might help motivate the question from a practical perspective.  Let's say you want to pass a core as an argument to a function, or that you're writing a function whose product is to be a core.  In the former case a type-check on the core is done for the relevant arm evaluation.  In the latter case, you should include an explicit cast for the sort of core you want to produce.  Either way, it's good to have a basic understanding of the core nesting rules.

The battery nesting rules for dry cores are relatively straightforward.  Let `A` and `B` be two cores.  The type of `B` nests under the type of `A` only if two conditions are met: (i) the batteries of `A` and `B` have exactly the same tree shape, and (ii) the product type of each `B` arm nests under the product type of the corresponding `A` arm.  These battery nesting rules hold for all four kinds of cores described in the section below.

#### Variance

The payload types of `A` and `B` must also be checked.  Certain payload nesting rules apply to all dry cores, if `B` is to nest under `A`: (i) if `A` is dry then `B` must also be dry; (ii) the original payload type of `A` must mutually nest with the current payload type of `A` (i.e., they nest under each other); and (iii) the current payload type of `B` must nest under the original payload type of `B`.

But there are different sets of nesting rules for finishing the payload type check, depending on the polymorphic properties of the payload.  The *variance* of a core determines which of these properties is present.

There are four kinds of dry core: *gold* (invariant), *iron* (contravariant), *zinc* (covariant), and *lead* (bivariant).   Variance information about each core is tracked by the Hoon type system.

Let's go through each kind more carefully.

#### Gold Cores (Invariant)

By default, most cores are gold.  This includes the cores produced with the `|%`, `|=`, `|_`, and `|-` runes, among others.

Assume that `A` is a gold core and that `B` is some other core.  The type of `B` nests under the type of `A` only if the original payload types of each mutually nest -- i.e., the original payload type of `A` nests under the original payload type of `B`, and _vice versa_.  Furthermore, only gold cores nest under other gold cores; so `B` must be gold to nest under `A`.

These payload nesting rules are the strictest ones for cores.  The payload type is neither covariant nor contravariant; when type-checking against a gold core, the target payload cannot vary at all in type.  Consequently, it is type-safe to modify every part of the payload: gold cores have a read-write payload.

Usually it makes sense to cast for a gold core type when you're treating a core as a state machine.  The check ensures that the payload, which includes the relevant state, doesn't vary in type.  To see state machines that involve casts for gold cores, see Chapter 2 of the Hoon tutorial.

Let's look at simpler examples here, using the `^+` rune:

```
> ^+(|=(^ 15) |=(^ 16))
< 1.xqz
  { {* *}
    {our/@p now/@da eny/@uvJ}
    <19.rga 23.byz 5.rja 36.apb 119.ikf 238.utu 51.mcd 93.glm 74.dbd 1.qct $141>
  }
>

> ^+(|=(^ 15) |=([@ @] 16))
nest-fail

> ^+(|=(^ 15) |=(* 16))
nest-fail
```

The first cast goes through because the target gold core (the one produced by the second `^+` subexpression) has the same sample type as the cast example core (produced by the first subexpression).  The sample types mutually nest.  The second cast fails because the target sample type is more specific than the cast sample type.  (Not all cells, `^`, are pairs of atoms, `[@ @]`.)  And the third cast fails because the target sample type is broader than the cast sample type. (Not all nouns, `*`, are cells, `^`.)

Two more examples:

```
> ^+(=>([1 2] |=(@ 15)) =>([123 456] |=(@ 16)))
<1.xqz {@ @ud @ud}>

> ^+(=>([1 2] |=(@ 15)) =>([123 456 789] |=(@ 16)))
nest-fail
```

In these examples, the `=>` rune is used to give each core a simple context.  The context of the first core in each case is a pair of atoms, `[@ @]`.  The first cast goes through because the target core also has a pair of atoms as its context.  The second cast fails because the target core has the wrong type of context -- three atoms, `[@ @ @]`.
 
#### Iron Cores (Contravariant)

Assume that `A` is an iron core and `B` is some other core.  The type of `B` nests under the type of `A` only if the original sample type of `A` nests under the original sample type of `B`.  That is, the target core sample type must be equivalent or a superset of the cast core sample type.  Furthermore, `B` must be either iron or gold to nest under `A`.

The context type of `B` isn't checked at all; and because of the nesting rules it isn't type-safe to do type-inference on the sample of an iron gate.  Thus, there are certain restrictions on iron cores that preserve type safety: (i) the context of an iron core is opaque, meaning that it cannot be written-to or read-from; and (ii) the iron core payload is write-only.

Iron gates are particularly useful when you want to pass gates (having various payload types) to other gates.  We can illustrate this use with a very simple example.  Save the following as `gatepass.hoon` in the `gen` directory of your urbit's pier:

```
|=  a=_^|(|=(@ 15))
^-  @
=/  b=@  (a 10)
(add b 20)
```

This generator is mostly very simple.  Ignoring the first line for a moment... the second line defines the gate output with a cast for an atom, `@`.  The third line includes a function call, passing `10` to `a`, and that value is stored in the subject as an atom, `@`, with the face `b`.  The third line adds `20` to `b`, and that product is the output value of the gate.

The first line is defines the gate sample as an iron gate, and gives it the face `a`.  So the function as a whole is for taking some gate, calling it by passing it the value `10`, adding `20` to it, and returning the result.  Let's try it out in the dojo:

```
> +gatepass |=(a=@ +(a))
31

> +gatepass |=(a=@ (add 3 a))
33

> +gatepass |=(a=@ (mul 3 a))
50
```

But we still haven't fully explained the first line of the code.  In particular, what does `_^|(|=(@ 15))` mean?  The inside is clear enough -- `|=(@ 15)` produces a normal (i.e., gold) gate that takes an atom and returns `15`.  The `^|` rune is used to turn gold gates to iron.  (Reverse alchemy!)  And the `_` character turns that iron gate value into a sturcture, i.e. a type.  So the whole subexpression means, roughly: "the same type as an iron gate whose sample is an atom, `@`, and whose product is an atom".  The context isn't checked.  This is good, because that allows us to accept gates defined in drastically different environments.  Let's try passing a gate with a different context:

```
> +gatepass =>([22 33] |=(a=@ +(a)))
31
```

Yup, it still works.  You can't do that with a gold core sample!

There's a simpler way to define an iron sample.  Revise the first line of `gatepass.hoon` to the following:

```
|=  a=$-(@ @)
^-  @
=/  b=@  (a 10)
(add b 20)
```

If you test it you'll find that it behaves the same as it did before the edits.  The `$-` rune is used to create an iron gate structure, i.e., an iron gate type.  The first expression defines the sample type of the iron gate, and the second subexpression defines the gate output type.

The sample type of an iron gate is contravariant.  This means that, when doing a cast with some iron gate, the target gate must have either the same sample type or a superset.  Let's illustrate:

```
> ^+(^|(|=(^ 15)) |=(^ 16))
< 1|xqz
  { {* *}
    {our/@p now/@da eny/@uvJ}
    <19.rga 23.byz 5.rja 36.apb 119.ikf 238.utu 51.mcd 93.glm 74.dbd 1.qct $141>
  }
>

> ^+(^|(|=(^ 15)) |=([@ @] 16))
nest-fail

 ^+(^|(|=(^ 15)) |=(* 16))
< 1|xqz
  { {* *}
    {our/@p now/@da eny/@uvJ}
    <19.rga 23.byz 5.rja 36.apb 119.ikf 238.utu 51.mcd 93.glm 74.dbd 1.qct $141>
  }
>
```

(As before, we use the `^|` rune to turn gold gates iron.)

The first cast goes through because the two gates have the same sample type.  The second cast fails because the target gate has a more specific sample type than the cast gate does.  If you're casting for a gate that accepts any cell, `^`, it's because we want to be able to pass any cell to it.  A gate that is only designed for pairs of atoms, `[@ @]`, can't handle all such cases, naturally.  The third cast goes through because the target gate sample type is broader than the cast gate sample type.  A gate that can take any noun as its sample, `*`, works just fine if we choose only to pass it cells, `^`.

We mentioned previously that an iron core has a write-only sample and an opaque core.  Let's prove it.

Let's define a trivial gate with a context of `[g=22 h=44 .]`, convert it to iron with `^|`, and bind it to `iron-gate` in the dojo:

```
> =iron-gate ^|  =>([g=22 h=44 .] |=(a=@ (add a g)))

> (iron-gate 10)
32

> (iron-gate 11)
33
```

Not a complicated function, but it serves our purposes.  Normally (i.e., with gold cores) we can look at a context value `p` of some gate `q` with a wing expression: `p.q`.  Not so with the iron gate:

```
> g.iron-gate
-find.g.iron-gate
```

And usually we can look at the sample value using the face given in the gate definition.  Not in this case:

```
> a.iron-gate
-find.a.iron-gate
```

If you really want to look at the sample you can check `+6` of `iron-gate`:

```
> +6.iron-gate
0
```

...and if you really want to look at the head of the context (i.e., where `g` is located, `+14`) you can:

```
> +14.iron-gate
22
```

...but in both cases all the relevant type information has been thrown away:

```
> -:!>(+6.iron-gate)
#t/*

> -:!>(+14.iron-gate)
#t/*
```

Let's clear up the dojo subject by unbinding `iron-gate`:

```
=iron-gate
```

#### Zinc Cores (Covariant)

Zinc cores are like 'inverted' versions of iron cores.

Assume that `A` is a zinc core and `B` is some other core.  The type of `B` nests under the type of `A` only if the original sample type of `B` nests under the original sample type of `A`.  That is, the target core sample type must be equivalent or a subset of the cast core sample type.  Furthermore, `B` must be either zinc or gold to nest under `A`.

As with iron cores, the context of zinc cores is opaque -- they cannot be written-to or read-from.  The sample of a zinc core is read-only.  That means, among other things, that zinc cores cannot be used as functions.  Function calls in Hoon involve a change to the sample (the sample is replaced with the argument value), which is disallowed as type-unsafe for zinc cores.  (Note: there is currently a bug that allows zinc gates to be called.  This bug will be squished soon.)

We can illustrate the casting properties of zinc cores with a few examples.  The `^&` rune is used to convert gold cores to zinc:

```
> ^+(^&(|=(^ 15)) |=(^ 16))
< 1&xqz
  { {* *}
    {our/@p now/@da eny/@uvJ}
    <19.rga 23.byz 5.rja 36.apb 119.ikf 238.utu 51.mcd 93.glm 74.dbd 1.qct $141>
  }
>

> ^+(^&(|=(^ 15)) |=([@ @] 16))
< 1&xqz
  { {* *}
    {our/@p now/@da eny/@uvJ}
    <19.rga 23.byz 5.rja 36.apb 119.ikf 238.utu 51.mcd 93.glm 74.dbd 1.qct $141>
  }
>

> ^+(^&(|=(^ 15)) |=(* 16))
nest-fail
```

The first two casts succeed because the target sample type is either the same or more specific than the cast core sample type.  The last one fails because the target sample type is a superset.

Zinc cores are not as widely used as gold or even iron cores.  But the arms of a zinc core can still be computed, and the sample can still be read.  Let's test this with a zinc gate of our own:

```
> =zinc-gate ^&  |=(a=_22 (add 10 a))

> a.zinc-gate
22

> $.zinc-gate
32
```

Let's clear up the dojo subject again by unbinding `zinc-gate`:

```
=zinc-gate
```

#### Lead Cores (Bivariant)

Lead cores have more permissive nesting rules than either iron or zinc cores.  There is no restriction on which sample types nest.  That means, among other things, that the sample type of a lead core is both covariant and contravariant ('bivariant').

In order to preserve type safety when working with lead cores, a severe restriction is needed.  The whole payload of a lead core is opaque -- the payload can neither be written-to or read-from.  For this reason, as was the case with zinc cores, lead cores cannot be called as functions.  (Note: because of a type system bug, this is a lie.  You can do function calls on lead gates right now.  But it's bad for your soul and you shouldn't do it.  I'll get that bug soon!)

The arms of a lead core can still be evaluated, however, as long as there is no change to the payload.  We can use the `^?` rune to convert a gold, iron, or zinc core to lead:

```
> =lead-gate ^?  |=(a=_22 (add 10 a))

> $.lead-gate
32
```

But don't try to read the sample:

```
> a.lead-gate
-find.a.lead-gate
```

Once more let's clear up the dojo subject:

```
=lead-gate
```

## Wet Core
