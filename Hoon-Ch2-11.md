---
navhome: /docs/
sort: 11
next: true
title: Examples
---

# Examples

Let's look at two more examples of Hoon code: the Sieve of Eratosthenes (a naive prime sieve) and a tic-tac-toe engine.

## Sieve of Eratosthenes

The following program produces the list of primes less than or equal to its argument, an atom:

```
|=  thru=@                                              ::  1
^-  (list @)                                            ::  2
=/  field=(set @)  (sy (gulf 2 thru))                   ::  3
|^  (sort ~(tap in sieve) lth)                          ::  4
++  sieve                                               ::  5
  =/  factor=@  2                                       ::  6
  |-  ^-  (set @)                                       ::  7
  ?:  (gth (mul factor factor) thru)                    ::  8
    field                                               ::  9
  $(factor +(factor), field (reap factor))              ::  10
::                                                      ::  11
++  reap                                                ::  12
  |=  factor=@                                          ::  13
  =/  count=@  (mul 2 factor)                           ::  14
  |-  ^-  (set @)                                       ::  15
  ?:  (gth count thru)                                  ::  16
    field                                               ::  17
  %=  $                                                 ::  18
    count  (add count factor)                           ::  19
    field  (~(del in field) count)                      ::  20
  ==                                                    ::  21
--                                                      ::  22
```

Save it as `sieve.hoon` in `/gen` of your urbit's pier and run in the dojo:

```
> +sieve 50
~[2 3 5 7 11 13 17 19 23 29 31 37 41 43 47]
```

### Sieve Explanation

```
|=  thru=@                                              ::  1
^-  (list @)                                            ::  2
=/  field=(set @)  (sy (gulf 2 thru))                   ::  3
```

Line 1: build a gate (function) whose sample (argument) is `thru`, an atom.

Line 2: the product of this gate is a list of atoms.

Line 3: introduce a face (variable) `field`, whose type is a set of atoms.  `gulf` creates a list of atoms, from `2` to `thru` inclusive.  The `sy` function converts this list to a set of those same atoms.

```
|^  (sort ~(tap in sieve) lth)                          ::  4
++  sieve                                               ::  5
               ...
++  reap                                                ::  12
               ...
--                                                      ::  22
```

Before explaining line 4, let's first talk about what the `|^` rune is for.  It creates a many-armed core, the first of which is named `$` and evaluated automatically.  The first subexpression after the `|^` rune defines the `$` arm.  This is followed by a series of arm definitions which is terminated with `--`.

Line 4: introduce a core which does the desired work.  The `$` arm is defined by the expression `(sort ~(tap in sieve) lth)`.  The `sieve` arm of the core evaluates to a set of prime numbers in `field`.  `tap` of `in` converts this set to a list, and `sort` arranges this list from least to greatest.

```
++  sieve                                               ::  5
  =/  factor=@  2                                       ::  6
  |-  ^-  (set @)                                       ::  7
  ?:  (gth (mul factor factor) thru)                    ::  8
    field                                               ::  9
  $(factor +(factor), field (reap factor))              ::  10
```

Lines 5-10 define the `sieve` arm of the `|^` core.  This arm works as a loop, sieving out non-primes from `field` on every pass until no primes are left.

Line 6: the `factor` face (variable) is given an initial value of `2`.

Line 7: indicates a recursion start point, i.e., the loop start point.  The product of the loop is cast to a `(set @)`, i.e., a set of atoms.

Lines 8-9: if `(mul factor factor)` is greater than `thru` then there are no more primes in `field`.  In that case, return `field`.

Line 10:  Otherwise there may still be primes in `field`.  Recurse (i.e., loop back to `|-`) but with `factor` increased in value by `1` and with the value of `field` replaced with `(reap factor)`.  What does `reap` do?

```
++  reap                                                ::  12
  |=  factor=@                                          ::  13
  =/  count=@  (mul 2 factor)                           ::  14
  |-  ^-  (set @)                                       ::  15
  ?:  (gth count thru)                                  ::  16
    field                                               ::  17
  %=  $                                                 ::  18
    count  (add count factor)                           ::  19
    field  (~(del in field) count)                      ::  20
  ==                                                    ::  21
```

Lines 12-21 define the `reap` arm of the core.  This arm removes all multiples of `factor` from field except `factor` itself.

Line 13 produces a gate (function) whose sample (argument) is an atom with the face `factor`.

Line 14: sets the face (variable) `count` to the initial value of `(mul 2 factor)`.

Line 15: indicates a recursion start point, i.e., a loop start point.  The value produced by the loop is to be a `(set @)`, i.e., a set of atoms.

Lines 16-17: if `count` is greater than `thru`, return `field`.

Lines 18-21: otherwise, recurse (i.e. loop back to `|-`), but with the value of `count` changed to `(add count factor)` and `field` changed to `(~(del in field) count)`.  `(~(del in field) count)` removes the atom `count` from `field`, if it's there.

## Tic-tac-toe

```
 x |   |
-----------
   | x |
-----------
 o | x | o
```

The `toe` tic-tac-toe program below takes a list of tic-tac-toe moves -- for example the move `[%x %2 %1]` indicates an `x` in the bottom-center square -- and returns and end-game result, e.g., `%x-wins`.

```
=<  |=  moves=(list move)                               ::  1
    ^-  outcome                                         ::  2
    ?~  moves  ~                                        ::  3
    =^  out  game  (step i.moves)                       ::  4
    ?~  out  $(moves t.moves)                           ::  5
    out                                                 ::  6
::                                                      ::  7
=>  |%                                                  ::  8
    +$  move  [=player =spot]                           ::  9
    +$  player  ?(%x %o)                                ::  10
    +$  spot  [x=num y=num]                             ::  11
    +$  num  ?(%1 %2 %3)                                ::  12
    +$  outcome  $?  ~                                  ::  13
                     %x-wins                            ::  14
                     %o-wins                            ::  15
                     %tie                               ::  16
                 ==                                     ::  17
    --                                                  ::  18
::                                                      ::  19
|_  [board=(map spot player) last=?]                    ::  20
++  game  .                                             ::  21
::                                                      ::  22
++  step                                                ::  23
  |=  mov=move                                          ::  24
  ^-  [outcome _game]                                   ::  25
  ?:  ?|  &(last =(player.mov %o))                      ::  26
          &(!last =(player.mov %x))                     ::  27
      ==                                                ::  28
    ~&(%move-order !!)                                  ::  29
  ?:  (~(has by board) spot.mov)  ~&(%spot-taken !!)    ::  30
  =.  board  (~(put by board) [spot.mov player.mov])    ::  31
  [outcome-check game(last !last)]                      ::  32
::                                                      ::  33
++  outcome-check                                       ::  34
  ?:  win-check                                         ::  35
    ?:(last %x-wins %o-wins)                            ::  36
  ?:(tie-check %tie ~)                                  ::  37
::                                                      ::  38
++  win-check                                           ::  39
  =/  who=player  ?:(last %x %o)                        ::  40
  %+  lien  winning-rows                                ::  41
  |=  a=(list spot)                                     ::  42
  %+  levy  a                                           ::  43
  |=  b=spot                                            ::  44
  =/  c=(unit player)  (~(get by board) b)              ::  45
  ?~(c | =(who u.c))                                    ::  46
::                                                      ::  47
++  tie-check                                           ::  48
  =(~(wyt in board) 9)                                  ::  49
::                                                      ::  50
++  winning-rows                                        ::  51
  ^-  (list (list spot))                                ::  52
  :~  ~[[%1 %1] [%2 %1] [%3 %1]]                        ::  53
      ~[[%1 %2] [%2 %2] [%3 %2]]                        ::  54
      ~[[%1 %3] [%2 %3] [%3 %3]]                        ::  55
      ~[[%1 %1] [%1 %2] [%1 %3]]                        ::  56
      ~[[%2 %1] [%2 %2] [%2 %3]]                        ::  57
      ~[[%3 %1] [%3 %2] [%3 %3]]                        ::  58
      ~[[%1 %3] [%2 %2] [%3 %1]]                        ::  59
      ~[[%1 %1] [%2 %2] [%3 %3]]                        ::  60
  ==                                                    ::  61
--                                                      ::  62
```

Save this as `toe.hoon` in `/gen` of your urbit's pier and run in the dojo:

```
> +toe ~[[%x %2 %2]]
~

> +toe ~[[%x %2 %2] [%o %1 %1] [%x %1 %3] [%o %3 %2] [%x %3 %1]]
%x-wins

> +toe ~[[%x %2 %2] [%o %1 %3] [%x %1 %1] [%o %3 %3] [%x %2 %3] [%o %2 %1] [%x %1 %2] [%o %3 %2] [%x %3 %1]]
%tie
```

### Tic-tac-toe Explanation

```
=<  |=  moves=(list move)                               ::  1
    ^-  outcome                                         ::  2
    ?~  moves  ~                                        ::  3
    =^  out  game  (step i.moves)                       ::  4
    ?~  out  $(moves t.moves)                           ::  5
    out                                                 ::  6
```

Lines 1-6 constitute the main loop of the program.  The game state (`game`) is modified for each `move` of the sample list until either the game ends in a victory or a tie, or there are no more moves to evaluate.

Lines 1-2: the `=<` rune is used to put a core into the subject before running the gate.  (We'll look at that core later in the code, but it's important to keep in mind that data from that core is used in lines 1-6.)

The `|=` rune produces a gate whose sample is `(list move)`, where a `move` represents a single move of tic-tac-toe.  The `^-` is used to cast for an `outcome`.  `move` and `outcome` are custom types defined later in the code.

Line 3: if there are no more moves in `moves`, the game ends without a victory or a tie.

Line 4: additional context is needed to understand this line.

`game`, as defined by code later in the program, is a door, i.e., a many-armed core with a sample.  This door has two pieces of data in its sample: (1) data representing the current game board arrangement, and (2) data representing which player made the last move, `x` or `o`.  We're basically using the `game` core as a state machine, where the core sample is the mutable state.

`step` is a function (defined later) that takes a `move` and returns two things: (1) an `outcome`, and (2) a modified version of `game` with the board modified to account the latest move.

The `=^` rune is used to evaluate the `step` call, setting the product `outcome` of (1) to `out`, and changing the value of `game` to the newer version from (2).

Lines 5-6: if `out` is `~` (i.e., not a victory or a tie), then there is a recursion back to `|-` on the next `move` in the list.  Otherwise, `out` is returned.

```
=>  |%                                                  ::  8
    +$  move  [=player =spot]                           ::  9
    +$  player  ?(%x %o)                                ::  10
    +$  spot  [x=num y=num]                             ::  11
    +$  num  ?(%1 %2 %3)                                ::  12
    +$  outcome  $?  ~                                  ::  13
                     %x-wins                            ::  14
                     %o-wins                            ::  15
                     %tie                               ::  16
                 ==                                     ::  17
    --                                                  ::  18
```

Lines 8-18 use a `|%` core to define all the custom types used in the program.

Line 8: the `=>` rune is used to put the `|%` core into the subject before going on to evaluate something else later in the code.  (This 'something else' is another core for defining `game`.)

Line 9: the `move` type is for cells in which the head is a `player` and the tail is a `spot`.  The `=player` and `=spot` syntax may be unfamiliar to you.  The former is used to indicate that the `player` value should have a `player` face on it, and the latter to indicate that the `spot` value should have the `spot` face on it.  If value `g` is known to be of the type `move`, you can get the `player` value with `player.g`.

Line 10: the `player` type is for the set of two constants: `%x` and `%o`.

Line 11: a `spot` is a pair of `num`s, one for the x coordinate and one for the y coordinate.

Line 12: `num` is for the set of three constants: `%1`, `%2`, `%3`.

Line 13: `outcome` is for the set of four constants: `~`, `%x-wins`, `%o-wins`, and `%tie`.

```
|_  [board=(map spot player) last=?]                    ::  20
++  game  .                                             ::  21
::                                                      ::  22
++  step                                                ::  23
                    ...
++  outcome-check                                       ::  34
                    ...
++  win-check                                           ::  39
                    ...
++  tie-check                                           ::  48
                    ...
++  winning-rows                                        ::  51
                    ...
--                                                      ::  62
```

Lines 20-62 produce a door, i.e., a many-armed core with a sample.  The sample of this core defines the game state, and the arms define the computations for making appropriate changes to that state.  The `game` arm is trivial; it's just the value from `.`.  Remember that `.` resolves to the entire subject, and that the subject of an arm computation is the parent core; hence, `game` is effectively a name of the entire door.

The door has a two-part sample: `board` is a map from `spot` to `player`.  `board` is essentially a record of which moves have already been made.  `last` is a simple flag for keeping track of whose turn it is to make a move.

```
++  step                                                ::  23
  |=  mov=move                                          ::  24
  ^-  [outcome _game]                                   ::  25
  ?:  ?|  &(last =(player.mov %o))                      ::  26
          &(!last =(player.mov %x))                     ::  27
      ==                                                ::  28
    ~&(%move-order !!)                                  ::  29
  ?:  (~(has by board) spot.mov)  ~&(%spot-taken !!)    ::  30
  =.  board  (~(put by board) [spot.mov player.mov])    ::  31
  [outcome-check game(last !last)]                      ::  32
```

Lines 23-32 are for the `step` arm.  This arm is the step function of the state machine.  It takes a `move` and returns a pair of (1) `outcome` and (2) a new state for the `game` door.

Lines 24-25: `mov` is the name of the `move` in the sample.  The cast on line 25 is for a pair of `outcome` and a core of the same type as `game`.  (Remember that `_game` is short for `$_(game)`.)

Lines 26-30: these lines are for checking whether the correct player made the latest move.  If not, the program will crash.

The `?:` conditional on line 26 has a complex condition that spans three lines.  If either of `&(last =(player.mov %o))` or `&(!last =(player.mov %x))` is true, the `true` branch is evaluated: `~&(%move-order !!)`.

Let's look at the first more carefully.  `&( )` is for logical AND, so `&(last =(player.mov %o))` is true if and only if both `last` and `=(player.mov %o)` are true.  When `last` is true, it's `x`'s turn to make a move.  If `o` moves out of turn, then the `!!` rune is used to crash the program.

Line 30: checks whether the latest move is for a `board` square that is already taken.  If `spot.mov` already has an assignment in `board`, the `!!` rune is used to crash the program.

Line 31: modifies `board` by `put`ing the latest move into it.

Line 32: return the pair of `outcome-check` and the `game` core with the `last` flag value flipped.  (Remember that `!` is for logical NOT -- `!last` is opposite in value to `last`.)  `outcome-check` is another arm in the core.

```
++  outcome-check                                       ::  34
  ?:  win-check                                         ::  35
    ?:(last %x-wins %o-wins)                            ::  36
  ?:(tie-check %tie ~)                                  ::  37
```

Lines 34-37 define the `outcome-check` arm, which is used to check the state and see whether a final game outcome has occurred.  First a `win-check` is performed to see whether the latest `move` is a winning one.  If so, that player wins.  Otherwise, a `tie-check` is performed.

```
++  win-check                                           ::  39
  =/  who=player  ?:(last %x %o)                        ::  40
  %+  lien  winning-rows                                ::  41
  |=  a=(list spot)                                     ::  42
  %+  levy  a                                           ::  43
  |=  b=spot                                            ::  44
  =/  c=(unit player)  (~(get by board) b)              ::  45
  ?~(c | =(who u.c))                                    ::  46
```

Lines 39-46 define the `win-check` arm, which is used to see whether the latest `move` is a winning one.  `who` is the last player to make a move.  This arm uses the `lien` and `levy` functions from the Hoon standard library.

`lien` takes a list and a gate that produces a flag, and it applies the gate to each item in the list.  If at least one of the resulting products is `%.y`, then `lien` produces `%.y`; otherwise, it produces a `%.n`.  (You can think of `lien` as being a logical OR for a list.)  For example:

```
> =g `(list @)`~[10 11 12 13]

> =h |=(a=@ (gth a 12))

> (lien g h)
%.y
```

`levy` is the same thing, except that it only evaluates to `%.y` if all applications of the gate to list items result in `%.y`.  (You can think of `levy` as being a logical AND for a list.)  For example:

```
> =g `(list @)`~[10 11 12 13]

> =h |=(a=@ (gth a 12))

> (levy g h)
%.n

> =h |=(a=@ (gth a 7))

> (levy g h)
%.y
```

With those functions in mind, let's get back to the code:

```
++  win-check                                           ::  39
  =/  who=player  ?:(last %x %o)                        ::  40
  %+  lien  winning-rows                                ::  41
  |=  a=(list spot)                                     ::  42
  %+  levy  a                                           ::  43
  |=  b=spot                                            ::  44
  =/  c=(unit player)  (~(get by board) b)              ::  45
  ?~(c | =(who u.c))                                    ::  46
```

Lines 41-46: `winning-rows` is an arm that evaluates to a list of the eight victory conditions for tic-tac-toe.  `lien` is used to determine if at least one of these eight conditions holds for the last player to make a move.

Each victory condition is recorded as a list of spots.  `levy` is used to determine whether the player has his mark on all of those spots.  If so, the last player wins.

The `~(get by board)` call is used to see which player has a mark on the spot in question, `b`.  `get` of `by` returns a `unit`, which means that `c` must be a `(unit player)`, not just a `player`.  We must use `?~` to test whether `c` is null before we can get the raw player data using `u.c`.

If no player has a mark at spot `b`, then the `~(get by board)` call returns `~`, null.

```
++  tie-check                                           ::  48
  =(~(wyt in board) 9)                                  ::  49
```

Lines 48-49 define the `tie-check` arm, which is used to see if the result is a tie game.  The game is tied if all nine `board` squares are taken and no one has won.  Because the `win-check` occurs before the `tie-check`, we only need to determine if all nine `board` squares have been taken.

We use `wyt` of `in` to check how many squares have been taken; if it's `9`, then return `%.y`, otherwise `%.n`.  `wyt` is a function that returns the number of items in a set.  It can be used for maps as well.

```
++  winning-rows                                        ::  51
  ^-  (list (list spot))                                ::  52
  :~  ~[[%1 %1] [%2 %1] [%3 %1]]                        ::  53
      ~[[%1 %2] [%2 %2] [%3 %2]]                        ::  54
      ~[[%1 %3] [%2 %3] [%3 %3]]                        ::  55
      ~[[%1 %1] [%1 %2] [%1 %3]]                        ::  56
      ~[[%2 %1] [%2 %2] [%2 %3]]                        ::  57
      ~[[%3 %1] [%3 %2] [%3 %3]]                        ::  58
      ~[[%1 %3] [%2 %2] [%3 %1]]                        ::  59
      ~[[%1 %1] [%2 %2] [%3 %3]]                        ::  60
  ==                                                    ::  61
```

Lines 51-61 define the `winning-rows` arm, which produces a list of the eight victory conditions for tic-tac-toe.  Each victory condition is a list of three `spot`s.  The `:~` rune is used to produce a list from a series of expressions, terminated with `==`.

Now you know how to write tic-tac-toe in Hoon!

## Exercise: Conway's Game of Life

Conway's Game of Life is a zero-player game played on an `n x n` grid of spaces, each of which is either 'alive' or 'dead'.  Every iteration, some or all of the spaces change from alive to dead or from dead to alive.

Specifically, for each iteration, each dead space becomes alive if and only if it is adjacent to exactly three live spaces.  Each live remains alive if and only if it is adjacent to either two or three live spaces.  In this context, "adjacent to" includes the eight spaces that share either a side or a corner with the space. Thus, in the following map, the center space (dead) is adjacent to exactly three live spaces, so it will be alive in the next iteration.  `#` signifies a live space and `.` signifies a dead space.

```
    ..#
    ...
    ##.
```

1.  Write a function `next-generation` that takes nine flag
    inputs, `nw`, `nc`, `ne`, `cw`, `cc`, `ce`, `sw`, `sc`, and
    `se`.  Produce 'yes' if the center space will be alive in the
    next generation and 'no' if it will be dead.

1.  Write a function `create-board` that takes an integer `n` and
    produces an `n` by `n` grid of flags, (hereafter,
    "board"), initialized to 'no'.  This grid should be
    represented as a list of `n` lists of length `n`.

1.  Write a function `toggle-space` that takes a board, a row
    index, and a column index, and produces a new board,
    identical to the old except that the state of the space
    referred to by the row and column indices has been toggled.

1.  Use the following function `print-board` to pretty-print the
    board.  For bonus points, try to follow what it's doing.

    ```
    |=  board=(list (list ?))
    ^-  tang
    %+  turn  board
    |=  row=(list ?)
    :-  %leaf
    ^-  tape
    %+  turn  row
    |=  space=?
    ?:  space
      '# '
    '. '
    ```

    If you run this function with a board, you'll get a
    pretty-printable object (a `tang`), but it won't be
    pretty-printed.  When running the generator from the dojo,
    run `&tang +generator arguments` to mark the result as a
    `tang`, which will tell the dojo to pretty-print it
    appropriately.

1.  Using the functions created in the previous exercises, create
    a 10 by 10 board, and flip the states of the spaces at
    coordinates (counting from the top left, indexed from 0)
    `(2,1)`, `(3,2)`, `(1,3)`, `(2,3)`, and `(3,3)`.  Use the
    pretty printer to print out the board.  Your output should be
    the following:

    ```
    . . . . . . . . . .
    . . # . . . . . . .
    . . . # . . . . . .
    . # # # . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    ```

1.  Write a function `step` that iterates every space on the
    board.  Print out the new board.  It should look like this:

    ```
    . . . . . . . . . .
    . . . . . . . . . .
    . # . # . . . . . .
    . . # # . . . . . .
    . . # . . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    . . . . . . . . . .
    ```

    >  Note that the spaces on the edge don't have the full nine
    >  neighbors.  For now, assume that they're always dead.

1.  Write a function `life` that runs the above function `n`
    times on the board.  Observe the board as the game
    progresses.  Congratulations, you've written Conway's Game of
    Life in Hoon!

1.  One common way to handle the edge of the board is to pretend
    that it "wraps around", forming a torus.  In other words, say
    that the spaces on the left edge of the map are adjacent to
    the ones on the right edge, and the spaces on the top edge
    are adjacent to the spaces on the bottom edge.  Modify `step`
    to act this way.  Pay special attention to the corners.

1.  Modify `create-board` to use `++og` to randomly initialize
    the board with live or dead spaces.
