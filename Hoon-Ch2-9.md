---
navhome: /docs/
sort: 9
next: true
title: Hoon Standard Library: Lists
---

# Hoon Standard Library: Lists

In this lesson we'll review what a list is and then cover many of the list-related functions in the Hoon standard library.

## List Review

A list is either null or non-null.  The null list is just `~`.  A non-null list is a cell in which the head is the first list item, and the tail is the rest of the list.  The tail is itself a list.  The head has the face `i` and the tail has the face `t`.  (For the sake of neatness, these faces aren't shown by the Hoon pretty printer.)

```
> `(list @)`~
~

> `(list @)`[1 2 3 4 5 ~]
~[1 2 3 4 5]

> `(list @t)`['Urbit' 'will' 'rescue' 'the' 'internet' ~]
<|Urbit will rescue the internet|>
```

To use the `i` and `t` faces of a list, you must first prove that the list is non-null by using the conditional family of runes, `?`:

```
> =>(b=`(list @)`[1 2 3 4 5 ~] i.b)
-find.i.b
find-fork-d

> =>(b=`(list @)`[1 2 3 4 5 ~] ?~(b ~ i.b))
1

> =>(b=`(list @)`[1 2 3 4 5 ~] ?~(b ~ t.b))
~[2 3 4 5]
```

You can construct lists of any type.  `(list @)` indicates a list of atoms, `(list ^)` indicates a list of cells, `(list [@ ?])` indicates a list of cells whose head is an atom and whose tail is a flag, etc.

## Tapes

Hoon has two kinds of strings: cords and tapes.  Cords are atoms with aura `@t`, and they're pretty-printed between `''` marks.

```
> 'this is a cord'
'this is a cord'

> `@`'this is a cord'
2.037.307.443.564.446.887.986.503.990.470.772
```

A tape is a list of `@tD` atoms (i.e., ASCII characters).

```
> "this is a tape"
"this is a tape"

> `(list @)`"this is a tape"
~[116 104 105 115 32 105 115 32 97 32 116 97 112 101]
```

You can also use the words `cord` and `tape` for casting:

```
> `tape`"this is a tape"
"this is a tape"

> `cord`'this is a cord'
'this is a cord'
```

### Caesar Cipher

Let's write a Caesar cipher encryption program in Hoon.  A Caesar cipher substitutes each letter in a message with another letter shifted `n` places in the alphabet.  For example, the Caesar cipher of `"ABC"` with a shift of `1` place is `"BCD"`.

If you like, try to write your own version of this program before looking below.

The following program takes an atom `a` and a tape `b`, and returns a tape that is like `b` except that each of its letters have been shifted `a` places in the alphabet:

```
|=  [a=@ b=tape]                           ::  1
?:  (gth a 25)                             ::  2
  $(a (sub a 26))                          ::  3
|-  ^-  tape                               ::  4
?~  b  ~                                   ::  5
:-                                         ::  6
  ?:  &((gte i.b 'A') (lte i.b 'Z'))       ::  7
    =.  i.b  (add i.b a)                   ::  8
    ?.  (gth i.b 'Z')  i.b                 ::  9
    (sub i.b 26)                           ::  10
  ?:  &((gte i.b 'a') (lte i.b 'z'))       ::  11
    =.  i.b  (add i.b a)                   ::  12
    ?.  (gth i.b 'z')  i.b                 ::  13
    (sub i.b 26)                           ::  14
  i.b                                      ::  15
$(b t.b)                                   ::  16
```

Save this as `caesar.hoon` in `/gen` of your urbit's pier and run in the dojo:

```
> +caesar [4 "test"]
"xiwx"

> +caesar [4 "The quick brown fox jumps over the lazy dog."]
"Xli uymgo fvsar jsb nyqtw sziv xli pedc hsk."
```

Let's look at this program, piece by piece:

```
|=  [a=@ b=tape]                           ::  1
?:  (gth a 25)                             ::  2
  $(a (sub a 26))                          ::  3
```

Line 1 has the `|=` rune for producing a gate and defines the gate's sample as a pair of `a=@` and `b=tape`.  `b` is the message and `a` is the number of places to shift each letter in the alphabet.  The English alphabet has 26 letters, so shifting by 26 places is the same as not shifting at all.  If `a` is greater than 25, we can subtract by 26 and get the same result.

Accordingly, if `a` is greater than `25`, line 3 restarts the function but with the value of `a` set to `(sub a 26)`.

```
|-  ^-  tape                               ::  4
?~  b  ~                                   ::  5
:-                                         ::  6
         ...
$(b t.b)                                   ::  16
```

Above is the body of the program with lines 7-15 taken out.  The `|-` on line 4 indicates a recursion start point; it marks the beginning of the program's main loop.  The output of the function is to be a `tape`.

If the message, `b`, is empty, i.e., a null list `~`, then the program returns a `~`.  The program is finished.

Otherwise `b` isn't empty, in which case the `:-` rune is used to indicate that the function returns a cell.  Remember, a `tape` is a list of `@tD` atoms.  The head of a non-empty list is the first item of the list, in this case the first character of the `tape`.  The tail is the rest of the list as another list.  The tail value is produced by a recursion, where `b` is replaced with the tail of `b`, i.e., `b` without the first character.

Lines 7-15 produce the replacement character for the head of `b`:

```
  ?:  &((gte i.b 'A') (lte i.b 'Z'))       ::  7
    =.  i.b  (add i.b a)                   ::  8
    ?.  (gth i.b 'Z')  i.b                 ::  9
    (sub i.b 26)                           ::  10
  ?:  &((gte i.b 'a') (lte i.b 'z'))       ::  11
    =.  i.b  (add i.b a)                   ::  12
    ?.  (gth i.b 'z')  i.b                 ::  13
    (sub i.b 26)                           ::  14
  i.b                                      ::  15
```

If the head of `b`, `i.b`, is a character between `A` and `Z` (line 7), then we add `a` to it (line 8) to shift it by the correct number of places in the alphabet.  If the result is a character beyond `Z`, then we subtract `26` from `i.b` to shift back through `A` at the beginning of the alphabet (lines 9-10).

Lines 11-14 do the same thing for the case in which `i.b` is a lowercase letter.  If `i.b` is neither uppercase nor lowercase, then we leave it unchanged (line 15).

## List Functions in the Hoon Standard Library

Lists are a commonly used data structure.  Accordingly, there are several functions in the Hoon standard library intended to make lists easier to use.

For a complete list of these functions, check out the standard library reference for lists here (add link later).  Here we'll look at a few of these functions.

### `flop`

The `flop` function takes a list and returns it in reverse order:

```
> (flop ~[11 22 33])
~[33 22 11]

> (flop "Hello!")
"!olleH"

> (flop (flop "Hello!"))
"Hello!"
```

#### `flop` Exercise

Without using `flop`, write a gate that takes a `(list @)` and returns it in reverse order.  There is a solution at the bottom of this lesson.

### `sort`

The `sort` function uses the 'quicksort' algorithm to sort a list.  It takes a list to sort and a gate that serves as a comparator.  For example, if you want to sort the list `~[37 62 49 921 123]` from least to greatest, you would pass that list along with the `lth` gate (for 'less than'):

```
> (sort ~[37 62 49 921 123] lth)
~[37 49 62 123 921]
```

To sort the list from greatest to least, use the `gth` gate ('greater than') as the basis of comparison instead:

```
> (sort ~[37 62 49 921 123] gth)
~[921 123 62 49 37]
```

You can sort letters this way as well:

```
> (sort ~['a' 'f' 'e' 'k' 'j'] lth)
<|a e f j k|>
```

The function passed to `sort` must produce a `flag`, i.e., `?`.

### `weld`

The `weld` function takes two lists and concatenates them:

```
> (weld ~[1 2 3] ~[4 5 6])
~[1 2 3 4 5 6]

> (weld "Happy " "Birthday!")
"Happy Birthday!"
```

### `snag`

The `snag` function takes an atom `n` and a list, and returns the `n`th item of the list, where `0` is the first item:

```
> (snag 0 `(list @)`~[11 22 33 44])
11

> (snag 1 `(list @)`~[11 22 33 44])
22

> (snag 3 `(list @)`~[11 22 33 44])
44

> (snag 3 "Hello!")
'l'

> (snag 1 "Hello!")
'e'

> (snag 5 "Hello!")
'!'
```

Note: there is currently a type system issue that causes some of these functions to fail when passed a list `b` after some type inference has been performed on `b`.  For an illustration of the bug, let's set `b` to be a `(list @)` of `~[11 22 33 44]` in the dojo:

```
> =b `(list @)`~[11 22 33 44]

> b
~[11 22 33 44]
```

Now let's use `?~` to prove that `b` isn't null, and then try to `snag` it:

```
> ?~(b ~ (snag 0 b))
nest-fail             
```

The problem is that `snag` is expecting a raw list, not a list that is known to be non-null.

#### `snag` exercise

Without using `snag`, write a gate that returns the `n`th item of a list.  There is a solution at the bottom of this lesson.

### `oust`

The `oust` function takes a pair of atoms `[a=@ b=@]` and a list, and returns the list with `b` items removed, starting at item `a`:

```
> (oust [0 1] `(list @)`~[11 22 33 44])
~[22 33 44]

> (oust [0 2] `(list @)`~[11 22 33 44])
~[33 44]

> (oust [1 2] `(list @)`~[11 22 33 44])
~[11 44]

> (oust [2 2] "Hello!")
"Heo!"
```

### `lent`

The `lent` function takes a list and returns the number of items in it:

```
> (lent ~[11 22 33 44])
4

> (lent "Hello!")
6
```

### `roll`

The `roll` function takes a list and a gate, and accumulates a value of the list items using that gate.  For example, if you want to add or multiply all the items in a list of atoms, you would use `roll`:

```
> (roll `(list @)`~[11 22 33 44 55] add)
165

> (roll `(list @)`~[11 22 33 44 55] mul)
19.326.120
```

### `turn`

The `turn` function takes a list and a gate, and returns a list of the products of applying each item of the input list to the gate.  For example, to add `1` to each item in a list of atoms:

```
> (turn `(list @)`~[11 22 33 44] |=(a=@ +(a)))
~[12 23 34 45]
```

Or to double each item in a list of atoms:

```
> (turn `(list @)`~[11 22 33 44] |=(a=@ (mul 2 a)))
~[22 44 66 88]
```

`turn` is Hoon's version of Haskell's `map`.

We can rewrite the Caesar cipher program using `turn`:

```
|=  [a=@ b=tape]
^-  tape
?:  (gth a 25)
  $(a (sub a 26))
%+  turn  b
|=  c=@tD
?:  &((gte c 'A') (lte c 'Z'))
  =.  c  (add c a)
  ?.  (gth c 'Z')  c
  (sub c 26)
?:  &((gte c 'a') (lte c 'z'))
  =.  c  (add c a)
  ?.  (gth c 'z')  c
  (sub c 26)
c
```

## Using `hoon.hoon` to Learn Hoon

The Hoon standard library and compiler are written in Hoon.  At this point, you know enough Hoon to be able to explore the standard library portions of `hoon.hoon` and find more functions relevant to lists.  [Look around in section `2b` (and elsewhere)](https://github.com/urbit/arvo/blob/master/sys/hoon.hoon#L459).

## Exercise Solutions

#### `flop`

```
::  flop.hoon
::
|=  a=(list @)
=|  b=(list @)
|-  ^-  (list @)
?~  a  b
$(b [i.a b], a t.a)
```

Run in dojo:

```
> +flop ~[11 22 33 44]
~[44 33 22 11]
```

#### `snag`

```
::  snag.hoon
::
|=  [a=@ b=(list @)]
?~  b  !!
?:  =(0 a)  i.b
$(a (dec a), b t.b)
```

Run in dojo:

```
> +snag [0 ~[11 22 33 44]]
11

> +snag [2 ~[11 22 33 44]]
33
```
