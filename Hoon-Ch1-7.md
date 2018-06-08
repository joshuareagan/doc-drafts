---
navhome: /docs/
sort: 7
title: Cores
---

# Hoon Tutorial 1.7: Cores

In the last couple of lessons you were introduced to cores.  In this lesson, you will get a more complete survey of the various kinds of cores and an explanation of how they work.

## Review of Cores

As stated in previous lessons, a *core* is a cell of (1) a *battery*, and (2) a *payload*.

```
[battery payload]
```

A battery is one or more *arms*.

```
[arm-1 arm-2 arm-3 ...]
```

An arm is a Nock formula that is designed to be evaluated with its parent core as the subject, possibly with a modified *sample*.

This brings us to the payload.  The payload of a core is data.  For almost every core, this is the data necessary for carrying out the instructions of the arm(s) in the battery of the core.

There are two kinds of cores: (1) those with a *sample* and (2) those without.  If a core has a sample, then the payload is a cell of (1) the sample, and (2) the *context*.

```
[sample context]
```

A special case of a core with a sample is a *gate*.  A gate is a one-armed core with a sample.  Function calls in Hoon are implemented with gates.  The instructions for executing the function call are encoded in the arm.  The argument (i.e., the input value) of the function call is stored as the sample, and any other contextual data needed for the arm evaluation is stored in the context.  If no argument is given in the function call, then the arm of the gate is evaluated with default data in the sample.

```
> (add 2 2)
4

> (add)
0
```

For cores with samples in general, the sample is for storing argument data and the context is for other data needed for the arm(s) in the battery.  

## Taking a Closer Look at Gates

You've been making function calls with gates from the first lesson of this tutorial.

```
> (add 2 3)
5

> (mul 2 3)
6
```

By this point you should have some idea of how these calls work.  Let's go further.

### Expanding Out Function Call Expressions

We've been using the `( )` characters to make function calls with the function name on the left side and the argument(s) on the right.  This is actually a syntactical shortcut for a slightly more awkward expression:

```
> ~($ add 2 3)
5

> ~($ mul 2 3)
6
```

The `~( )` expression has three parts: (1) the arm to be evaluated, identified by a wing; (2) the parent core of the arm; and (3) argument(s).  The default sample of the parent core is replaced with the given arguments, and then the arm is evaluated with that core as the subject.

Remember that in a gate, the lone arm has a special name: `$`.  The expression `~($ add 2 3)` is evaluated as follows: the `add` arm of a Hoon standard library core is evaluated, producing a gate with a default sample of `[a=0 b=0]`.  This sample is replaced with a new one: `[a=2 b=3]`.  The `$` arm is then evaluated with this modified version of the gate as the subject.  This produces the answer: `5`.

### Defining a Gate

Defining your own gate is surprisingly simple.  We can use the `|=` rune to do so.  `|=` expressions have two parts: (1) a definition of the sample, and (2) an expression that, when evaluated, is the gate product.  

#### Your First Gate

Let's define a trivial gate as an example.  It will define a function that takes any atom as input and returns `15` as the product:

```
> |=(a=@ 15)
< 1.ata
  { a/@
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
```

We use `a=@` to define the sample as an atom and put the `a` face on it.  So the input value must be an atom, and we can refer to that value with `a` later in the gate expression.  The `15` expression is pretty straightforward: no matter what the input is, return `15`!

Once the gate expression is evaluated, we can see that a gate is produced.  Let's take a closer look:

```
> +2:|=(a=@ 15)
[1 15]

> +6:|=(a=@ 15)
a=0

> +7:|=(a=@ 15)
[ [ our=~bitnus-holleb-rocmeg-fotfer--migsum-nidsym-hilput-patmus
    now=~2018.6.8..08.28.23..b8ef
      eny
    \/0v38u.p4clt.jfo23.p45ov.12of4.23lj1.5k02a.arbap.0h3vb.i3haj.5he6v.8fu69.h\/
      m19q.tmj1l.r2r7r.rji7v.35rpq.46vh8.3stt0.9kggm.khpp2
    \/                                                                         \/
  ]
  ~
  <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
]
```

At `+2` of the gate you can see the arm, the Nock formula `[1 15]`.  At `+6` is the sample with the `a` face on it; the default value of an atom is `0`.  And at `+7`, the context, we see something that looks awfully familiar....

```
> .
[ [ our=~bitnus-holleb-rocmeg-fotfer--migsum-nidsym-hilput-patmus
    now=~2018.6.8..08.34.54..f385
      eny
    \/0v2j1.fegie.gel7m.4p260.09fhi.uijur.ul08k.l9v7l.1j3et.a7vca.3lmb9.577na.j\/
      1gko.mpegi.p74u6.e7spc.jnhpb.phvea.ajjcd.skh0e.s0mhh
    \/                                                                         \/
  ]
  ~
  <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
]
```

Remember that you can use the expression `.` to see the entire subject, which in this case is the dojo subject.

When you define a gate, the subject of the gate expression is stored as the context of the gate.  This is why you see the dojo subject at `+7` of the gate you defined above.  Anything that is used from the subject to define the gate's product expression is therefore guaranteed to remain available to the gate when you call it.  (For this gate we didn't use anything from the subject, but the subject is stored in the context all the same.)

While we're on that topic, how do we call this new gate?

#### Calling Your First Gate

You can call it the way you might with any other gate, using `( )`.

```
> (|=(a=@ 15) 11)
15

> (|=(a=@ 15) 22)
15

> (|=(a=@ 15) 33)
15
```

It works!  You can also use the slightly more complex `~( )` syntax:

```
> ~($ |=(a=@ 15) 11)
15

> ~($ |=(a=@ 15) 22)
15
```

Finally, you can put a face on it and call it using the face.  In the examples below we `b` as the face:

```
> (b 11):b=|=(a=@ 15)
15

> (b 22):b=|=(a=@ 15)
15

> ~($ b 22):b=|=(a=@ 15)
15

> ~($ b 33):b=|=(a=@ 15)
15
```

#### A Second Gate

Let's do a more interesting example.  We'll define a doubling function, i.e., a gate that takes as its sample some atom `a` and returns `(mul 2 a)`.

```
> |=(a=@ (mul 2 a))
< 1.lqe
  { a/@
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>

> +6:|=(a=@ (mul 2 a))
a=0
```

You can see that, as before, the dojo subject is stored as the context of your newly defined function.  If you look at the sample, `+6`, you can see that the default value is `0`.  Let's try calling the gate, giving it the face `double`:

```
> (double 4):double=|=(a=@ (mul 2 a))
8

> (double 5):double=|=(a=@ (mul 2 a))
10

> (double 6):double=|=(a=@ (mul 2 a))
12
```

It works!  Now let's call it with no argument:

```
> (double):double=|=(a=@ (mul 2 a))
0
```

The default sample is `0`, and double `0` is still just `0`.

This gate works by taking advantage of `mul` from the Hoon standard library.  Without `mul`, `double` wouldn't work.  However, we don't ever have to worry about `mul` going missing.  The initial gate definition stores the entire subject in the context, `+7`; that includes all the dependencies of that gate.  Thus, we can be confident that whenever the `double` gate is in the subject a call to it will work.

## Other Cores

Let's talk about some of the cores that aren't gates, and how they're used.

### Cores Without Samples

Cores don't need a sample to be useful.  You've been using one such core from the beginning of this tutorial series.  The core containing the arms defining `add`, `mul`, and many of the other basic Hoon functions has no sample.  Let's look at `add`'s parent core with `..add`:

```
> ..add
<31.ohr 1.jmk $143>

> +...add
<1.jmk $143>
```

At the head is a battery with 31 arms.  At the tail is the payload, which in this case is just another core.

`add`'s parent core is used to store a library of basic Hoon functions.  Each of its arms evaluates to a gate when called.  But arms can defined to evaluate to other products than gates.  Let's use a dojo face define a multi-arm core whose arms evaluate to simpler values, `my-core`:

```
> =my-core |%  ++  two  2  ++  three  (sub 5 2)  --

> my-core
< 2.eha
  { {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>
```

(As you type in the first line above make sure you attend to the spacing of the example; in certain places two spaces are used instead of one.  Copy the spacing exactly!)

We won't fully explain how `|%` expressions work in this lesson, but don't worry if you don't understand that first line---just take for granted that it defines a core with two arms and no sample.  At the head of the core you see the two arms indicated, and in the payload you see the whole subject.  So whereas a gate `|=` definition stores the subject in the context, a `|%` core definition stores the subject in the payload.  That's because there is no sample in `|%` cores.

Let's use `my-core`:

```
> two.my-core
2

> three.my-core
3
```

As you can see, `two` names the arm of `my-core` that computes `2`, which is just `2`; and `three` names the arm that computes `(sub 5 2)`, which is `3`.

Let's erase this core and make a new one, this time with arms that compute to gates:

```
> =my-core

> =my-core |%  ++  double  |=(a=@ (mul 2 a))
  ++  triple  |=(a=@ (mul 3 a))
  --
```

(Again, attend carefully to the spacing as you type this example into the dojo.  Notice also that you're able to enter this expression on multiple lines; you can use line breaks to keep the expression from getting too wide.)

This snippet defines two arm, each of which evaluates to a gate.  It's probably not hard to guess, but let's see what each does:

```
> (double.my-core 14)
28

> (triple.my-core 14)
42
```

If you're familiar with an object-oriented programming language, you may have noticed that an arm of a core is much like a computed attribute of an object.  (If you're not familiar with object-oriented programming, don't worry about it!)

### Doors

A *door* is a core with a sample.  It follows from this definition that a gate is a special case of a door -- a gate is a one-armed door whose arm is named `$`.

But in this section we'll focus more on multi-arm doors.

#### Doors in the Hoon Standard Library

Back in lesson 1.2 you were introduced to atom auras, and you learned how Hoon uses atoms with auras to represent floating point numbers.  For example, the float 3.14159 can be represented as a single-precision (32 bit) float with the literal expression `.3.14159`.  The aura of this single-precision float is `@rs`.

But you can't use the ordinary `add` gate to get the correct sum of two `@rs` atoms:

```
> (add .3.14159 .2.22222)
2.153.203.882
```

That's because the `add` gate is defined for raw atoms.  In lesson 1.2 we told you that you can add two `@rs` atoms as follows:

```
> (add:rs .3.14159 .2.22222)
.5.36381
```

It turns out that the `rs` in `add:rs` is a door.  Let's take a closer look:

```
> rs
<21?vij {r/?($n $u $d $z) <54.tyv 119.wim 31.ohr 1.jmk $143>}>
```

The battery of this core, pretty-printed as `21?vij`, has 21 arms that define functions specifically for `@rs` atoms.  One of these arms is named `add`; it's a different `add` from the standard one we've been using for vanilla atoms.  So when you invoke `add:rs` instead of just `add` in a function call, (1) the `rs` core is produced, and then (2) the name search for `add` resolves to the special `add` arm in `rs`.  This produces the gate for adding `@rs` atoms:

```
> add:rs
< 1.dfv
  {{a/@rs b/@rs} <21.vij {r/?($n $u $d $z) <54.tyv 119.wim 31.ohr 1.jmk $143>}>}
>
```

What about the sample of `rs`?  The pretty-printer shows `r/?($n $u $d $z)`.  What does this mean?  Without yet explaining this notation fully, we'll simply say that the `rs` door's sample can take one of four values: `%n`, `%u`, `%d`, and `%z`.  These argument values represent four options for how to round `@rs` numbers:

```
%n -- round to the nearest value
%u -- round up
%d -- round down
%z -- round to zero
```

The default value is `%z` -- round to zero.  When we invoke `add:rs` to call the addition function, there is no way to modify the `rs` door sample, so the default rounding option is used.  How do we change it?  We use the `~( )` notation: `~(arm core sample)`.  

Let's evaluate the `add` arm of `rs`, also modifying the door sample to `%u` for 'round up':

```
> ~(add rs %u)
< 1.dfv
  {{a/@rs b/@rs} <21.vij {r/?($n $u $d $z) <54.tyv 119.wim 31.ohr 1.jmk $143>}>}
>
```

This is the gate produced by `add`, and you can see that its sample is a pair of `@rs` atoms.  But if you look in the context you'll see the `rs` core.  Let's look in the sample of that core to make sure that it changed to `%u`.  We'll use the wing `+6.+7` to look at the sample of the gate's context:

```
> +6.+7:~(add rs %u)
r=%u
```

It did indeed change.  We can do the same thing for rounding down, `%d`:

```
> +6.+7:~(add rs %d)
r=%d
```

Let's see the rounding differences in action.  Because `~(add rs %u)` produces a gate, we can call it like we would any other gate:

```
> (~(add rs %u) .3.14159265 .1.11111111)
.4.252704

> (~(add rs %d) .3.14159265 .1.11111111)
.4.2527037
```

This difference between rounding up and rounding down might seem strange at first.  There is a difference of 0.0000003 between the two answers.  Why does this gap exist?  Single-precision floats are 32-bit and there's only so many distinctions that can be made in floats before you run out of bits.  This is clearer if we cast the answers to hexadecimals, `@ux`:

```
> `@ux`(~(add rs %u) .3.14159265 .1.11111111)
0x4088.1627

> `@ux`(~(add rs %d) .3.14159265 .1.11111111)
0x4088.1626
```

Here we see a difference of just `1` in these 32 bit hexadecimal numbers.

Just as there is a door for `@rs` functions, there is a Hoon standard library door for `@rd` functions (double-precision 64 bit floats), another for `@rq` functions (quad-precision 128 bit floats), and more.

## Conclusion

There is a lot more to learn about cores than we've covered in this lesson.  But we'll cover more advanced applications of cores in later chapters.  You should now have a fairly good understanding of the basics of subject-oriented programming in Hoon.  In the next chapter of the Hoon tutorial series you'll learn how to write substantive Hoon programs that run in the dojo.  As you learn about this, you'll also deepen your understanding of the various concepts taught in this chapter.
