# The State of Dotty
[Guillaume Martres](http://guillaume.martres.me) - EPFL

<!-- .element: style="text-align: center !important" -->

次世代Scalaコンパイラー Dottyの今

<!-- .element: class="footer" -->
--
## What is DOT?
- <!-- .element: class="fragment" --> Theoretical foundations for Scala
- <!-- .element: class="fragment" --> Goal: prove Scala type system is sound
  - <!-- .element: class="fragment" --> If code compiles and does not use unsafe features, no `ClassCastException`

DOT計算は、Scala型システムの理論的基礎

<!-- .element: class="footer" -->
--
## What is Dotty?
- <!-- .element: class="fragment" --> A new, experimental compiler for Scala developed at EPFL
- <!-- .element: class="fragment" --> Type system internals redesigned, inspired by DOT, but externally very similar

Dotty は EPFL が開発中の実験的な新コンパイラ

<!-- .element: class="footer" -->
--
## A chance to redesign components
- <!-- .element: class="fragment" --> Improved incremental compilation
- <!-- .element: class="fragment" -->  Better pattern matching checks
- <!-- .element: class="fragment" --> Some of these components may be ported back to Scala 2

インクリメンタルコンパイルやパターンマッチを再設計するチャンス<br>
これらの改善は Scala 2 にも取り込めるかも

<!-- .element: class="footer" style="font-size: 83% !important;" -->
--
## Tooling
A good developer experience requires good tools:
- <!-- .element: class="fragment" --> A simple REPL (with syntax highlighting!)
- <!-- .element: class="fragment" --> Dottydoc
- <!-- .element: class="fragment" --> IDE support using the [Language Server Protocol](https://github.com/Microsoft/language-server-protocol) (Visual Studio Code, Eclipse, ...)
- <!-- .element: class="fragment" --> See Felix's talk tomorrow for more details!

良い開発体験のためにはコンパイラ以外のツール群の改善も必要

<!-- .element: class="footer" -->
--
## The Dotty Linker
- <!-- .element: class="fragment" --> Call-graph analysis enables many global optimizations:
  - <!-- .element: class="fragment" --> Dead Code Elimination
  - <!-- .element: class="fragment" --> Automatic specialization (no more `@specialize` !)
- <!-- .element: class="fragment" --> See [Dmitry's ScalaDays talk](https://www.youtube.com/watch?v=h8KBLF0AgUc)

Dotty リンカが呼び出しグラフを解析して最適化<br>
到達不能コードの削除、@specialize の自動化

<!-- .element: class="footer" -->
--
## The future
- <!-- .element: class="fragment" --> First Technology Preview release of Dotty soon!
- <!-- .element: class="fragment" --> Scala 2 will continue to evolve for a long time
- <!-- .element: class="fragment" --> Convergence:
  - <!-- .element: class="fragment" --> Compatibility mode in Dotty for deprecated features
  - <!-- .element: class="fragment" --> Scala 2 will start implementing features prototyped in Dotty (trait parameters, ...)

近日中に最初の技術プレビュー版 Dotty がリリース！<br>
Scala 2 も Dotty で検証した機能を取り込みながら開発を継続

<!-- .element: class="footer" -->
--
## Migrating code
- Some Scala 2 code may not work well with Dotty:
  - <!-- .element: class="fragment" --> Dropped features
  - <!-- .element: class="fragment" --> Differences in type inference
- <!-- .element: class="fragment" --> [Scalafix](https://github.com/scalacenter/scalafix) automatically rewrites your code to ease migration

Scalafix は Scala 2 から Dotty への移行を自動化するツール

<!-- .element: class="footer" -->
--
# Dropped features

廃止された機能

<!-- .element: class="footer" -->
--
## Macros
- <!-- .element: class="fragment" --> Scala 2 macros are too tied to the compiler internals
- <!-- .element: class="fragment" --> New macro system: [`scala.meta`](https://github.com/scalacenter/scalafix)
  - <!-- .element: class="fragment" --> Easier to use, more typesafe
  - <!-- .element: class="fragment" --> Should support both Scala 2 and Dotty

従来のマクロに代わり、scala.meta が新しいマクロシステムとなる<br>
Scala 2 と Dotty の両方をサポートする予定

<!-- .element: class="footer" style="font-size: 85% !important;" -->
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

プロシージャ構文の変更（必ず `Unit` をつける）

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

`forSome` は代わりにワイルドカードで書き換える

<!-- .element: class="footer" -->
--
# Implemented features

実装された新機能

<!-- .element: class="footer" -->
--
## Improved type inference
```scala
def ap1[A,B](a: A)(f: A => B): B = f(a)
def ap2[A,B](a: A, f: A => B): B = f(a)

```
<!-- .element: class="fragment" -->
```scala
ap1(1)(x => x * 2) // Compiles with Scala2 and Dotty
ap2(1, x => x * 2) // Only compiles with Dotty

```
<!-- .element: class="fragment" -->

-  <!-- .element: class="fragment" --> See the talk [Dotty and Types: The Story So Far](http://guillaume.martres.me/talks/typelevel-summit-oslo/#/) for more details

型推論の改善

<!-- .element: class="footer" -->
--
## Functions with very many parameters
- <!-- .element: class="fragment" --> Standard library contains classes `Function1`, `Function2`, ..., `Function22`
- <!-- .element: class="fragment" --> New in Dotty: If function has
  too many parameters, use `FunctionXXL` which takes an array of
  parameters.

Dotty では関数の引数の 22 個制限がなくなる

<!-- .element: class="footer" -->
--
## Intersection types
- <!-- .element: class="fragment" --> A value has type `A & B` if it has both type `A` and type `B`
  - <!-- .element: class="fragment" --> replaces `A with B`

`A with B` を置き換える交差型 (`A & B`) の導入

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

合併型 (`A | B`) は `A` と `B` のどちらでもある型<br>
Scala 2 では実現できなかった型の表現

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

コンパイラが合併型の網羅性を警告してくれるので安全

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

トレイトパラメータの導入により事前定義の機能は廃止

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

クラス・パラメータに相当する

<!-- .element: class="footer" -->
--
# Proposed features
- <!-- .element: class="fragment" --> Enums
- <!-- .element: class="fragment" --> Non-nullable types
- <!-- .element: class="fragment" --> Effect system
- <!-- .element: class="fragment" --> Generic programming, like what Shapeless provides (abstract over case classes, ...)

現在提唱されている機能

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

Java ライクな列挙型。代数的データ型も定義できる

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

`null` を許容しない型をデフォルトとする

<!-- .element: class="footer" -->
--
## Contributing
- <!-- .element: class="fragment" --> Documentation: [dotty.epfl.ch](http://dotty.epfl.ch/)
- <!-- .element: class="fragment" --> Join our Gitter chat: [gitter.im/lampepfl/dotty](https://gitter.im/lampepfl/dotty)
- <!-- .element: class="fragment" --> Try to compile your own project, report issues
- <!-- .element: class="fragment" --> Test our IDE support
- <!-- .element: class="fragment" --> Look at issues marked "help wanted" on [Github](https://github.com/lampepfl/dotty/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)
  - <!-- .element: class="fragment" --> Many things to do even if you're not a compiler expert!

皆さんの協力が必要です。コンパイラの専門家じゃなくても大丈夫！<br>
自分のプロジェクトをコンパイルして問題を報告、IDE のテストなど

<!-- .element: class="footer" style="font-size: 84% !important;" -->
