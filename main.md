# Adding polymorphic functions to Scala
[Guillaume Martres](http://guillaume.martres.me) - EPFL

<!-- .element: style="text-align: center !important" -->
--
## Methods
``` scala
def m(x: Int): List[Int] = List(x)
```
- <!-- .element: class="fragment" --> Methods are not first-class values, they
  can only be called.
- <!-- .element: class="fragment" --> Trying to use a method as a value converts it into a function by eta-expansion

``` scala
val f: Int => List[Int] = m
```
<!-- .element: class="fragment" --> 

Adapted to: <!-- .element: class="fragment" -->

``` scala
val f: Int => List[Int] = x => m(x)
```
<!-- .element: class="fragment" --> 

--
## Functions
``` scala
val f: Int => List[Int] = x => m(x)
f(1)
```

Interpreted as: <!-- .element: class="fragment" --> 

``` scala
val f: Function1[Int, List[Int]] = new Function1[Int, List[Int]] {
  def apply(x: Int): List[Int] = m(x)
}
f.apply(1)
```
<!-- .element: class="fragment" -->

--
## <span style="text-transform: none;">Function*</span>

- <!-- .element: class="fragment" --> From Function0 to Function22

``` scala
trait Function1[+T, -R] {
  def apply(x: T): R
}
```
<!-- .element: class="fragment" --> 

- <!-- .element: class="fragment" --> New in Dotty, FunctionN for all N
  - <!-- .element: class="fragment" --> Erases to FunctionXXL

``` scala
trait FunctionXXL {
  def apply(xs: Array[Object]): Object
}
```
<!-- .element: class="fragment" -->

--

## Back to methods
- <!-- .element: class="fragment" --> Multiple parameter lists:

``` scala
def m(x: Int)(y: Int): List[Int]
```
<!-- .element: class="fragment" --> 

``` scala
Int => Int => List[Int]
```
<!-- .element: class="fragment" --> 

- <!-- .element: class="fragment" --> Repeated parameters:

``` scala
def m(xs: Int*): List[Int]
```
<!-- .element: class="fragment" --> 

``` scala
Seq[Int] => List[Int]
```
<!-- .element: class="fragment" --> 

- <!-- .element: class="fragment" --> Default parameters:

``` scala
def m(x: Int = 1): List[Int]
```
<!-- .element: class="fragment" --> 

``` scala
Int => List[Int] // ¯\_(ツ)_/¯
```
<!-- .element: class="fragment" --> 

--
## But wait, there's more

- <!-- .element: class="fragment" --> Implicit parameters:

``` scala
def m(implicit x: Int): List[Int]
```
<!-- .element: class="fragment" --> 

``` scala
implicit Int => List[Int]
```
<!-- .element: class="fragment" --> 

- <!-- .element: class="fragment" --> See the Dotty documentation
- <!-- .element: class="fragment" --> See the Simplicitly paper by Odersky et al.<!-- .element: class="fragment" -->

--
## But wait, there's even more
- <!-- .element: class="fragment" --> Dependent method:

``` scala
def m(x: Int): List[x.type]
```
<!-- .element: class="fragment" --> 

``` scala
(x: Int) => List[x.type]
```
<!-- .element: class="fragment" --> 

``` scala
Function1[Int, List[Int]] {
  def apply(x: Int): List[x.type]
}
```
<!-- .element: class="fragment" --> 

Erases to Function1 <!-- .element: class="fragment" -->

--
## Polymorphism

``` scala
def m[T](x: T): List[T]
```
<!-- .element: class="fragment" --> 

``` scala
[T] => (x: T) => List[T]
```
<!-- .element: class="fragment" --> 

``` scala
trait PolyFunction
```
<!-- .element: class="fragment" --> 

``` scala
PolyFunction {
  def apply[T](x: T): List[T]
}
```
<!-- .element: class="fragment" --> 

``` scala
val f = [T] => (x: T) => List(x)
```
<!-- .element: class="fragment" --> 

``` scala
val f = new PolyFunction {
  def apply[T](x: T): List[T] = List(x)
}
```
<!-- .element: class="fragment" --> 

--
## Magic Erasure

Given: <!-- .element: class="fragment" --> 

``` scala
PolyFunction {
  def apply[T_1, ..., T_M](x_1: T, ..., x_N: S): List[T]
}

```
<!-- .element: class="fragment" --> 

Erase it to: <!-- .element: class="fragment" --> 

``` scala
FunctionN
```
<!-- .element: class="fragment" --> 

--
## Polymorphic values ?
``` scala
val f: [T] List[T] = Nil
```
 <!-- .element: class="fragment" --> 

- Unsound: <!-- .element: class="fragment" --> 

``` scala
class Ref[T](var x: T)
val f: [T] Ref[List[T]] = new Ref(Nil)

val r1: Ref[List[Int]] = f[Int]
val r2: Ref[List[String]] = f[String]
r1.x = List(1)

val elem: String = r2.x.head // ClassCastException 😱
```
 <!-- .element: class="fragment" --> 

- <!-- .element: class="fragment" --> Solution in SML: "value restriction" 
--
## Just use covariance
``` scala
val f: List[Nothing] = Nil

val f1: List[Int] = f
```
--
# Usecases
--
## Writing Haskell fanfiction

``` haskell
forall a b. Functor f => (a -> b) -> f a -> f b
```
 <!-- .element: class="fragment" --> 

``` scala
[F[_], A, B] => (A => B) => F[A] => implicit Functor[F] => F[B]
```
 <!-- .element: class="fragment" --> 
 
``` scala
[F[_]: Functor, A, B] => (A => B) => F[A] => F[B]
```
 <!-- .element: class="fragment" --> 

--
## Scala fashion
- <!-- .element: class="fragment" --> 2018 trends:
  - <!-- .element: class="fragment" --> Free monads are out
  - <!-- .element: class="fragment" --> Tagless final is in

``` scala
type Program[Alg[_[_]], A] = [F[_]: Applicative] => Alg[F] => F[A]
```
<!-- .element: class="fragment" --> 

- <!-- .element: class="fragment" --> See "Optimizing Tagless Final" blog post
  series by Luka Jacobowitz on https://typelevel.org/blog

--
## Resource leaks

```haskell
runST :: forall a. (forall s. ST s a) -> a
```

--
## Todo List

- <!-- .element: class="fragment" --> Implement type inference
- <!-- .element: class="fragment" --> Figure out the syntax

--
## Questions ?
- PR: https://github.com/lampepfl/dotty/pull/4672
- More info: [dotty.epfl.ch](dotty.epfl.ch)
- Come chat with us: [gitter.im/lampepfl/dotty](http://gitter.im/lampepfl/dotty)
- Contributors welcome!
