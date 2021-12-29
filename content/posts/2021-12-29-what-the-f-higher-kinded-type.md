+++
date = 2021-12-29T17:50:00+08:00
title = "What the F[_]? A quick introduction to Higher Kinded Types in Scala"
+++

## Generics in Scala

We can have generics in Scala through type constructors.

For example, we can have `List[A]` which is a type that accepts a type argument `A` such that `A` can be any types.
Why do we need this? Well, this is so that we can write generic methods or functions as follows.

```scala
def doSomething[A](list: List[A], a: A): List[A] = ???

val newIntList: List[Int] = doSomething(List(1, 2, 3), 3)

val newStringList: List[String] = doSomething(List("alpha", "beta", "delta"), "omicron")
```

Note that `doSomething` works with `List[Int]` or `List[String]` or a `List` of anything for that matter.

## What is a Higher Kinded Type (HKT)?

So far, the example above is just the same old Java-esque generics.
What Scala allows us to do but Java can't is the capability of using Higher Kinded Types.

Generally speaking, a Higher Kinded Type is a type that abstracts over a type parameter which itself takes a type parameter.

## What the F[_]?

In the example below, instead of using a concrete type like `List`, the function `map`
accepts (i) an `F[A]` and (ii) a function that transforms from `A` to `B`; and return an `F[B]`.
This means that the caller can pass in any types that accept exactly one other type,
instead of being limited to only `List`

```scala
def map[F[_], A, B](a: F[A], f: A => B): F[B]

val newIntList: List[Int] = map(List(1,2,3), i => i * 2)

val newIntStream: Stream[Int] = map((1 to 100_000_000).toStream, i => i * 2)
```

For some reason, the convention is using `F[_]` to represent the abstract type parameter
(maybe because people generally use Functor as an example to explain higher kinded types, I guess?)

## Use case

`F[_]` is heavily used in functional programming libraries like [cats](https://typelevel.org/cats/).
It is also often used for implementing Tagless Final (or Finally Tagless) encoding in Scala.

```scala
trait Users[F[_]] {
    def getAll(): F[List[User]]
    def getById(id: String): F[Option[User]]
}
```

The above is a "Tagless Algebra".
You can observe that instead of returning concrete types,
the methods of an algebra return an `F` of something.
At the edge of the program, this `F` will usually be
some effect type, for example, a Cats Effect IO, a Monix Task, a ZIO.
One day I will write a blog post on Tagless Final to dive deeper into the whats, whys and hows.
