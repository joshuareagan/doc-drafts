---
navhome: /docs/
sort: 5
next: true
title: Structures and Data Validation
---

# Structures and Data Validation

Up to this point we've only talked about relatively simple data types.  So far you've seen (1) the basic types and (2) one way of combining basic types to make non-recursive complex types.  In this lesson you'll learn other ways to create custom data types in Hoon.  Data types in Hoon are called *structures*. (Vocab needs work here.)

You'll also learn how you can use structures to validate untrusted data.

## Review

Let's review (1) and (2) types briefly.

### Basic Types

The basic types of Hoon are: `*` for nouns, `@` for atoms (possibly with aura information, e.g., `@ud` and `@sx`), `^` for cells, `?` for flags, and `~` for null.  You can also make constant, one-value types by using `%` followed by a series of lowercase letters, the hyphen symbol `-`, and numbers.  E.g., `%red`, `%2`, `%kebab-case123`.  The lone values of these one-value types are sometimes called 'tags'.

```
> `%blah`%blah
%blah

> `%blah`%not-blah
nest-fail
```

### Complex Types

In certain parts of Hoon code (not all) you can define a complex cell type from simpler types using square brackets, `[ ]`.  For some examples: `[@ ^]`, `[@ud @ub]`, `[@t [? ^]]`, `[%employee @t ? @]` etc.

```
> `[@ud @ub]`[12 0b11]
[12 0b11]

> `[@ud @ub]`[12 14]
nest-fail

> `[%employee @t ? @]`[%employee 'Bob Smith' %.y 123]
[%employee 'Bob Smith' %.y 123]

> `[%employee @t ? @]`[%employer 'John Williams' %.n 321]
nest-fail
```

Cell structures with a constant type at the head are sometimes described as 'tagged'.

## Building Structures: The `$` Rune Family

The purpose of the `$` family of runes is to construct user-defined complex structures from simpler ones.  In the following you'll learn how to use many of these runes, as well as some common ways in which the resulting structures are used in Hoon programming.

### `$:` Build a Cell Structure

The `$:` takes two subexpressions, each of which must be a structure.  The result is a cell structure whose head and tail are the two structures indicated.  For example, the type defined by `$:(@ ?)` is the set of cells whose head is an atom, `@` and whose tail is a flag, `?`.  Most of the time you can achieve the same result using square brackets: `[@ ?]`.

```
> `$:(@ ?)`[123 %.y]
[123 %.y]

> `$:(@ ?)`[%.y 123]
nest-fail

> `$:(%yes ?)`[%yes %.n]
[%yes %.n]

> `$:(%yes ?)`[%no %.n]
nest-fail
```

#### Two Parsing Modes: Pattern vs. Value

Why are `$:(@ ?)` and `[@ ?]` the same only 'most of the time'?  Why not always?  The answer has to do with the way Hoon handles certain expressions.  Some subexpressions of certain runes are reserved exclusively for type information.  For example, the `^-` rune is always followed by two subexpressions, the first of which must always indicate a type.  Subexpressions reserved for type information are parsed by Hoon in 'pattern' mode.  The result of an expression parsed in pattern mode is *always* a structure.  When `[@ ?]` is parsed in pattern mode, the result is structure equivalent to that of `$:(@ ?)`.

The first subexpressions after the `^+` and `|=` runes are also parsed in pattern mode.  But most expressions of Hoon are parsed in 'value' mode.  Expressions parsed in value mode can be evaluated to any kind of noun.  A pair of types in square brackets parsed in value mode won't evaluate to a structure.  It'll evaluate to something not equivalent to its `$:` analogue.  E.g., `[@ ?]` parsed in value mode isn't equivalent to `$:(@ ?)`.  `$:` expressions always evaluate to structures.  You can force a value mode expression to be evaluated in pattern mode by preceding it with `,`: `,[@ ?]`.  However, there usually is no need for this.

You'll learn more about the difference between value mode and pattern mode later in this lesson, in the section on data validation and default values.

### `$-` Define a Gate Type

You can use `$-` to define a gate type.  The `$-` rune takes two subexpressions, which correspond to the gate input type and output type, respectively.  For example, if you want to cast for a gate that takes `@` and returns `@` when called, use `$-(@ @)`:

```
> (`$-(@ @)`|=(@ `@`15) 12)
15

> (`$-(@ @)`|=(@ `*`15) 12)
nest-fail

> `$-(@ @)`add
nest-fail

> `$-([@ @] @)`add
< 1|xpc
  { {@ @}
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>

> `$-([@ @] ?)`gte
< 1|uxz
  { {@ @}
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>
```

### `$?` Define a Union

In set theory, the [union](https://en.wikipedia.org/wiki/Union_(set_theory)) of sets A and B is a set containing all members of both A and B.  For example, the union of the set of even integers and the set of odd integers is the set of all integers.

It's useful to be able to define a type that is the union of other types.  The `$?` rune lets us do this.  The `$?` takes a series of subexpressions, each of which must be a structure.  The resulting type is the union of all types indicated in the expression.  For example, `$?(%green %yellow %red)` is the union of the types indicated by `%green`, `%yellow`, and `%red`.

```
> `$?(%green %yellow %red)`%green
%green

> `$?(%green %yellow %red)`%yellow
%yellow

> `$?(%green %yellow %red)`%red
%red

> `$?(%green %yellow %red)`%blue
nest-fail
```

The irregular form of `$?` is just `?( )`:

```
> `?(%green %yellow %red)`%green
%green

> `?(%green %yellow %red)`%yellow
%yellow
```

`$?` should only be used on types that are disjoint, i.e., which have no values in common.  For example, it shouldn't be used on atom types differing only in aura.

```
> `?(@ud @ux)`10
10

> `?(@ud @ux)`0x12
18
```

Notice that in the latter cast, the type of the literal `0x12` is ignored and the value is printed as `@ud`.  To avoid this when using `$?`, make sure you use types with no values in common.

### `$%` Define a Tagged Union

A [tagged union](https://en.wikipedia.org/wiki/Tagged_union) is a structure corresponding to a union of tagged cell types.  A tagged cell is a cell whose head is a tag, e.g., `[%employee name='John Smith' full-time=%.y]`.  The type of this cell is `[%employee name=@t full-time=?]`, and you may want a union of this type and another tagged cell type, `[%customer name=@t]`.  To construct such a type, use `$%`.

The `$%` rune takes a series of subexpressions, each of which must define a tagged cell type.  The result is a tagged union.  For example, we can make a tagged union of the tagged cell types above using `$%`.  We'll store this structure using the dojo `=` operation (not a feature of Hoon):

```
> =user $%([%employee name=@t full-time=?] [%customer name=@t])

> `user`[%employee 'John Smith' &]
[%employee name='John Smith' full-time=%.y]

> `user`[%employee 'Ryan Jones' |]
[%employee name='Ryan Jones' full-time=%.n]

> `user`[%customer 'Sally Williams']
[%customer name='Sally Williams']

> `user`[%burglar 'Hamburglar']
nest-fail
```

`$%` is especially useful when you need a composite type of various categories, each of which has a different structure.  There is to be a unique tag for each category, which your Hoon program will use to determine how to handle the data appropriately.

### `$@` Define a Union of Atom and Cell Types

The `$@` rune takes two subexpressions.  The first must be an atomic type, and the second must be a cell type.  The result is a union.

Often `$@` is used to define a type with a null or trivial case for the `@` case.  For example, we can expand the `user` structure using `$@` to give it a null case:

```
> =user $@(~ $%([%employee name=@t full-time=?] [%customer name=@t]))

> `user`[%employee 'Ryan Jones' |]
[%employee name='Ryan Jones' full-time=%.n]

> `user`[%customer 'Sally Williams']
[%customer name='Sally Williams']

> `user`~
~
```

Now the `~` value is included as a possible value for `user`.

## Validating Data, Default Values

Hoon structures can be used to validate untrusted data.  To understand how, we need to revisit the difference between 'pattern mode' and 'value mode'.  Remember that certain subexpressions for types are parsed by Hoon in pattern mode, meaning that the result of this expression must be a structure.

What happens when a simple structure is evaluated in value mode?  Check for yourself in the dojo, using `*`, `@`, and `^`:

```
> *
< 1.fel
  { *
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>

> @
< 1.nsr
  { *
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>

> ^
< 1.cgx
  { *
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>
```

You should recognize this pattern -- in all three cases, the result of evaluation is a gate!  In all three cases, the head is a single arm, signified by `1.xxx`.  The sample -- i.e., the head of the tail -- in each case is of the type `*`.  Whenever these structures are evaluated in value mode, they produce special gates called 'molds'.  Molds are gates with two special properties: (1) they accept any noun as the sample, and (2) they are [idempotent](https://en.wikipedia.org/wiki/Idempotence).  A gate `i` is idempotent if and only if the result of applying `i` multiple times to a value produces the same result as applying it once.  In other words, `(i val)` must produce the same result as `(i (i val))`.

The value produced by a mold is always of the type of that mold's originating structure.  We can see this by calling these molds:

```
> (@ 123)
123

> (@ 'hello')
478.560.413.032

> (@t 'hello')
'hello'

> (^ [12 14])
[12 14]

> (? %.y)
%.y
```

But what if we call a mold with the 'wrong' type of sample?  The mold will instead produce a default value of the correct type, sometimes called a 'bunt' value:

```
> (@ [12 14])
0

> (^ 12)
[0 0]

> (? 123)
%.y
```

Remember how we said that `[@ ?]` isn't always equivalent to `$:(@ ?)`?  We can see this when we evaluate each as value mode expressions:

```
> $:(@ ?)
< 1.bli
  { *
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>

> [@ ?]
[ < 1.nsr
    { *
      {our/@p now/@da eny/@uvJ}
      $~
      <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
    }
  >
  < 1.bbx
    { *
      {our/@p now/@da eny/@uvJ}
      $~
      <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
    }
  >
]
```

It turns out that any structure evaluated in value mode produces a mold, even complex structures.  `$:(@ ?)` evaluates to a gate, as you can see above.  But `[@ ?]` produces a pair of gates; in fact, the pair of molds for `@` and `?` respectively.  You can force Hoon to evaluate an expression in pattern mode by preceding it with `,`.  As a result, we can use `,[@ ?]` to make a mold equivalent to the one for `$:(@ ?)`:

```
> ,[@ ?]
< 1.bli
  { *
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>
```

For any mold you can produce its default value by preceding it with `*`:

```
> *^
[0 0]

> *?
%.y

> *@
0

> *@ux
0x0

> *$:(@ ?)
[0 %.y]
```

Say you have a complex structure and you use it to produce a mold.  What happens if you call that mold with data that partially matches the type, but not completely?  Let's use the `user` type in the dojo to see:

```
> =user $%([%employee name=@t full-time=?] [%customer name=@t])

> user
< 1.hly
  { *
    {our/@p now/@da eny/@uvJ}
    $~
    <16.zwv 19.rrs 41.gam 112.cpm 224.nab 54.tyv 119.wim 31.ohr 1.jmk $143>
  }
>

> (user [%employee 'Jake Simms' 123])
[%employee name='Jake Simms' full-time=%.y]

> (user [%employee [12 14] %.n])
[%employee name='' full-time=%.n]
```

First, we define `user` as a tagged union.  Next, we evaluate `user` in the dojo to see that it does indeed produce a gate.  Then we call `user` as a gate with a noun of the wrong type.  `[%employee 'Jake Simms' 123]` is wrong because `123` isn't a flag, `?`.  As a result, the `123` is discarded and replaced with the default value for `?`, `%.y`.  `[%employee [12 14] %.n]` is wrong because instead of a cord, `@t` for the `name`, a cell `[12 14]` is given.  As a result, the cell is discarded and replaced with a null cord, `''`.

We can use molds to validate untrusted data by comparing the unvalidated data with the result of passing that data to the appropriate mold.  If they match, the data is of the correct type.  Otherwise, not.  Let's look at a simple generator that validates nouns of the type `user`:

```
|=  a=*
^-  ?
=/  user  $%([%employee name=@t full-time=?] [%customer name=@t])
=/  b  (user a)
=(a b)
```

Save this as `validate.hoon` in your urbit's pier and try it in the dojo.  It evaluates to `%.y` if the noun passed is of the correct type, and to `%.n` otherwise:

```
> +validate [%employee 'Jake Simms' 123]
%.n

> +validate [%employee 'Jake Simms' %.y]
%.y

> +validate [%customer 'Jake Simms' %.y]
%.n

> +validate [%customer 'Jake Simms']
%.y
```
