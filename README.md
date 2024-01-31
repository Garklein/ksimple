# k/simple

a tiny k interpreter for educational purposes by arthur whitney

## target audience

the contents of this repository assume readership who have read and 
understood [k&r](https://en.wikipedia.org/wiki/The_C_Programming_Language)
at some point in their careers. any experience with using interpreters of
vector languages is beneficial but not necessary. any experience with
implementing language interpreters would definitely be of help.

## background

in early january 2024, a group of motivated individuals asked [arthur whitney](https://en.wikipedia.org/wiki/Arthur_Whitney_(computer_scientist))
to demonstrate the fundamentals of the design, organization and style of his
method of writing interpreters in c in the most succinct and approachable 
way possible.

about a day later, arthur offered a tiny interpreter of a toy vector 
language, implemented in about 25 lines of c with a small header file.
his code is published in this repository under [/ref](/ref).

a few days later, the regents of kparc were invited to share their 
impressions on this codebase, and optionally provide some additional 
commentary.

our commentary is offered in form of two files `a.[ch]` in the root of 
this repository, which serve a number of purposes:

1. both files can be seen as two chapters of a small "essay", header
   file [a.h](/a.h) being the first chapter, [`a.c`](/a.c) source file being the second.
   not only the text seeks to explain the code in reasonable detail,
   which is strictly line by line, but also takes a few liberties and
   diversions in a bid to elucidate the "bigger picture" behind this tiny piece of code.

3. for the ease of comprehension, the narrative has been made
   **completely linear**, and hopefully makes for an easy, entertaining and
   useful read. to make this possible, the original code had seen some
   very minor restructuring and regrouping; however, no refactoring,
   no changes to logic or naming conventions have taken place, except 
   for a handful cosmetic enhancements which have been discussed with
   atw and are seen as beneficial to the cause. a number of less trivial one-line
   functions (e.g. `e()`) are presented in "exploded view" to simplify discussion
   of their control flow and role of individual components.

   although the c code of `k/simple` is formatted to fit on mobile phone screens
   in portrait mode, for a more comfortable reading experience we recommend
   using professional equipment.

5. with help of the included [makefile](/makefile), kparc's "essay" can
   be built to any available architecture, e.g. riscv, arm, wasm32, xtensa
   or even x86. default make target assumes presence of a recent gcc, clang
   or tinyc compiler.

## quick start

```
$ cd && git clone https://github.com/kparc/ksimple
$ cd ksimple && make
$ ./a
k/simple (c) 2024 atw/kpc
 2+2
4
 x:!9
 y:2+x
 x-y
-2 -2 -2 -2 -2 -2 -2 -2 -2
 z:x,y
 #z
18
 x+!3
Add:59 length
 \w
36
 x:y:z:0
 \w
0
 ^C
 $ vi k.c
```

## language specification / disclaimer

*caveat emptor*, manage your expectations. this interpreter is useless for any practical purpose except the stated one, which is education. the language is as follows:

* although k/simple is a computer language, its grammar is defined in terms we normally use to denote parts of human speech. that is, k expression is composed of **nouns**, **verbs** and **adverbs**.

* the definition of **verb** is something in between definitions of **operation** in mathematics and **operator** in computer science. to make things simple, we define *verb* as an instruction to the interpreter to 
  do something with exactly one or exactly two operands, or **nouns**. a verb that takes one noun to operate on is called monadic, and verb that takes two nouns is called dyadic. consider the following two 
  sentences:

  * *shuffle this deck of cards*
  * *take three random cards from this deck*

  in linguistics, the structure of the first sentence is known as verb-only predicate, while the second is a verb-plus-direct-object predicate.
  in k, we recognize the verb "shuffle" in the first sentence to be monadic, while the second is a dyadic verb "take". the semantics of an action
  of a verb is defined by the number of operands given to it, also called "rank" or "valence". this means that monadic `+x` and dyadic `x+y` have
  two entirely different meanings.

* a **noun** in k/simple can be either an **atom** (aka scalar value) or a **vector** (aka ordered list of atoms).

* the one and only type of **atom** formally supported by k/simple is a signed 8-bit integer `-128..127`.

* a vector of atoms is limited in length to unsigned 8-bit integer, that is no more than 255 items.

* k/simple supports 6 verbs, `+`, `-`, `!`, `#`, `,` and `@`, all of which have different meanings depending on their *rank*. these meanings are very well documented in [a.c](/a.c) where they are also declared and defined.

* an **adverb** is a higher-order function: it takes a verb and modifies its action in some desirable way. k/simple implements one adverb `over`, also known as `fold` and `reduce` in 
  functional speak. that is, `over` folds a given vector of values and reduces it into a scalar using a given verb. for example, `+/` reads as "plus over" and computes a sum elements of a given vector:

  ```
   +/1,2,3,4,5,6
  21
  ```

* k expressions are evaluated in a way you're probably not familiar with, which is *right to left*, or, in math speak, *left of right*. this only sounds nonsensical until you get lit:

```
 3+2+1     /take 1, add 2, add 3
6

 3*2+1     /take 1, add 2, multiply by 3
9
```

* This also means that there is only one operator precededence rule to be learned in k/simple: there are no precedence rules to be learned in k/simple. all operators have the same binding strength, including arithmetic. 
  for example, multiplication doesn't bind stronger than addition, and doesn't get computed first. the lack of precedence is not what we've all learned very early in our careers, but it actually works pretty well.

* k/simple is simple enough to get away without tokenizer and parser. instead, it accepts user input as a string of up to 99 tokens, and evaluates it token by token strictly left of right (see above). a token in k/simple 
  is a single character, which can be either a verb `+-!#,@`, a noun `0..9` or a name of a global variable `abc..xyz` which are also treated as nouns.

* assignment of a value to a global variable is not a verb, and it is not `=`. instead, it is a special case and is `:`. for example:

```
x:4
x:x+1
x
 5
```

* global namespace `a..z` is very useful, and has an unusual feature: you can assign a value to a global in a traditional way `x:2`,
  or do it *inline*, as in `x:1+y:2`, which is the same as `y:2` followed by `x:1+y`. one expression, two assignments.

* k/simple implements simple memory management by refcounting. two handy system commands are implemented, which are useful for refcount debugging:

  * `\w` print current workspace size
  * `\v` print global namespace: varname, refcount, length

* `\\` exits the process.

* k/simple can run in two modes: interactive or batch. batch mode expects a filename of a k script, e.g.:

```
$ ./a t.k
-1
0
0
-1
1 2 3 4 5 6 7 8 9
2
0 2 4 6 8 10 12 14 16
0 -1 -2 -3 -4 -5 -6 -7 -8
...
```

## suggested exercise

the authors hope that this material enables and inspires further experimentation on reader's own, which can be a very rewarding and fun pastime. for example, one might consider the following toy problems of various degrees of difficulty:

**i'm too young to die:**

* implement a few more simple verbs, e.g. dyadic `f*x`.
* modify repl to support the exit command (traditionally, a double backslash `\\`)
* add workspace size monitor (i.e. memory allocation) e.g. to prompt or via extra command (`\w`)

**hurt me plenty:**

* inspect verb implementations provided by atw for edge cases, and add checks (which are intentionally omitted)
* extend maximum vector length to `MAX_UINT`
* find and fix at least three segfaults

**ultraviolence:**

* change base type from 8bit integer to `long`
* fix tokenizer to support integer numerals greater than 9 and less than 0
* fix tokenizer to support efficient direct vector input (e.g. `42 57 1010` instead of `1,2,3,4`)

**nightmare:**

* implement a simple parser (e.g. to support quoted strings and parens)
* implement nested vectors, and verb `flip`
* make vector arithmetic penetrating
* implement adverbs `scan` and `over`

**progress takes sacrifice:**

* implement floating point type
* implement functions and local scope 
* implement a test suite for your `k`


`//:~`














