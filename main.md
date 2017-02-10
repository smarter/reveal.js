# The State of Dotty
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Guillaume Martres](http://guillaume.martres.me) - EPFL

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## What is DOT?
- <!-- .element: class="fragment" --> Theoretical foundations for Scala
- <!-- .element: class="fragment" --> Goal: prove Scala type system is sound
  - <!-- .element: class="fragment" --> If code compiles and does not use unsafe features, no `ClassCastException`

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## What is Dotty?
- <!-- .element: class="fragment" --> A new, experimental compiler for Scala developed at EPFL
- <!-- .element: class="fragment" --> Type system internals redesigned, inspired by DOT, but externally very similar

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## A chance to redesign components
- <!-- .element: class="fragment" --> Improved incremental compilation
- <!-- .element: class="fragment" -->  Better pattern matching checks
- <!-- .element: class="fragment" --> Some of these components may be ported back to Scala 2

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Tooling
A good developer experience requires good tools:
- <!-- .element: class="fragment" --> A simple REPL (with syntax highlighting!)
- <!-- .element: class="fragment" --> Dottydoc
- <!-- .element: class="fragment" --> IDE support using the [Language Server Protocol](https://github.com/Microsoft/language-server-protocol) (Visual Studio Code, Eclipse, ...)
- <!-- .element: class="fragment" --> See Felix's talk tomorrow for more details!

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## The Dotty Linker
- <!-- .element: class="fragment" --> Call-graph analysis enables many global optimizations:
  - <!-- .element: class="fragment" --> Dead Code Elimination
  - <!-- .element: class="fragment" --> Automatic specialization (no more `@specialize` !)
- <!-- .element: class="fragment" --> See [Dmitry's ScalaDays talk](https://www.youtube.com/watch?v=h8KBLF0AgUc)
--
## The future
- <!-- .element: class="fragment" --> First Technology Preview release of Dotty soon!
- <!-- .element: class="fragment" --> Scala 2 will continue to evolve for a long time
- <!-- .element: class="fragment" --> Convergence:
  - <!-- .element: class="fragment" --> Compatibility mode in Dotty for deprecated features
  - <!-- .element: class="fragment" --> Scala 2 will start implementing features prototyped in Dotty (trait parameters, ...)

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Migrating code
- Some Scala 2 code may not work well with Dotty:
  - <!-- .element: class="fragment" --> Dropped features
  - <!-- .element: class="fragment" --> Differences in type inference
- <!-- .element: class="fragment" --> [Scalafix](https://github.com/scalacenter/scalafix) automatically rewrites your code to ease migration

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
# Dropped features

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Macros
- <!-- .element: class="fragment" --> Scala 2 macros are too tied to the compiler internals
- <!-- .element: class="fragment" --> New macro system: [`scala.meta`](https://github.com/scalacenter/scalafix)
  - <!-- .element: class="fragment" --> Easier to use, more typesafe
  - <!-- .element: class="fragment" --> Should support both Scala 2 and Dotty

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Procedure syntax
- <!-- .element: class="fragment" --> Instead of writing:

```scala
def foo() {
  ...
}
```
<!-- .element: class="fragment" -->
- <!-- .element: class="fragment" --> You should write:

```scala
def foo(): Unit = {
  ...
}
```
<!-- .element: class="fragment" -->

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## <!-- .element: style="text-transform: none;" --> `forSome`
- <!-- .element: class="fragment" --> This syntax is no longer supported:

```scala
val x: Array[T] forSome { type T <: Foo } = ...
```
<!-- .element: class="fragment" -->
- <!-- .element: class="fragment" --> Can usually be rewritten using wildcards:

```scala
val x: Array[_ <: Foo] = ...
```
<!-- .element: class="fragment" -->

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
# Implemented features

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Functions with very many parameters
- <!-- .element: class="fragment" --> Standard library contains classes `Function1`, `Function2`, ..., `Function22`
- <!-- .element: class="fragment" --> New in Dotty: If function has
  too many parameters, use `FunctionXXL` which takes an array of
  parameters.

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Intersection types
- <!-- .element: class="fragment" --> A value has type `A & B` if it has both type `A` and type `B`
  - <!-- .element: class="fragment" --> replaces `A with B`

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Union types
- <!-- .element: class="fragment" --> A value has type `A | B` if it has type `A` or type `B`

```scala
val foo = if (cond) 1 else "test"
```
<!-- .element: class="fragment" -->
- <!-- .element: class="fragment" --> In Scala 2: `foo` has type `Any`
- <!-- .element: class="fragment" --> In Dotty: `foo` has type `Int | String`

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Pattern matching on a union type
```scala
val x = if (cond) 1 else "foo"
x match {
  case x: Int => ...
  case x: String => ...
}
```
- <!-- .element: class="fragment" --> This is safe: if you forget a case in your match the compiler will warn you

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Trait parameters ([SIP-25](http://docs.scala-lang.org/sips/pending/trait-parameters.html))
```scala
trait A {
  val x: String
  println(x)
}
class B extends A { val x = "b" }
new B // prints "null"

```
<!-- .element: class="fragment" -->
- <!-- .element: class="fragment" --> Using early definitions (dropped feature):

```scala
class B extends { val x: String = "b" } with A
new B // prints "b"
```
<!-- .element: class="fragment" -->

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Trait parameters ([SIP-25](http://docs.scala-lang.org/sips/pending/trait-parameters.html))
- Trait parameters are like class parameters (with some restrictions)

```scala
trait A(x: String) {
  println(x)
}
class B extends A("b")
new B // prints "b"
```
<!-- .element: class="fragment" -->

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
# Proposed features
- <!-- .element: class="fragment" --> Enums
- <!-- .element: class="fragment" --> Non-nullable types
- <!-- .element: class="fragment" --> Effect system
- <!-- .element: class="fragment" --> Generic programming, like what Shapeless provides (abstract over case classes, ...)

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Enums
```scala
// Java-like
enum Color {
  case Red
  case Green
  case Blue
}
```
<!-- .element: class="fragment" -->

```scala
// Algebraic Data Type
enum List[+T] {
  case Cons(x: T, xs: List[T])
  case Nil extends List[Nothing]
}
```
<!-- .element: class="fragment" -->

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Non-nullable types
- `null` has type `Null`
- <!-- .element: class="fragment" --> Currently: `Null` is a subtype of every non-value class
- <!-- .element: class="fragment" --> Proposal: `Null` is a subtype of no other type

```scala
val x: String = null // This would be illegal
```
<!-- .element: class="fragment" -->

```scala
val x: String | Null = null // This is fine
```
<!-- .element: class="fragment" -->

```scala
val x: String? = null // Shorter syntax
```
<!-- .element: class="fragment" -->

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
--
## Contributing
- <!-- .element: class="fragment" --> Documentation: [dotty.epfl.ch](http://dotty.epfl.ch/)
- <!-- .element: class="fragment" --> Join our Gitter chat: [gitter.im/lampepfl/dotty](https://gitter.im/lampepfl/dotty)
- <!-- .element: class="fragment" --> Try to compile your own project, report issues
- <!-- .element: class="fragment" --> Test our IDE support
- <!-- .element: class="fragment" --> Look at issues marked "help wanted" on [Github](https://github.com/lampepfl/dotty/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)
  - <!-- .element: class="fragment" --> Many things to do even if you're not a compiler expert!

JapaneseTranslationGoesHere
<!-- .element: class="footer" -->
