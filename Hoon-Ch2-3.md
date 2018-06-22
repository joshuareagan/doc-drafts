---
navhome: /docs/
sort: 3
next: true
title: Simple One-Gate Programs
---

# Simple One-Gate Programs

In this lesson we'll try to cover enough basic Hoon to get you writing your own simple programs, while also having a decent understanding of how they work.  Each of these programs defines a simple gate, and hence a simple function.

For each program we encourage you to: (1) write out the program yourself, (2) save it to the `/home/gen` directory of your urbit's pier, and (3) run it from the dojo.  The best way to learn to program in Hoon is by doing it yourself.

We will also continue to run one-line snippets of code in the dojo.  We encourage you to enter these for yourself as well.

## Intro to Writing Gates

You've seen the simple gate that takes any atom and returns `15`.  In flat form:

```
|=(a=@ 15)
```

As you can see, the `|=` takes two subhoons, in this case `a=@` and `15`.  The first subhoon must be a type, and the second may evaluate to any value.  The first defines the gate's sample type and applies a face to it, and the second defines the output of the gate.

You could have picked another type for the first subhoon and left off the face:

```
> (|=(@ 15) 11)
15

> (|=(@ 15) 22)
15

> (|=(* 15) 11)
15

> (|=(* 15) 22)
15

> (|=(^ 15) 11)
nest-fail
```

You passed the arguments `11` and `22` in these function calls.  These are atoms, `@`, and hence nouns, `*`.  So when you defined the sample as either `@` or `*` the evaluation works as desired.  But they aren't cells, `^`, so when you defined the sample of the gate as a cell and called it, you got a `nest-fail` crash.  For all function calls, Hoon's type system checks whether the type of the argument fits under the type of the sample.  If not, the the function will not be evaluated and you get the `nest-fail`.

Ordinarily a face is included in the first subhoon of `|=` hoons.  This makes it easier to refer to the sample in the second subhoon of the `|=` hoon.  After all, we usually want gate products to depend on the arguments passed.

```
> (|=(a=@ (add 2 a)) 12)
14

> (|=(a=@ (add 2 a)) 14)
16
```

### Checking the Gate's Output

You are probably fallible.  A fallible person will, at least occasionally, write code that produces output of the wrong type.  Hoon's type system is designed to help you avoid this problem, but you have to 'ask' using some sort of cast.  (You first learned about casts in lesson 1.2 (link).)  It is good practice to include a cast with every gate you write in Hoon.  There are various ways of casting but for now we'll stick to using `^-`.

Consider the even number checker given in the last lesson:

```
|=  a=@                                                 ::  even number checker
^-  ?                                                   ::  output is a flag
=(0 (mod a 2))                                          ::  if remainder=0 'yes'
```

The first `|=` subhoon is `a=@`, and the second is the rather larger:

```
^-  ?
=(0 (mod a 2))
```

This latter hoon uses the `^-` rune, which itself takes two subhoons.  The first must be a type, `?`, and the second subhoon is to be evaluated, `=(0 (mod a 2))`.  The resulting value of the second has its own type that is compared against the first.  If the latter subhoon's type fits under the first type, the type-check passes.  If not the program will fail to compile, giving a `nest-fail` crash.  The program -- at least potentially -- produces the wrong kind of value.

The way Hoon knows the value of the second `^-` subhoon is by 'type inference'.  We'll talk in more detail about how Hoon's type inference works later in the chapter.

Save the even number checker to the `/home/gen` directory of your pier as `even.hoon` and run it from the dojo:

```
> +even 11
%.n

> +even 22
%.y
```

Now replace the `=(0 (mod a 2))` in the third line with just `15`.  The result:

```
|=  a=@                                                 ::  even number checker
^-  ?                                                   ::  output is a flag
15                                                      ::  return 15
```

Save this and try to run it from the dojo again:

```
> +even 22
[stack trace]
nest-fail
```

You changed the gate so that the output would be `15`, which is an unsigned decimal atom, `@ud`.  But you're casting for a flag, `?` -- either `%.y` or `%.n`.  Because `@ud` doesn't nest under `?` the program failed to compile.

### Multiple Inputs and Outputs

A gate is a one-armed core with a sample:

```
         [gate]
        /      \
    [$ arm]  [payload]
             /       \
        [sample]   [context]
```

When a function call to a gate occurs, the `$`arm is evaluated with a modified version of the gate itself as the subject.  The modification to the gate: the default sample is replaced with the argument(s) of the function call.  This is how `$` 'knows' what the argument(s) are.

The sample is just the noun in that slot.  The product of the arm evaluation is just another noun.  Thus, gates always have a single noun for input and a single noun for output.

Naturally we would like to use functions that take multiple inputs, and other functions with multiple outputs.  It isn't difficult to realize the solution: cells.

Let's write a program that can check two numbers at a time and tell us if each is even:

```
::
::  even2.hoon
::
::  this gate takes a pair of atoms, tests whether each is even, and then
::  returns two flags.
::  
|=  [a=@ b=@]                                           ::  check two numbers
^-  [? ?]                                               ::  output is two flags
:-  =(0 (mod a 2))                                      ::  :-(x y) makes [x y]
=(0 (mod b 2))                                          ::
```

The first `|=` subhoon is now `[a=@ b=@]`.  This indicates a complex type for the gate's sample.  The gate takes a pair of atoms, labelling the first `a` and the second `b`.

The second `|=` subhoon is:

```
^-  [? ?]
:-  =(0 (mod a 2))
=(0 (mod b 2))
```

The first of two `^-` subhoons is `[? ?]`, indicating another complex type.  The inferred type of the second `^-` subhoon must be a cell of two flags, or else the program will fail to compile.  The second `^-` subhoon:

```
:-  =(0 (mod a 2))
=(0 (mod b 2))
```

The `:-` rune takes two subhoons and makes them into a cell.  Each of the subhoons, `=(0 (mod a 2))` and `=(0 (mod b 2))`, evaluates to a flag, `?`.

Save the whole program in your pier as `even2.hoon` (or as something else if you like) and test it in the dojo.  Remembering to pass two arguments as a cell:

```
> +even2 [2 2]
[%.y %.y]

> +even2 [2 3]
[%.y %.n]

> +even2 [3 3]
[%.n %.n]

> +even2 [22 33]
[%.y %.n]
```

A gate may take as input or produce as output arbitrarily complex cells.  For example, one could write a gate that takes a cell of five different things:

```
|=  [a=* b=@ud c=^ d=? e=@sb]
...
```

## Basic Tools

### `.+` and `.=`

The `.+` rune is for 'incrementing'; it takes an atom and adds `1` to it.  That's it!  Here it is in action:

```
> .+  22
23

> .+(22)
23

> .+(101)
102
```

But it's nearly always used in its irregular form, which lacks the `.`:

```
> +(22)
23

> +(101)
102
```

You've already used the `.=` rune, particularly in its irregular form `=( )`.  This rune takes two subhoons, evaluates them, and then does a simple test of equality on the resulting values.

```
> =(22 11)
%.n

> =(22 (add 11 11))
%.y
```

But there's one more quirk that you should know about: `.=(a b)` includes a type check.  Of the inferred types of `a` and `b`, one must nest in the other or else the code won't compile:

```
> =(12 [12 14])
nest-fail
```

The inferred type of `12` is `@ud`, and the inferred type of `[12 14]` is `[@ud @ud]`.  Neither type nests under the other, so the code fails.  One can avoid this check for type safety by casting one of the values to a noun, `*` -- everything nests under `*`:

```
> =(^-(* 12) [12 14])
%.n
```

The best policy is to avoid wiggling out of type safety checks, however.

### Introduction to Conditionals: The `?` Rune Family

We won't cover all the `?` runes here -- we'll save some for the upcoming lesson on types.  But we'll cover enough to get you writing simple Hoon programs.

#### `?:` if-then-else

The `?:` rune indicates an 'if-then-else' expression and takes three subhoons: (1) the condition, which must evaluate to a flag, (2) a hoon to be evaluated if the condition evaluates to `%.y`, and (3) a hoon to be evaluated if the condition evaluates to `%.n`.

```
> ?:(%.y 11 22)
11

> ?:  %.y
    11
  22
11

> ?:(%.n 11 22)
22
```

#### `?.` ifnot-then-else

The `?.` rune does precisely the same thing as `?:`, except that it reverses the order of the second and third subhoons: if the condition evaluates to `%.n` then the second subhoon is evaluated, otherwise the third subhoon is evaluated.

```
> ?.(%.y 11 22)
22

> ?.  %.y
    11
  22
22

> ?.(%.n 11 22)
11
```

The reasons for Hoon's having both `?:` and `?.` involve code neatness and clarity.  Use whichever makes the code easier to understand.

#### `?&` logical 'AND'

The `?&` rune is for logical 'AND'.  It takes two or more subhoons, each of which must evaluate to a flag, `?`; and if all subhoons evaluate to `%.y` then the result of the `?&` hoon is `%.y`.  Otherwise it's `%.n`.  In tall form the hoon is terminated with `==`.

```
> ?&  %.y  %.y  %.y  ==
%.y

> ?&  &  |  &  ==
%.n

> ?&(%.y %.y %.y)
%.y

> ?&(& =(1 2) &)
%.n
```

Using `&` by itself is another way of indicating 'yes' and `|` by itself another way of indicating 'no':

```
> &
%.y

> |
%.n
```

The irregular form of `?&` allows you to drop the `?` symbol:

```
> &(& & &)
%.y

> &(| & &)
%.n

> &(& | &)
%.n

> &(& & |)
%.n
```

#### `?|` logical 'OR'

The `?|` rune is for logical 'OR'.  It takes two or more subhoons, each of which must evaluate to a flag, `?`.  If at least one subhoon evaluates to `%.y` then the whole `?|` hoon evals to `%.y`; otherwise it's `%.n`:

```
> ?|  %.n  %.n  %.n  ==
%.n

> ?|  |  |  &  ==
%.y

> ?|(=(12 12) | |)
%.y
```

The irregular form drops the `?` symbol:

```
> |(=(12 12) | |)
%.y

> |(=(12 13) | |)
%.n
```

#### `?!` logical 'NOT'

The `?!` rune is for logical 'NOT'.  It takes one subhoon that evaluates to a flag and returns the opposite value.

```
> ?!  &
%.n

> ?!(|)
%.y
```

The irregular form drops the `?` symbol and uses no parentheses:

```
> !|
%.y

> !=(22 33)
%.y
```

#### Are All Three Even?

Let's write a program that takes three atoms, `@`, and returns `%.y` if they're all even, otherwise `%.n`.

```
|=  [a=@ b=@ c=@]                                       ::  check three @
^-  ?                                                   ::  output is a flag
?&  =(0 (mod a 2))                                      ::  `a` is even AND
    =(0 (mod b 2))                                      ::  `b` is even AND
    =(0 (mod c 2))                                      ::  `c` is even
==
```

Save this in your urbit's pier as `even3.hoon` and run it from the dojo:

```
> +even3 [12 13 14]
%.n

> +even3 [12 16 14]
%.y
```

#### If So, Multiply -- Otherwise Add

Let's write a slightly more complicated program.  The gate will take three atoms, `@`, and if all three are even it'll multiply all of them together.  Otherwise, it will add them all together.

```
|=  [a=@ b=@ c=@]                                       ::  take three @
^-  @                                                   ::  output is an @
?:  ?&  =(0 (mod a 2))                                  ::  if `a` is even AND
        =(0 (mod b 2))                                  ::  `b` is even AND
        =(0 (mod c 2))                                  ::  `c` is even
    ==
  (mul a (mul b c))                                     ::  then multiply
(add a (add b c))                                       ::  otherwise add
```

Save this as `addmul.hoon` and try it from the dojo:

```
> +addmul [2 2 2]
8

> +addmul [2 2 3]
7

> +addmul [4 4 4]
64

> +addmul [4 4 5]
13
```

### Subject Modification: The `=` Rune Family

As with the `?`, we won't explain all the runes of the `=` family -- just a few to get you started.

#### `=>` Evaluate a Hoon on a Subject

The `=>` rune takes two subhoons.  The first subhoon is evaluated, with the result becoming the new subject; the second subhoon is evaluated on that subject.

A few examples:

```
> =>  [[a=12 b=14] [c=16 d=18]]  +
[c=16 d=18]

> =>  [[a=12 b=14] [c=16 d=18]]  -
[a=12 b=14]

> =>  [[a=12 b=14] [c=16 d=18]]  +>
d=18

> =>  [[a=12 b=14] [c=16 d=18]]  a
12

> =>  [[a=12 b=14] [c=16 d=18]]  d
18
```

#### `=<` Evaluate a Hoon on a Subject, Reversed

The `=<` rune is just like `=>` but with the subhoons reversed.  The second subhoon is evaluated, with the result becoming the new subject; the first subhoon is evaluated on that subject.

A few examples:

```
> =<  +  [[a=12 b=14] [c=16 d=18]]
[c=16 d=18]

> =<  -  [[a=12 b=14] [c=16 d=18]]
[a=12 b=14]

> =<  +>  [[a=12 b=14] [c=16 d=18]]
d=18

> =<  a.-  [[a=12 b=14] [c=16 d=18]]
12

> =<  d.+  [[a=12 b=14] [c=16 d=18]]
18
```

The `=>` and `=<` runes should feel vaguely familiar.  You've already used the irregular version of `=<` with the `:` symbol:

```
> +:[[a=12 b=14] [c=16 d=18]]
[c=16 d=18]

> -:[[a=12 b=14] [c=16 d=18]]
[a=12 b=14]

> +>:[[a=12 b=14] [c=16 d=18]]
d=18

> a:[[a=12 b=14] [c=16 d=18]]
12

> d:[[a=12 b=14] [c=16 d=18]]
18
```

#### `=+` 'Pin' a Noun to the Subject

The `=+` rune takes two subhoons.  The first subhoon is evaluated and then the resulting value is 'pinned' to the head of the subject -- that is, the old subject becomes the tail of the new subject, and the value produced becomes the head of the new subject.  The second `=+` subhoon is evaluated on the new subject.  Examples:

```
> =>  [22 33 44]  =+(101 -)
101

> =>  [22 33 44]  =+(101 +)
[22 33 44]

> =>  [22 33 44]  =+(101 +1)
[101 22 33 44]

> =>  [b=22 c=33 d=44]  a
-find.a

> =>  [b=22 c=33 d=44]  =+(a=101 a)
101
```

#### `=/` Pin a Faced (and Possibly Typed) Noun To the Subject

The `=/` rune is a more powerful variation of `=+`.  `=/` takes three subhoons: (1) the first is the face for the noun to be pinned (e.g., `a`), and possibly includes a type (e.g., `a=@`); (2) the second is the noun to be pinned; and (3) the third is the hoon to be evaluated on the modified subject.  

Some examples:

```
> =/(b=@ 22 [b b b])
[22 22 22]

> =/(b=@ 22 (add b b))
44
```

The `=/` rune is the closest thing Hoon has to an instruction for 'initializing a variable' while giving it an initial value.  What actually occurs in Hoon is that the value is pinned to the head of the subject, and the indicated face now refers to that value.

#### `=|` Pin a Faced, Typed Default Noun to the Subject

The `=|` is a lot like the `=/` rune except that (1) you must include type information when you use it, and (2) the programmer doesn't set the initial value of the face -- it's a default value for the type in question, sometimes called a 'bunt' value.  The `=|` rune takes two subhoons.  The first is a face names (e.g., `a`), followed by `=`, followed by a type (e.g., `@`).  The second subhoon is to be evaluated against the modified subject.

```
> =|  a=@  a
0

> =|  a=^  a
[0 0]

> =|  a=?  a
%.y
```

The `=|` rune is a lot like 'initializing a variable' but for faces, without giving the face an interesting initial value (a default value for the type is used instead, i.e., the 'bunt').

#### `=.` Change a Leg of the Subject

The `=.` rune takes three subhoons.  The first is a wing hoon, indicating the leg of the subject whose value is to be changed.  The second is the new value for the indicated leg.  The third is the hoon to be evaluated against the modified subject.

```
> =/  a=@  22  =.(a 99 a)
99

> =/  a=@  22  [a =.(a 99 a)]
[22 99]

> =>  [12 14]  =.(+2 22 .)
[22 14]
```

This rune is often used for modifying the value of a face whose value was set beforehand.

## Recursion

How do we do loops in Hoon?  Like other functional programming languages Hoon uses [recusion](https://en.wikipedia.org/wiki/Recursion_(computer_science)).  In rough terms, a recursive function solves a problem in part by calling on itself to solve a slightly easier or smaller case of the problem.  The self-calling loop continues until the simplified problem is so simple that it has a trivial solution.

Seeing an example will make this process easier to understand.

### Addition

Let's write a gate that takes a pair of atoms, `@`, and which returns the sum of these atoms.  There is a cheap solution to this problem, of course.  The Hoon standard library has `add` in it:

```
> (add 22 33)
55
```

But let's solve the problem without using `add` from the Hoon standard library.  We have the `.+` rune for adding `1` and the `.=` rune for testing equality.  We'll also use `dec` from the standard library and some recursion.  That's it!

What's `dec`?  It takes some atom, `@`, and returns the decrement -- i.e., it takes some number `a` and returns `a - 1`:

```
> (dec 101)
100

> (dec 51)
50

> (dec 11)
10
```

For adding two atoms together, the algorithm we'll use is very simple.  We start with two numbers, `a` and `b`.  If `b` is `0`, then the answer is `a`.  (This is the trivial case of addition for our algorithm, sometimes also called the 'base' case.)  Otherwise, the answer is the sum of `a + 1` and `b - 1` -- now do *that* addition.  (This is the *recursive* case.)

Do you see how this process calls for a loop of sorts?  If you want to sum `7` and `2`, check to see whether the second number is `0`.  It isn't, so now do the sum of `8` and `1`.  Is the second number `0`?  No, so do the sum of `9` and `0`.  Now the second number is `0` -- the answer is `9`.

Here's the program:

```
|=  [a=@ b=@]                                           ::  take two @
^-  @                                                   ::  output is one @
?:  =(b 0)                                              ::  if b is 0
  a                                                     ::    return a, else
$(a +(a), b (dec b))                                    ::  add a+1 and b-1
```

Save it as `add.hoon` in your urbit's pier and run it from the dojo:

```
> +add [22 33]
55

> +add [122 33]
155
```

It works!

Every line of `add.hoon` should be clear up to the last one.  What's going on there?  The `$( )` is used for recursive function calls.  It calls the function it's in, but with certain values modified according to the instructions inside the parentheses.  Two values are being modified for the call: `a` is modified to `+(a)`, and `b` is modified to `(dec b)`.  A comma is used to separate the two modification instructions.

### `%=` Resolve to a Wing with Changes

The `$( )` syntax is an irregular form of the the `%=` rune.  We can rewrite that part of `add.hoon` in regular form:

```
|=  [a=@ b=@]                                           ::  take two @
^-  @                                                   ::  output is one @
?:  =(b 0)                                              ::  if b is 0
  a                                                     ::    return a, else
%=  $                                                   ::  resolve to $
  a  +(a)                                               ::  but change a to +(a)
  b  (dec b)                                            ::  and b to (dec b)
==
```

This version is precisely equivalent to the old one.  The first `%=` subhoon is a wing expression.  In the program above, this is just `$`.  The other subhoons of `%=` indicate the changes to be made upon resolution to the wing.  You can make as many of these changes as desired when using `%=`, so the hoon must be terminated with `==`.

Remember from lessons 1.4-1.5 (links) that wing resolution works in two different ways.  If the wing resolves to a leg, that fragment of the subject is returned.  If the wing resolves to an arm, the arm is evaluated with its parent core as the subject.  In lesson 1.5 (link) you learned that `$` is the special name of the arm in a one-armed core.  Because we wrote `add.hoon` with `|=` it defines a gate, and a gate is a one-armed core.  The `%=` hoon above thus evaluates the `$` arm of the gate, but after having made changes to the parent core first: `a` is incremented, and `b` is decremented.

### Decrement

We've used `dec`, but now let's write our own version.  That is, let's write a gate that takes some atom, `@`, and returns the 'decrement' of that value.  There is a cheap solution to this problem too.  The Hoon standard library has the `sub` function in it for subtraction:

```
> (sub 100 50)
50

> (sub 100 1)
99
```

But let's solve the problem without using the Hoon standard library.  We have the `.+` rune for adding `1`, the `.=` rune for testing equality, and various runes of the `=` family for manipulating the subject.  And we'll again use recursion.

How do we decrement using only those tools?  The algorithm is pretty simple.  We'll start by defining a counter value, `0`, and give it the face `c`.

If `c + 1` equals the value to be decremented, then `c` is the answer.  (This is the trivial case.)  Otherwise, we increment `c` by `1` and try again.  (This is the recursive case.)

There's one quirk about this algorithm that makes it different from the addition algorithm.  We didn't set any initial values in the addition program, and so calling the whole function at the end wasn't a problem.  If we do the same thing with decrement -- call the whole function at the end -- we'll end up setting the same face, `c`, to the same initial value again, `0`.  We don't want that.  We want the counter to increase in value each time!  There should be a recursion start point that comes after `c` is set initially.  For that you can use the `|-` rune.

Here's the program:

```
|=  a=@                                                 ::  take one @
=/  c=@  0                                              ::  set c to 0
|-                                                      ::  recursion start
^-  @                                                   ::  output is @
?:  =(a +(c))                                           ::  if a = c+1
  c                                                     ::    return c, else
$(c +(c))                                               ::  add 1 to c, recurse
                                                        ::    back to the |-
```

Save it as `dec.hoon` and try it from the dojo:

```
> +dec 101
100

> +dec 97
96

> +dec 11
10
```

At an intuitive level this code should make sense.  The `$( )` at the end effectively serves as a loop going back to the `|-` rune, with a modification to the value of `c`.  But isn't `$` the name of the gate's single arm?  What's going on?  Let's take a moment to understand how `|-` works at a deeper level.

### `|-` Make and Call a One-armed Core

This rune makes a one-armed core whose arm has the name `$`, and then it calls `$`.  The `|-` rune takes one subhoon, which determines what `$` does:

```
> |-  2
2

> |-  (add 22 33)
55
```

Let's look at the decrement program again, this time using the regular version of the `%=` rune.

```
|=  a=@                                                 ::  take one @
=/  c=@  0                                              ::  set c to 0
|-                                                      ::  recursion start
^-  @                                                   ::  output is @
?:  =(a +(c))                                           ::  if a = c+1
  c                                                     ::    return c, else
%=  $                                                   ::  resolve to $
  c  +(c)                                               ::  add 1 to c
==
```

Strictly speaking, when you use `$( )` or `%=  $` you're asking for a resolution to `$` (with changes).  And because `$` is an arm, that means evaluating it with its parent core as the subject.

When we use `|-` in our decrement program, we're defining a _new_ core whose arm name is `$`.  So the subject ends up having two arms named `$` in it: one for the gate defined by `|=`, and one for the core defined by `|-`.

When `$( )` or `%=  $` is run the subject is searched for the `$` arm, and the `|-` arm is found first because it was defined after the `|=` arm.  You can skip the first name match for `$` by using `^`, if you like.  That is, you can recurse all the way to the beginning of the function using `^$( )` or `%=  ^$`.

### Exercises

You should know enough at this point to write some basic Hoon programs.  Practice with some of these exercises.  Solutions are provided at the bottom of the page.  You may want to use

#### Subtraction

Write a program that takes two numbers and returns the difference.  You may use `dec`, but don't use anything else from the Hoon standard library.

#### Factorial

Write a program that takes a number `n` and returns the factorial of `n`, `n!`.  `1!` is `1`, `2!` is `2 x 1`, `3!` is `3 x 2 x 1`, `4!` is `4 x 3 x 2 x 1`, and so on.  You may use the Hoon standard library.

#### Prime Number Checker

Write a program that takes a number `n` and returns `%.y` if it's prime, and `%.n` if it isn't.  You may use the Hoon standard library.

### Exercises Answers

There are many possible solutions so yours may look a bit different from the ones we have below.  These solutions aren't optimized for performance.  Instead, clarity and simplicity have been prioritized.

#### Subtraction

```
|=  [x=@ y=@]
^-  @
?:  =(y 0)
  x
$(x (dec x), y (dec y))
```

#### Factorial

```
|=  n=@
^-  @
?:  (lte n 1)
  1
(mul n $(n (dec n)))
```

#### Prime

```
|=  a=@
?:  (lth a 2)  |
?:  =(a 2)  &
=/  c  2
|-  ^-  ?
?:  =(0 (mod a c))  |
?:  (gth (mul c 2) a)  &
$(c +(c))
```
