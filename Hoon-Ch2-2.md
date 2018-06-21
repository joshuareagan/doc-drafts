---
navhome: /docs/
sort: 2
next: true
title: Syntax
---

# Hoon syntax

The syntax of a programming language determines what counts as admissible code in that language.  In this lesson we'll introduce you to the rules of Hoon's syntax.

There's no denying that Hoon's syntax is a bit weird, especially for those used to working with other programming languages.  Hoon makes heavy use of ASCII symbols, and this can be intimidating to newcomers.  However, we believe that anyone who learns Hoon syntax will find that it has a number of advantages.

## Motivation

Hoon's syntax is designed to address three serious problems in functional syntax design.  These problems are: (1) *terminator piles*, (2) *indentation creep*, and (3) *feature/label confusion*.

### Terminator piles

A *terminator pile* is a series of termination characters required to end some expression in a language.  Think of Lisp's stacks of right parentheses.  For a trivial example, let's say you want to add 22 five times in Lisp using `+`:

```
(+ 22 (+ 22 (+ 22 (+ 22 22))))
```

In a sufficiently complicated expression it's tedious to keep track of all the parentheses in the terminator pile.  You can write terminator piles in Hoon code if you really want to:

```
> (add 22 (add 22 (add 22 (add 22 22))))
110
```

...but you are never forced to do so.  There is always a semantically equivalent alternative syntax without them:

```
> %+  add  22  
  %+  add  22
  %+  add  22
  %+  add  22  22
110
```

### Indentation creep

In some programming languages there is an indentation convention according to which an expression embedded within a parent expression is indented more to the right.  In a sufficiently long and complicated function with many embedded subexpressions, the tendency is for the code to shift more and more to the right.  This tendency is called *indentation creep*, and it can become an irritating problem when working with long functions.  It's true that short functions are better, but long ones sometimes need to be written.

Ordinary indentation conventions give long functions a _diagonal_ shape, which is a poor fit for most editor windows.  Hoon is designed to be written with 'backstep' indentation (explained later in this lesson) so that the code flows _down_, not down and across.

### Feature/label confusion

A language lends itself to *feature/label confusion* (FLC) when it's difficult to distinguish general features of the language from labels in a particular program.  This can be a serious problem other functional programming languages.  In Lisp, for example, a user-defined macro has the same syntax as a function call.  

The worst-case result of FLC is "DSL cancer".  Every source file with this malady is effectively written in its own domain-specific language (DSL).  Learning to read such a file is like learning a language.  One can also describe the result as "write-only code".  User-level macros, operator overloading, and even excessive use of higher-order programming, can all lead quickly to the tragedy of write-only code.

Hoon largely avoids FLC by its use of runes.  Hoon has no user-level macros or operator overloading. It does support higher-order programming; write-only Hoon can certainly be (and has been) written.  However, with a little discipline one can write Hoon without inventing a DSL.

## Hoon Characters

Hoon source files are composed almost entirely of ASCII characters.  Hoon does not accept non-ASCII text in source files except for UTF-8 in quoted strings.  Hard tab characters are illegal.

### Reading Hoon Aloud

Hoon code makes heavy use of non-alphanumeric symbols.  Reading off code aloud using the proper names of ASCII symbols is tedious, so we've mapped syllables to symbols:

```
ace [1 space]    gal <                pal (
bar |            gap [>1 space, nl]   par )
bas \            gar >                sel [
buc $            hax #                sem ;
cab _            hep -                ser ]
cen %            kel {                sig ~
col :            ker }                soq '
com ,            ket ^                tar *
doq "            lus +                tec `
dot .            pam &                tis =
fas /            pat @                wut ?
zap !
```

Learning and using these names of non-alphanumeric symbols is entirely optional; you can become an expert Hoon programmer without them.  Many Hoon programmers find them helpful, however.

Note that the list includes two separate whitespace forms: `ace` for a single space; `gap` is either 2+ spaces or a newline.  In Hoon, the only whitespace significance is the distinction between `ace` and `gap` -- i.e., the distinction between one space and more than one.

## Hoons -- i.e., Expressions of Hoon

An [expression](https://en.wikipedia.org/wiki/Expression_(computer_science)) is a combination of characters that the language interprets and evaluates as producing a value.  Hoon programs are made up entirely of expressions.  Each expression of Hoon is called a "hoon".

We can divide hoons into two categories: (1) base-level hoons and (2) complex hoons.  Base-level hoons are fundamental Hoon expressions that can't be broken down into smaller expressions.  Complex hoons are Hoon expressions made up of smaller expressions (i.e., subexpressions, or 'subhoons').

Every Hoon program, no matter how big, can be understood as a single complex hoon.  Longer programs have more subhoons in them, naturally -- subhoons in subhoons in subhoons, etc. -- but they're all put together in such a way as to constitute one complex hoon.

### Literals

Every piece of data in Hoon is a noun.  A noun is either an atom or a cell.  An atom is an unsigned integer and a cell is a pair of nouns.

One kind of base-level hoon is an atom literal.  A [literal](https://en.wikipedia.org/wiki/Literal_(computer_programming)) is just a notation for representing a fixed value in Hoon.  In lesson 1.2 (link) you learned about the many ways of expressing atoms of various 'auras' with literals.  Every atom literal in Hoon evaluates to itself.  Examples:

```
> 1
1

> 123
123

> 123.456
123.456

> 'hello'
'hello'

> ~zod
~zod

> 0xdead.beef
0xdead.beef

> 0b1101.1001
0b1101.1001
```

Cell literals can be written in Hoon using `[ ]`.  Cell literals are complex hoons because other hoons are used inside the square brackets.  Examples:

```
> [6 7]
[6 7]

> [[12 14] [654 123.456 980]]
[[12 14] 654 123.456 980]

> ['hello' 0xdad]
['hello' 0xdad]

> [123 [~zod ~ten] .1.2837]
[123 [~zod ~ten] .1.2837]
```

### Wings

A wing expression is a series of limb expressions connected with `.` characters.  You learned about these in section 1.4 (link).  A limb expression can be understood as a trivial case of a wing expression; there is only one limb in the series.  Such trivial wing expressions are base-level hoons.  Some one-limb wings:

- `+2`
- `-`
- `+>`
- `&4`
- `a`
- `b`
- `add`
- `mul`
- `kebab-case-123`

As a special case of a limb we also have `$`, i.e., the name of the arm in special one-armed cores called "gates".  (We covered the role of `$` in lesson 1.5 (link).)

Wing expressions with multiple limbs are complex hoons.  Examples:

- `+2.+3.+4`
- `-.+.+`
- `+>.$`
- `a.b.c`
- `-.b.+2`
- `-.add`

### Type Symbols

In lessons 1.2 and 1.3 (links) we introduced Hoon's type system.  In so doing we used special symbols to indicate certain fundamental types of Hoon: `*` (noun), `@` (atom, sometimes with an aura included, e.g., `@ud`), `^` (cell), and `?` (flag, i.e., Hoon's version of a boolean).  Each of these symbols can be used as a standalone base-level hoon.

They may also be put in brackets to indicate compound types, e.g., `[@ ^]`, `[@ud @sb]`, `[? *]`.  (Technically these complex hoons don't _always_ indicate compound types.  In certain contexts they're interpreted in a different way.  We'll address this variation of meaning when we cover the type system in more detail.)

### Rune Expressions

A *rune* is just a pair of ASCII symbols (a digraph).  You've already seen some runes in Chapter 1: `^-`, `|=`, and `|%`.  We usually pronounce runes by combining their symbols' names, e.g.: 'ket-hep' for `^-`, 'bar-tis' for `|=`, and 'bar-cen' for `|%`.  These names are, of course, entirely optional.

The vast majority of runes are used to indicate the beginning of a complex hoon.  We'll call such hoons 'rune hoons'.  (Not all runes begin expressions: `--` and `==` are used to indicate the end of hoons of indeterminate length.)

Runes are classified by family.  The first of the two symbols indicates the family -- e.g., the `^-` rune is in the `^` family of runes, and the `|=` and `|%` runes are in the `|` family.  The runes of particular family usually have related meanings.  Two simple examples: the runes in the `|` family are all used to create cores, and the runes in the `:` family are all used to create cells.  

Rune hoons are complex, which means they have one or more subhoons.  The appropriate syntax varies from rune to rune; after all, they're used for different purposes.  To see the correct syntax of a particular rune, consult the rune reference (link).  Nevertheless, there are some general principles that hold of all rune hoons.

#### Tall and Flat Forms

There are two rune syntax forms: *tall* and *flat*.  Tall form is usually used for multi-line hoons, and flat form is used for one-line hoons.  Most runes can be used in either of tall or flat forms.  Tall form hoons may contain flat form subhoons, but flat form hoons may not contain tall form subhoons.

The spacing rules differ in the two forms.  In tall form, every part of the hoon must be separated from all others by a `gap` -- that is, by two or more spaces, or a line break.  In flat form the rune is immediately followed by parentheses `( )`, and the various parts of the hoon must be separated from the others by an `ace` -- that is, by a single space.

Seeing an example will help you understand the difference.  The `:-` rune is used to produce a cell.  Accordingly, it is followed by two subhoons: the first defines the head of the cell, and the second defines the tail.  Here are three different ways to write a `:-` hoon in tall form:

```
> :-  11  22
[11 22]

> :-  11
  22
[11 22]

> :-
  11
  22
[11 22]
```

These all do the same thing.  The first example shows that, if you really want to, you can write a 'tall' form hoon on a single line.  Notice that there are two spaces between the `:-` rune and `11`, and also between `11` and `22`.  This is the minimum spacing necessary between the various parts of a tall form hoon -- any fewer will result in a syntax error.

Usually one or more line breaks are used to break up a tall form hoon.  This is especially useful when the subhoons are themselves long stretches of code.  The same `:-` hoon in flat-form is:

```
> :-(11 22)
[11 22]
```

This is the preferred way to write a hoon on a single line.  The rune itself is followed by a set of parentheses, and the subhoons inside are separated by a single space.  Any more spacing than that results in a syntax error.

Nearly all rune expressions can be written in either form, but there are exceptions.  The `|%` and `|_` hoons, for example, can only be written in tall form.  (Those hoons (link) are a bit too complicated to fit comfortably on one line anyway.)


#### Irregular Forms

Some runes are used so frequently that they have irregular counterparts that are easier to write and which mean the same thing.  Irregular rune syntax is hence a form of [syntactic sugar](https://en.wikipedia.org/wiki/Syntactic_sugar).  All irregular rune syntax is flat -- thus, all tall form hoons are regular.

Let's look at another example.  The `.=` rune takes two subhoons, evaluates them, and tests the results for equality.  If they're equal it produces `%.y` for 'yes'; otherwise it produces `%.n` for 'no'.  In tall form:

```
> .=  22  11
%.n

> .=  22  (add 11 11)
%.y

> .=  22
  11
%.n

> .=  22
  (add 11 11)
%.y
```

And in flat form:

```
> .=(22 11)
%.n

> .=(22 (add 11 11))
%.y
```

The irregular form of the `.=` rune is just `=( )`:

```
> =(22 11)
%.n

> =(22 (add 11 11))
%.y
```

The examples above have another irregular form: `(add 11 11)`.  This is the irregular form of `%+`, which calls a gate (i.e., a Hoon function) with two arguments for the sample.

```
> %+  add  11  11
22

> %+(add 11 11)
22
```

We saw this rune earlier in the lesson:

```
> %+  add  22  
  %+  add  22
  %+  add  22
  %+  add  22  22
110
```

The irregular `( )` gate-calling syntax is versatile -- it can also be a shortcut for calling a gate with one argument, which is what the `%-` rune is for:

```
> (dec 11)
10

> %-  dec  11
10

> %-(dec 11)
10
```

The `( )` gate-calling syntax can be used for gates of any other [arity](https://en.wikipedia.org/wiki/Arity) as well.

You've already seen two other irregular hoon forms.  In lesson 1.7 (link), you defined a gate that takes an atom for its sample and returns the atom `15`, regardless of the sample:

```
> |=(a=@ 15)
< 1.ata
  { a/@
    {b/a/@ud our/@p now/@da eny/@uvJ}
    $~
    <16.yvz 19.rrs 41.kdd 112.kgv 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>

> (|=(a=@ 15) 11)
15

> (|=(a=@ 15) 22)
15

> (|=(a=@ 15) 33)
15
```

The `a=@` subhoon -- used to define the face and the type of the gate's sample -- is the irregular form of `$=(a @)`:

```
> (|=($=(a @) 15) 11)
15

> (|=($=(a @) 15) 22)
15

> (|=($=(a @) 15) 33)
15
```

You also defined faces in nouns as in the following examples:

```
> b:[a=15 b=25 c=35]
25

> c:[a=15 b=25 c=35]
35
```

The `a=15` subhoon is the irregular form of `^=(a 15)`:

```
> b:[^=(a 15) ^=(b 25) ^=(c 35)]
25

> c:[^=(a 15) ^=(b 25) ^=(c 35)]
35
```

You can find other irregular forms in the irregular hoon reference document (link).

### Irregular Hoons

There are certain hoons that aren't syntactic sugar for regular form rune hoons -- they're altogether irregular.  There isn't much in general that can be said about these because they're all different, but we can look at some examples.

```
> `12
[~ 12]

> `[12 14]
[~ 12 14]

> `[[12 14] 16]
[~ [12 14] 16]
```

Here we use the `` ` `` symbol to create a cell whose head is null, `~`.

```
> -:[a=[12 14] b=[16 18]]
a=[12 14]

> ,.-:[a=[12 14] b=[16 18]]
[12 14]

> ,.-:[[12 14] b=[16 18]]
[12 14]

> +:[a=[12 14] b=[16 18]]
b=[16 18]

> ,.+:[a=[12 14] b=[16 18]]
[16 18]

> ,.+:[a=[12 14] [16 18]]
[16 18]
```

Putting `,.` in front of a wing removes a face, if there is one.  To see other irregular hoons, check the irregular hoon reference document (link).

## Hoon Style

In this section we'll talk about preferred Hoon code style.  Strictly speaking, all the guidelines below are optional -- if you follow the minimum syntax rules, your code should parse and run correctly.  On the other hand, code clarity and maintainability are important.  We think the following guidelines will help you to write better, clearer code.

### Code Margin, Blank Lines

An 80-column right margin is strongly encouraged.  Really well-groomed Hoon uses a 55-column code margin and puts a standard line comment at column 57.  Use `::` to begin a comment -- Hoon will ignore everything past the `::` until the line break.  Here's an example of code that follows these principles:

```
|=  a=@                                                 ::  even number checker
^-  ?                                                   ::  output is a flag
=(0 (mod a 2))                                          ::  =(...) evals to flag
```

Blank lines are discouraged.  If you want vertical space in your code, use `::` at the beginning of one or more lines.

```
|=  a=@                                                 ::  even number checker
^-  ?                                                   ::  output is a flag
::
::  lots
::  of
::  space
::
=(0 (mod a 2))                                          ::  =(...) evals to flag
```

### Rune Hoon Body Types

Let's call everything in the hoon after the initial rune the hoon *body*. There are four kinds of body: *fixed*, *running*, *jogging*, and *battery*.  There is a preferred manner of styling for each body kind.

#### Fixed

Runes with a *fixed* number of subhoons self-terminate.  For example, the `:-` and `.=` runes each have exactly two subhoons.  

Bodies with a fixed number of subhoons use "backstep" indentation in tall form.  In backstep indentation the code slopes backward and to the right by two spaces per line.  The last subhoon shouldn't be indented at all, and the first subhoon is on the same line as the rune.  The indentation of the first subhoon therefore depends upon the total number of subhoons.

We'll use the `:` family of runes to illustrate.  The `:-` rune creates a cell from two subhoons, `:+` creates a cell from three, and `:^` creates a cell from four.  In flat form:

```
> :-(11 22)
[11 22]

> :+(11 22 33)
[11 22 33]

> :^(11 22 33 44)
[11 22 33 44]
```

Let's look at these in tall form, using backstep indentation:

```
> :-  11
  22
[11 22]

> :+  11
    22
  33
[11 22 33]

> :^    11
      22
    33
  44
[11 22 33 44]
```


Some other *1-fixed*, *2-fixed*, *3-fixed* and *4-fixed* examples, using `p`, `q`, `r`, and `s` for the subhoons:

```
|.
p

=|  p
q

?:  p
  q
r

%^    p
    q
  r
s
```

In optimal usage of backstep indentation the most complicated subhoons are at the bottom.  This helps keep code flow vertical rather than diagonal.

#### Running

A running body doesn't have a fixed number of subhoons; hoons with running bodies can be arbitrarily long.

To indicate the end of such hoons in tall form, use the `==` rune.  The first subhoon is two spaces after the initial rune, and all other subhoons are indented to line up with the first.  The terminating `==` should align vertically with the initial rune.  In flat form, the parentheses are used to indicate when the hoon is finished, so the `==` is unnecessary.

The `:*` rune is used to create a cell of arbitrary length:

```
> :*(11 22 33)
[11 22 33]

> :*(11 22 33 44)
[11 22 33 44]

> :*(11 22 33 44 55)
[11 22 33 44 55]
```

We'll use `:*` to illustrate proper tall form running body style:

```
> :*  11
      22
      33
      44
      55
  ==
[11 22 33 44 55]
```

More generally:

```
:*  p
    q
    r
    s
    t
==
```

#### Jogging

A jogging body is an arbitrarily long series of subhoon pairs.  Most jogging bodies are preceded by a fixed sequence of subhoons.  The first jogging pair is one line after the last fixed subhoon, and indented two spaces behind it.  There are two jogging conventions: *flat* (each pair on one line) and *tall* (each pair split across lines).  In either case the hoon is terminated with a `==`.

The `?-` rune is `1-fixed` followed by a jogging body.  The fixed subhoon is evaluated and then compared against the left subhoon of each of the jogging pairs.  When a match is found, the subhoon to the right of the match is evaluated.

For example:

```
> ?-  `?(%2 %3 %4)`%2
    %2  %yes
    %3  %no
    %4  %no
  ==
%yes
```

More generally, the *flat jogging* style (preceded by *1-fixed* `p`):

```
?-  p
  q  r
  s  t
  u  v
==
```

A *tall jogging* example (preceded by *1-fixed* `p`):

```
?-    p
    q
  r
    s
  t
    u
  v
==
```

In flat form for *jogging* bodies, subhoon pairs are separated by commas:

```
$(a +(a), b (dec b), c (add 2 c))
```

#### Battery

Certain runes are used to define a core.  Cores with multiple arms have a special syntax for defining each of the arms in the battery.  A core has no fixed limit on the number of arms contained in its battery, so you must terminate core hoons with the `--` rune.  Each arm in the battery is defined with three things: (1) a rune in the `+` family (e.g., `++`, `+*`, and `+$`), (2) and arm name (ordinary name restrictions apply; see lesson 1.4 (link)), and (3) a subhoon defining the content of that arm.  Runes in the `+` may only go in core hoons.  Putting them anywhere else results in a syntax error.

Here's an example of a battery body:

```
|%
++  arm-1
  %p
++  arm-2
  %q
++  arm-3
  %r
--
```
