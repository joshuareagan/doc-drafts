---
navhome: /docs/
sort: 1
next: true
title: Intro to Hoon Data Structures
---

# Hoon Tutorial 1.1: Intro to Hoon Data Structures

## Getting Started with Urbit

This tutorial series is designed to teach you the basics of Hoon without assuming you have an extensive programming background.  In fact, you should be able to follow it without any programming experience at all, though of course experience helps.

Before we begin, however, you will need to (1) have Urbit installed and (2) boot a 'ship' that you can use for testing Hoon examples.  We strongly encourage you to type in all the examples of each lesson.  These lessons are meant for the beginner but they aren't meant to be skimmed.

### Installing Urbit

First, [install Urbit](../../using/install/) on any Mac or Unix machine.  On Windows, make a virtual Linux machine using VirtualBox or a similar tool.

Once you're finished you can boot your very own ship.

### Ships

A *ship* is an Urbit virtual computer with persistent state that can connect to the Urbit network.  Each ship is associated with a unique number that plays four distinct roles: (1) it's the name of a the ship in question, (2) it's an address on the Urbit network, (3) it's a cryptographic identity, and (4) it's (in principle) a human memorable name.  Normally a ship's name is represented as a string starting with `~`, as in `~zod` or `~taglux-nidsep`.

These may not look like numbers, but they are.  Each ship name is written in a base-256 format, where each digit is a syllable.  Imagine your phone number as a pronounceable string which sounds like a name in a foreign language. `~zod` is ship zero (making it a very important ship indeed!). An ordinary user-level ship is a 'planet', and it's named by a 32-bit number which becomes a four-syllable string.  The planet name `~taglux-nidsep` is the number 6,095,360.

For this tutorial you'll boot and run what we call a 'comet'.  *Comets* are ships whose names are 128-bits or 16 syllables, such as:

`~hillyn-pitwet-hasdur-paswer--miszod-rabpex-divrup-fogdur`

Comet names aren't quite as memorable as others, but they're disposable identities that anyone can make for free to join the live network. Thus, comets make the ideal ship for playing around with Hoon, asking for help from other Urbit devs in [Talk](../../using/messaging/) if you need it, and showing off Urbit to your hacker friends.

### Booting a Comet

To create your comet, go into the command line and run the following command from the directory that was created during Urbit installation:

```
$ urbit -c mycomet
```

This will take a few minutes and will spin out a bunch of boot messages. Toward the end, you'll see something like:

```
ames: on localhost, UDP 31337.
http: live (insecure, public) on 8080
http: live ("secure", public) on 8443
http: live (insecure, loopback) on 12321
~palnul_nocser:dojo>
```

(There will likely be other messages as well.)

Looking at the prompt, `~palnul_nocser:dojo>`, to the left of the `:` you'll see an abbreviation of the comet name you just booted.  To the right is the name of a local application, `dojo`.  *Dojo* is the Urbit command line app; it's also a Hoon [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) we'll use to run simple Hoon examples.

To make sure everything is working, type `(add 2 2)` at the prompt and hit return.  Your screen now shows:

```
ames: on localhost, UDP 31337.
http: live (insecure, public) on 8080
http: live ("secure", public) on 8443
http: live (insecure, loopback) on 12321
> (add 2 2)
4
~palnul_nocser:dojo>
```

You just used a function from the Hoon standard library, `add`, to compute your first 'noun'.  But before we get started on what a noun is, quit Urbit with `ctrl-d`:

```
> (add 2 2)
4
~palnul_nocser:dojo>
$
```

Your comet isn't running anymore and you're back at your computer's normal terminal prompt. You now have a `mycomet` directory for your Urbit comet; this directory is your comet's *pier*.  

Right now the only thing in your pier is your comet's system files:

```
$ ls -a /path/to/mycomet
./  ../ .urb/
```

Restarting your already-created comet from the terminal is like creating a new one, but without the `-c` flag.  (The `-c` is for "create".)

```
$ urbit mycomet
```

Because the comet has already been booted it won't take very long to get it running again.  There are also fewer startup messages:

```
$ urbit mycomet
[...]
ames: on localhost, UDP 31337.
http: live (insecure, public) on 8080
http: live ("secure", public) on 8443
http: live (insecure, loopback) on 12321
~palnul_nocser:dojo>
```

We should note that your comet's state is [ACID](https://en.wikipedia.org/wiki/ACID).  You can stop your comet politely with `ctrl-d`, abruptly with `ctrl-z`, even violently with `kill -9`.  None of these should cause the comet to lose data.  If you do manage to corrupt your comet, it's a bug; please let us know.

### Another Noun

You've already made one noun in dojo.  Now that your comet is running again, let's make another.  Enter the number `42`.

*We won't show the `~palnul_nocser:dojo> ` prompt from here on out.  We'll just show the echoed command along with its result.*

You'll see:

```
> 42
42
```

You asked dojo to evaluate the noun `42` and it echoed the number back at you.  But what is a noun?

## Noun Definition

In Urbit every piece of data (that is, every 'datum') is a noun.  In order to understand Hoon we must first understand nouns.

A *noun* is either an atom or a cell.  An *atom* is an unsigned integer of any size.  A *cell* is an [ordered pair](https://en.wikipedia.org/wiki/Ordered_pair) of nouns, usually indicated with square brackets around the nouns in question; i.e., `[a b]`, where `a` and `b` are nouns.

Here are some atoms:

- `0`
- `87`
- `325`

Here are some cells:

- `[12 13]`
- `[12 [487 325]]`
- `[[12 13] [87 65]]`
- `[[83 [1 17]] [[23 [32 64]] 90]]`

All of the above are nouns.

You can think of a noun as a [binary tree](https://en.wikipedia.org/wiki/Binary_tree) whose leaves are numbers.  To visualize this, consider the following representation of the noun: `[12 [17 45]]`:

```
[12 [17 45]]
    .
   / \
 12   .
     / \
   17  45
```

Each number is an atom; each dot is a cell. Another example, this time for `[[7 13] [87 65]]`:

```
[[7 13] [87 65]]
        .
      /   \
    .       .
   / \     / \
  7  45  87  65
```

An atom is a trivial tree of just one node; e.g., `42`.

## Atoms in the Dojo

Atoms are simply unsigned integers.  These seem to be represented by the dojo as unsigned integers in decimal notation, e.g., `42`.  Is this always the case?

To be sure we understand this decimal thing, let's enter some atoms:

```
> 3
3
```

Seems straightforward.

```
> 32
32
```

Why not try it again to make sure?

```
> 320
320
```

Can we add another zero?

```
> 320
```

It didn't work!  When you tried to type that next zero and turn `320` into `3200`, the dojo actually *deleted the `0` and beeped at you.*

Isn't an atom an unsigned integer of any size?  It is.  But the way to represent the four-digit number `3200` is actually `3.200`:

```
> 3.200
3.200
> (mul 32 100)
3.200
```

Just think of it as "3,200", written the German way with a dot.

Why?  Long numbers are difficult to read.  Human beings know this, so we group digits in threes.  For some historical reason most programming languages don't use this marvelous innovation.  Hoon does.

English notation for decimals is more common than German notation.  Unfortunately, dot is URL-safe and comma isn't, and it's nice to have a regular syntax for atoms that's URL-safe.

As for why the dojo deleted your zero: it parses the command line as you type and rejects any characters after the parser stops.  This prevents you from entering expressions that aren't well-formed.

## Cells in the Dojo

There's not much mystery about cells.  The left of a cell is called the *head*, and the right is called the *tail*.  Cells are typically represented in Hoon as square brackets around a pair of nouns.

```
> [32 320]
[32 320]
```

In this cell `32` is the head and `320` is the tail.

So far whenever we've entered a noun into the dojo, it not only returned the same noun; the noun was printed in the same way it was entered.  This doesn't always happen.  The dojo is obligated to evaluate the input and return the correct answer, but it doesn't have to show the result in exactly the same way:

```
> [6 [62 620]]
[6 62 620]

> [6 62 620]
[6 62 620]

> [[6 62] 620]
[[6 62] 620]
```

In the first example, the inner pair of square brackets is dropped.  Whenever cell brackets are missing in Hoon, the association is assumed to be to the right.  That is, the tail is assumed to be a cell of the right-most elements.

If you look at the first and third nouns from above as binary trees you can see the difference:

```
 [6 [62 620]]       [[6 62] 620]
     .                  .
    / \                /  \
   6   .              .   620
      / \            / \
    62  620         6  62
```

The dojo always drops superfluous cell brackets:

```
> [[12 13] [[14 15] [16 17]]]
[[12 13] [14 15] 16 17]
```

### Exercise 1.1.1

True or false: Without using the dojo, assess whether each of the following pairs of nouns is equivalent.  (Answers are at the bottom of the page.)

```
1.  [[1 3] [6 2]]                 [1 3 [6 2]]
2.  [[[17 18] 20] 19 21]          [[[17 18] 20] [19 21]]
3.  [[[[[12 13] 14] 15] 16] 17]   [12 13 14 15 16 17]
4.  [12 [13 [14 [15 [16 17]]]]]   [12 13 14 15 16 17]
5.  [37 [99 17] 83]               [37 [[99 17] 83]]
```

## Noun Addresses

What if you want to refer to a fragment of a noun, and not the whole thing?  One way is to use the 'address' of that fragment.  To define noun addresses we'll need to use recursion.  Address `1` of a noun is the entire noun.  If the noun at address `n` is a cell, then the head is at address `2n` and the tail is at address `2n + 1`.  For example, if address `5` of some noun is a cell, then its head is at address `10` and its tail address `11`.

If the way this works isn't immediately clear, remember that each noun can be understood as a binary tree.  The following diagram shows the address of the first several nodes of the tree:

```
             1
           /   \
         2       3
       /   \    /  \
      4     5  6    7
     / \   / \     / \
    8   9 10 11   14 15
```

Let's do some examples in the dojo.  We're going to use the `axis` operator, `+`, to return fragments of a noun.  For any unsigned integer `n`, `+n` indicates the fragment at address `n`.

```
> +1:[22 [33 44]]
[22 33 44]

> +2:[22 [33 44]]
22

> +3:[22 [33 44]]
[33 44]

> +6:[22 [33 44]]
33

> +7:[22 [33 44]]
44
```

You'll notice that we use the `:` between the axis notation and the noun.  Don't worry about what this means yet; we'll explain it in another lesson.  For now you can read it as the word "of": `+7:[22 [33 44]]` is "axis `7` of `[22 [33 44]]`".

What happens if you ask for a fragment that doesn't exist?

```
> +4:[22 [33 44]]
ford: build failed *stack trace*
```

Let's do a few more examples:

```
> +2:[[21 34] [87 28]]
[21 34]

> +5:[[21 34] [87 28]]
34

> +15:[21 32 43 54 65 76]
[54 65 76]

> +31:[21 32 43 54 65 76]
[65 76]
```

For those who prefer to think in terms of binary numbers, there is another (equivalent) way to understand of noun addressing.  As before, the root of the binary tree (i.e., the whole noun) is at address `1`. For the node of a tree at any address `b`, where `b` is a binary number, you get the address of the head by concatenating a `0` to the end of `b`; and to get the tail, concatenate a `1` to the end. For example, the head of the node at binary address `111` is at `1110`, and the tail is at `1111`.

### Exercise 1.1.2

Without using the dojo, evaluate the following expressions.  (Answers are at the bottom of the page.)

```
1. +6:[[34 54] [86 48]]
2. +3:[13 [[[65 35] 54] 77]]
3. +12:[13 [[[65 35] 54] 77]]
4. +7:[13 [[[65 35] 54] 77]]
5. +10:[[[2 4 5] [3 6]] 11]
```

### Exercise 1.1.3

Without using the dojo, for each fragment below, provide the address relative to the noun `[[51 [52 [53 54] 55] 56] [57 58]]`.  (Answers are at the bottom of the page.)

```
1. [57 58]
2. [[52 [53 54] 55] 56]
3. 51
4. [52 [53 54] 55]
5. [53 54]
```

That's it for our basic introduction to nouns.  Hit `ctrl-d` to exit your comet, or else move on to the next lesson.

## 1.1 Exercise Answers

### 1.1.1

1. F
2. T
3. F
4. T
5. T

### 1.1.2

1. 86
2. [[[65 35] 54] 77]
3. [65 35]
4. 77
5. 3

### 1.1.3

1. +3
2. +5
3. +4
4. +10
5. +42
