---
navhome: /docs/
navuptwo: true
next: true
sort: 1
title: Technical overview
---

# Technical overview

Urbit is a clean-slate system software stack designed to implement an encrypted P2P network of general-purpose personal servers.  Each server on this network is a deterministic computer called an 'urbit' that runs inside a Unix-based virtual machine.

The current Urbit stack includes (among other things):

- Arvo: the functional operating system of each urbit, written in Hoon.
- Hoon: a high-level strictly typed functional programming language whose standard library includes a Hoon-to-Nock compiler.
- Nock: a low-level combinator language whose formal specification gzips to 340 bytes.
- Vere: a Nock interpreter and Unix-based VM that mediates between each urbit and the Unix software layer.

## Anatomy of a personal server

Your urbit is a deterministic computer in the sense that its state is a pure function of its event history.  Every event in this history is a [transaction](https://en.wikipedia.org/wiki/Transaction_processing); your urbit's state is an [ACID database](https://en.wikipedia.org/wiki/ACID) in a [single-level store](https://en.wikipedia.org/wiki/Single-level_store).

Because each urbit is deterministic we can describe its role in straightforwardly functional terms: it maps an input event and the urbit state to a list of output actions and the subsequent state.  This is the urbit transition function.

```
(input event, old state) -> (output actions, new state)
```

Events can be wholly internal to your urbit (e.g., completion of a computation in a locally-running application) or they can originate from elsewhere (e.g., a packet received from another urbit).

Can events originating from your urbit cause side-effects in the outside world?  The answer had better be "yes," because a personal server without side effects isn't useful for much.  In another sense the answer had better be "no," or else there is a risk of losing functional purity; your urbit can't guarantee that the side effects in question actually occur.  What's the solution?

Each urbit is [sandboxed](https://en.wikipedia.org/wiki/Sandbox_(computer_security)) in a virtual machine, Vere, which runs on Unix.  Code running in your urbit cannot make Unix system calls or otherwise affect the underlying platform.  Strictly speaking, internal urbit code can only change internal urbit state; it has no way of sending events outside of its runtime environment.  Functional purity is preserved.

In practical terms, however, we don't want our urbit to be an impotent brain in a vat.  That's why Vere also serves as the intermediary between your urbit and Unix.  Vere observes the list of output events, and when external action is called for makes the appropriate system calls itself.  When external actions relevant to your urbit occur, Vere encodes and delivers them as input events.

### Arvo

Arvo is a purely functional, [non-preemptive](https://en.wikipedia.org/wiki/Cooperative_multitasking) OS that serves as the event manager of your urbit.  It can upgrade itself and everything inside it from over the network without downtime.  The Arvo kernel proper is quite simple; it's only about 600 lines of Hoon.

The urbit transition function is implemented in Arvo.  Upon being 'poked' by Vere with the pair of (input event, state), Arvo directs the event to the appropriate OS module.  The result is a pair of (output events, new state).  Events are typed, and each has an explicit call-stack structure indicating the event's source module in Arvo.

Arvo modules are also called 'vanes'.  Arvo's vanes are:

- `%ames`: defines and implements Urbit's encrypted P2P network protocol, as well as Urbit's identity protocol.
- `%behn`: manages timer events for other vanes.
- `%clay`: global, version-controlled, and referentially-transparent file system.
- `%dill`: terminal driver.
- `%eyre`: HTTP web client and server.
- `%ford`: typed functional build system.
- `%gall`: application sandbox and manager.
  - Dojo (app): Shell and Hoon REPL.
  - Talk (app): Chat client.

### Hoon

Hoon is a strictly typed functional programming language that compiles itself to Nock and is designed to support higher-order functional programming without requiring knowledge of category theory or other advanced mathematics.  Haskell is fun but it isn't for everybody.

Hoon aspires to a concrete, imperative feel.  To discourage the creation of write-only code, Hoon forbids user-level macros and uses ASCII digraphs instead of keywords.  The type system infers only forward and does not use unification, but is not much weaker than Haskell's.  The compiler and inference engine is about 3000 lines of Hoon.

### Nock

Nock is a low-level [homoiconic](https://en.wikipedia.org/wiki/Homoiconicity) combinator language without lambdas or even syntax.  It's so simple that its [specification](../../nock/definition) fits on a t-shirt.  In some ways Nock resembles a nano-Lisp but its ambitions are more narrow.  Most Lisps are one-layer: they create a practical language by extending a theoretically simple interpreter.  The abstraction is simple and the implementation is practical; but there is no actual Lisp codebase both simple and practical.  Hoon and Nock are two layers: Hoon, the practical layer, compiles itself to Nock, the simple layer.  Your urbit runs in Vere, which is a Nock interpreter, not a Hoon interpreter, so it can upgrade Hoon over the network without downtime.

The Nock data model is quite simple.  All data are structured as 'nouns'.  A noun is an atom or a cell.  An atom is any unsigned integer.  A cell is an ordered pair of nouns.  Nouns are acyclic and expose no pointer equality test.

### Vere\*

Vere is the Nock runtime environment and Urbit VM.  It's written in C, runs on Unix, and is the intermediate layer between your urbit and Unix.  As noted earlier, Unix system calls are made by Vere, not Arvo; Vere must also encode and deliver relevant external events to Arvo.  Vere is also responsible for implementing jets and maintaining the persistent state of each urbit.

In principle, Vere keeps a comprehensive log of every event from the time you initially booted your urbit.  What happens if the physical machine loses power and your urbit's state is 'lost' from memory?  When your urbit restarts it will replay its entire event history and totally recover its latest state from scratch.

In practice, event logs become large and unwieldy.  Periodically a snapshot of the permanent state is taken and the logs are pruned.  You're still able to rebuild your state in case of power outage, down to the last keystroke.

\*Vere is not essential to the Urbit stack; one can imagine using Urbit on a hypervisor, or even bare metal.  One member of the community is even working on an independent implementation of Urbit using Graal/Truffle on the JVM.

The Urbit stack (compiler, standard library, kernel, modules, and applications, but excluding Vere) is about 30,000 lines of Hoon.  Urbit is patent-free and MIT licensed.
