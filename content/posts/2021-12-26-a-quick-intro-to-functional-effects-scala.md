+++
date = 2021-12-26T16:15:00+08:00
title = "A quick introduction to functional effects in Scala"
+++

## What is an Effect and why do you need it?

Functional programming (FP) is easy if you just have to write pure functions that do some calculation based
on some inputs and return some output.
However, most useful programs involve side effects, for instance querying a database system,
calling an internal or external API, reading/writing files or even printing to console.
The act of performing any of those side effects is considered an "effectful operation".
In FP, we can use an effect system for dealing with any effectful operations.

The Scala standard library does not have a functional effect system.
Fortunately there are a few really popular and mature open source libraries in the Scala ecosystem.
For example, Cats Effect, Monix (Task) and ZIO provide an effect runtime
and some useful constructs to work with functional effect.

## Exploring Cats Effect

I will use Cats Effect as an example as I am most familiar with it.
The Cats Effect library has a data type, `IO[A]` which you can think of as a blueprint that,
when evaluated, can perform effectful operations and finally return a value of type `A`.

```scala
val getUsersIO = IO(db.query(...))
```

The code above creates an `IO` with an effectful operation
(imagine `db.query(...)` has to query a database system).
Note that by creating an `IO`, any effectful operation is not executed yet.
Remember an `IO[A]` is a _blueprint_ that describes some effectful operations and
will only perform those effectful operations when evaluated.

How do we evaluate a Cats Effect `IO`? We can do that by calling `unsafeRunSync()` on the `IO` value.

```scala
getUsersIO.unsafeRunSync()
```

This will evaluate the IO value, perform the side effects as described and return the result of type `A`,
which could be a `List[User]` in this case.

Having said that, we should only run `unsafeRunSync()` or similar methods at the very edge of our program
which is the entry point of our programs, ie. the main method. As the method name `unsafeRunSync` implies,
the method will perform the side effect right away and therefore should be avoided throughout the rest of
the program.

## But... what is wrong with Future?

You have most likely used `Future` to perform asynchronous computation.
`Future` is available in the Scala standard library and works for the side effects described above so
what is wrong with `Future`? Why do we still need functional effects?

Well, `Future` is evaluated _eagerly_ whereas a functional effect like Cats Effect `IO` is evaluated _lazily_.
What that means is that a `Future` performs side effects immediately upon creation whereas a Cats Effect `IO`
is a blueprint for which the side effects will only be performed when we call something like
`unsafeRunSync()` on it. This is best illustrated by the comic below which is my favorite.

![impure-effects](https://imgur.com/D4Gt7JP.png)
(Credits to [impurepics.com](https://impurepics.com/posts/2019-10-11-dynamite-effects.html))

This means that we can't compose `Future` as values and using `Future` won't make your program
referentially transparent (this is a topic that deserves a blog post on its own).
