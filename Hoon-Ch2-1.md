---
navhome: /docs/tutorials/ch2-hoon/
sort: 1
next: true
title: Hoon Programs
---

# Writing Hoon Programs

So far you've only entered short snippets of Hoon code into the dojo.  Let's look at some substantive programs!

There many kinds of Hoon programs: generators, Gall apps, Sail documents, and more.  In this chapter we'll work exclusively with a subset of the first kind: generators.  This narrow focus will allow us to set aside complications that aren't necessary for learning the basics of Hoon.  We won't even talk about the full range of generators supported in Urbit just yet.  Other kinds of programs will be covered in later chapters.

## Setup

For now we'll work exclusively with 'naked' generators.  A naked generator is a gate that can be run from the dojo and that takes exactly one argument for its sample.  Generators are to be stored in the `/gen` directory of your urbit's filesystem.

To use your favorite text editor to write Hoon programs you'll first need to expose your urbit's filesystem to Unix.  If you haven't already done so, 'mount' your home desk as follows:

```
> |mount %
>=
```

(Note: we are assuming that you have mounted from your 'home' desk in the dojo.  If you don't know what this means, don't worry -- this only matters if you have changed directories since you've booted your urbit, and you probably haven't.)

Mounting may take a moment.  The `>=` response means the command is successful.  From within Unix (or your favorite file explorer) you should be able to see a `/home` directory in your urbit's pier, and in that directory is `/gen`.  This is where you'll put Hoon source files in order to run them from the dojo.

## List of Numbers

Let's look at a Hoon program that implements a simple function: it takes as input some atom `n` and returns a list of all the atoms from `1` to `n`.  (Remember that an atom is just an unsigned integer.)

Each line is commented with a number for convenient line reference:

```
|=  end=@                                               ::  1
=/  count=@  1                                          ::  2
|-                                                      ::  3
^-  (list @)                                            ::  4
?:  =(end count)                                        ::  5
  ~                                                     ::  6
:-  count                                               ::  7
$(count (add 1 count))                                  ::  8
```

Save this as 'listnum.hoon' in `/home/gen` of your urbit's pier.  To run it enter `+listnum 25` in the dojo:

```
> +listnum 25
~[1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24]
```

### Function Programming, Runes, and Whitespace

Before going over how the above code works we should point out three things.

First, when Listnum is written in a non-functional language it's typically simpler to generate the output as a side effect of the program, e.g., with a `printf()` command inside a loop.  Hoon is a functional language, so the code above instead defines a function.  Functions don't have side effects.  The `listnum` function takes an atom as its input, and then returns an entire list of atoms as a single piece of data.

Second, one syntactic feature of Hoon that distinguishes it from other programming languages is its use of _runes_ rather than keywords.  A rune is just a pair of non-alphanumeric ASCII characters.  Below we give a brief explanation of each of the runes used so you can have a rough idea of how the program works.

Finally, it's important to know that whitespace is more strictly regulated in Hoon than it is in most other languages.  Attempting to run a Hoon program with incorrect spacing results in a syntax error.  The spacing rules will be explained in the next lesson.

### High Level Overview

The program returns a list of numbers, `(list @)`, from `1` up to `end`.  But what do we mean by a 'list'?

There are two kinds of lists in Hoon: empty and non-empty.  The empty list has nothing in it; it's just the null value, `~`.  A non-empty list has at least one item and is a cell of the following sort:

```
[item-1 rest-of-list]
```

That is, the head of the cell is the first list item, and the tail is the rest of the list.  The tail is itself another list.  In Hoon, the end of a list is indicated with the null character, `~`.  Accordingly, we say that lists in Hoon are 'null-terminated'.

Below are binary trees representing a two-item list and a four-item list, respectively:

```
two items:  [item-1 [item-2 ~]]

            .
           / \
     item-1   .
             / \
       item-2   ~

four items:  [item-1 [item-2 [item-3 [item-4 ~]]]]

            .
           / \
     item-1   .
             / \
       item-2   .
               / \
         item-3   .
                 / \
           item-4   ~
```

So, for example, the list of atoms `~[1 2 3 4]` is structured as the following noun:

```
   [1 [2 [3 [4 ~]]]]

          .
         / \
        1   .
           / \
          2   .
             / \
            3   .
               / \
              4   ~
```

The program is to generate the appropriate list of numbers given an input value, `end`: `~[1 2 3 ...]`.  How?  

Essentially, the program uses a loop to produce the list.  Assuming the initial value of `end` is `> 1`, a cell is created in which the head is the value `count`.  The tail is the rest of the list, itself a list.  It's created by incrementing `count` by `1`, and then looping and running the same code again.  When `count` equals `end`, the list is complete and the program ends.

Let's look at the code line-by-line.

### Listnum.hoon: Code explanation

```
|=  end=@                                               ::  1
=/  count=@  1                                          ::  2
|-                                                      ::  3
^-  (list @)                                            ::  4
```

Line 1 uses the gate rune, `|=`, to define a function that takes one argument (i.e., input value).  The argument labelled `end` is stored as the gate's sample; and the sample must be of the type `@`, an atom.  The rest of the program determines the value to be returned by the function.

Line 2 uses `=/` to put an atom into the subject, `1`, labelled `count`.  This is analogous to defining a variable and giving it an initial value in other languages.

Line 3 uses `|-` to indicate a loop starting point.  We'll give `|-` a more precise definition later in the chapter.

Line 4 uses a 'cast' rune, `^-`, to define the product of the loop as a `(list @)`, i.e., as a list of atoms.  If the rest of the program following `(list @)` doesn't provably evaluate to a list of atoms, the program will fail to compile.

```
?:  =(end count)                                        ::  5
  ~                                                     ::  6
```

The `?:` rune in line 5 is an ordinary "if-then-else" conditional.  If the expression that comes directly after it, `=(end count)`, evaluates to true, then the expression on line 6 is evaluated; otherwise the code starting at line 7 is evaluated.  `=(end count)` itself is nothing more than a test of equality on `end` and `count`.

If the values of `count` and `end` are equal then line 6 produces the
value `~`, null.  This indicates the end of the list.  When `count` is equal to `end`, the program is finished.

Otherwise...

```
:-  count                                               ::  7
$(count (add 1 count))                                  ::  8
```

If not, line 7 uses the rune `:-` to produce a cell.  Why is the program producing a cell here?  Remember that cells are used to represent non-null lists in Hoon.  The head of the cell is the value `count` and the tail is the product of evaluating `$(count (add 1 count))`.  

The product of the `$( )` call on line 8 is another list.  It essentially says that the value of `count` is to be replaced by `(add 1 count)`, and that the code starting at `|-` is to be run again.

And that's it!
