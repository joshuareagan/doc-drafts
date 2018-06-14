---
navhome: /docs/
sort: 1
next: true
title: Hoon Programs
---

# Writing Hoon Programs

So far you've only entered short snippets of Hoon code into the dojo.  Let's look at some programs!

There many kinds of Hoon programs: generators, Gall apps, Sail documents, and more.  In this chapter we'll work exclusively with a subset of the first kind: generators.  This narrow focus will allow us to set aside complications that aren't necessary for learning the basics of Hoon.  We won't even talk about the full range of generators supported in Urbit just yet.  Other kinds of programs will be covered in later chapters.

## Setup

For now we'll work exclusively with 'naked' generators.  A naked generator is nothing more than a gate that can be run from the dojo and that takes exactly one argument for its sample.  Generators are located in the `/gen` directory of your urbit's filesystem.

To use your favorite text editor to write Hoon programs you'll first need to expose your urbit's filesystem to Unix.  If you haven't already done so, 'mount' your home desk as follows:

```
> |mount %
>=
```

(Note: we are assuming that you have mounted from your 'home' desk in the dojo.  If you don't know what this means, don't worry -- this only matters if you have changed directories since you've booted your urbit, and you probably haven't.)

Mounting may take a moment.  The `>=` response means the command is successful.  From within Unix (or your favorite file explorer) you'll now be able to see a `/home` directory in your urbit's pier, and in that directory you'll find `/gen`.  This is where we'll be putting Hoon source files in order to run them from the dojo.

## Fizzbuzz Demo

We'll start by looking at a Hoon implementation of [FizzBuzz](https://en.wikipedia.org/wiki/Fizz_buzz).  Fizzbuzz takes some number `n` and then makes a list of numbers from `1` to `n`, except that numbers evenly divisible by `3` are replaced with 'Fizz', numbers evenly divisible by `5` are replaced with 'Buzz', and numbers evenly divisible by both are replaced with 'FizzBuzz'.

Each line is commented with its number:

```
|=  end=@                                               ::  1
=/  count=@  1                                          ::  2
|-                                                      ::  3
^-  (list tape)                                         ::  4
?:  =(end count)                                        ::  5
  ~                                                     ::  6
:-                                                      ::  7
  ?:  =(0 (mod count 15))                               ::  8
    "FizzBuzz"                                          ::  9
  ?:  =(0 (mod count 3))                                ::  10
    "Fizz"                                              ::  11
  ?:  =(0 (mod count 5))                                ::  12
    "Buzz"                                              ::  13
  <count>                                               ::  14
$(count (add 1 count))                                  ::  15
```

Save this as 'fizzbuzz.hoon' in `/home/gen` of your urbit's pier.  To run it enter `+fizzbuzz 25` in the dojo:

```
> +fizzbuzz 25
<<
  "1"
  "2"
  "Fizz"
  "4"
  "Buzz"
  "Fizz"
  "7"
  "8"
  "Fizz"
  "Buzz"
  "11"
  "Fizz"
  "13"
  "14"
  "FizzBuzz"
  "16"
  "17"
  "Fizz"
  "19"
  "Buzz"
  "Fizz"
  "22"
  "23"
  "Fizz"
>>
```

### Fizzbuzz as a Function, Runes, and Whitespace

Before we explain how the code works we should point out three things.

First, when FizzBuzz is written in non-functional languages, the output is generated as a side effect of the program, e.g., with `printf()`.  Hoon is a functional language, however, and so the code above defines a fizzbuzz function.  Functions don't have side effects.  Accordingly, we implement the fizzbuzz function so that it returns a list of strings when passed an argument.  In this list each item is either a number, "Fizz", "Buzz", or "Fizzbuzz".

Second, one syntactic feature of Hoon that distinguishes it from other programming languages is its use of _[runes](../../about/glossary#rune)_ rather than keywords.  A rune is just a pair of non-alphanumeric ASCII characters.  Almost all runes are used to begin a Hoon expression.  We'll give a brief explanation of each of the runes used above so you can have a rough idea of how the program works.

Finally, it's important to know that whitespace is more strictly regulated in Hoon than it is in most other languages.  Incorrect spacing is a syntax error.  The spacing rules will be explained in the next lesson.

### Explaining Fizzbuzz

```
|=  end=@                                               ::  1
=/  count=@  1                                          ::  2
|-                                                      ::  3
^-  (list tape)                                         ::  4
```

Line 1 uses the gate rune, `|=`, to define a function that takes one argument -- i.e., the gate's sample.  The sample must be of the type `@`, an atom, and it is given the face `end`.  The rest of the program determines the value to be returned by the function.

Line 2 uses `=/` to put an atom into the subject, `1`, and give it the face `count`.  This is analogous to defining a variable and giving it an initial value in other languages.

In rough terms, line 3 uses `|-` to indicate the starting point of a loop.  We'll give `|-` a more precise definition in later chapters.

Line 4 uses the cast rune, `^-`, to define the product of the loop as a list of `tape`s.  You've first saw this rune in lesson 1.2 -- if the rest of the program following `(list tape)` doesn't provably evaluate to the correct type of value, you'll get a `nest-fail` error.  A `tape` is one of Hoon's two string types; you saw the other type, `cord`, in lesson 1.2 as well.  But whereas a `cord` is an atom, a `tape` is a list of characters.  

```
?:  =(end count)                                        ::  5
  ~                                                     ::  6
```

The `?:` rune in line 5 is an ordinary "if-then-else" conditional.  The expression that comes directly after it, `=(end count)`, evaluates to true, then the expression on line 6 is evaluated; otherwise the code starting at line 8 is evaluated.  `=(end count)` itself is nothing more than a test of equality for `end` and `count`.

If the values of `count` and `end` are equal then line 6 produces the
value `~`.  Recall that the product of the whole function is a list of `tape`s.  In Hoon, we indicate the end of a list with the null character, `~`.  Accordingly, lists in Hoon are 'null-terminated'.  When `count` is equal to `end`, the program is finished.

Otherwise...

```
:-                                                      ::  7
  ?:  =(0 (mod count 15))                               ::  8
    "FizzBuzz"                                          ::  9
  ?:  =(0 (mod count 3))                                ::  10
    "Fizz"                                              ::  11
  ?:  =(0 (mod count 5))                                ::  12
    "Buzz"                                              ::  13
  <count>                                               ::  14
$(count (add 1 count))                                  ::  15
```

If not, line 7 uses the rune `:-` to produce a cell.  Why is the program producing a cell here?  Isn't it supposed to be making a list?  Cells are used to represent lists in Hoon.  There are two kinds of lists: null and non-null.  The null list, `~`, has nothing in it.  Non-null lists have at least one item, and are structured as follows:

```
[item-1 rest-of-list]
```

That is, the head of the cell is the first item in the list, and the tail of the cell is the rest of list.  The tail is itself another list.

```
two items:  [item-1 [item-2 ~]]

three items:  [item-1 [item-2 [item-3 ~]]]

four items:  [item-1 [item-2 [item-3 [item-4 ~]]]]

...
```

So the cell created for our fizzbuzz gate is `[a b]`: (a) an item to be determined by lines 8-14; and (b) the rest of the list, produced by line 15.

For the first value of the cell, (a): The first value of the cell is: (i) "FizzBuzz", or "Fizz", or "Buzz", if the respective conditional tests hit.  For example, if `=(0 (mod count 15))` is true -- i.e., if `count` is evenly divisible by 15 -- then the value returned is "FizzBuzz".  If none of the three test conditions is true, then the value is (ii) the number in question, `count`.  In case (ii), line 14 uses the `< >` symbols to convert `count` from an `@` to a `tape`.

For the second value of the pair, (b): the rest of the output list is produced by line 15.  This line uses `$( )` to loop back to the `|-` in line 3, but only after modifying the value of `count`: `count` is increased in value by `1`.

The product of the `$( )` call on line 15 is another list.  Either the modified `count` equals `end`, in which case the null list, `~`, is returned on line 6; or a new cell is produced in which case the head is an item determined by lines 8-14 and the tail is determined by line 15.  The process repeats until `count` equals `end`.
