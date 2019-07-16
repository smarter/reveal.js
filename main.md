# A universal encoding for functions
[Guillaume Martres](http://guillaume.martres.me) - EPFL

<!-- .element: style="text-align: center !important" -->
--
## In the beginning, there were methods
``` scala
def m(x: Int): List[Int] = List(x)
```
- <!-- .element: class="fragment" --> Methods are not first-class values, they
  can only be called.
- <!-- .element: class="fragment" --> Trying to use a method as a value converts it into a function by eta-expansion

``` scala
val f = m
```
<!-- .element: class="fragment" -->

Adapted to: <!-- .element: class="fragment" -->

``` scala
val f = x => m(x)
```
<!-- .element: class="fragment" -->

--
## Let there be functions!
``` scala
val f = x => m(x)
f(1)
```

Equivalent to: <!-- .element: class="fragment" -->

``` scala
val f = new Function1[Int, List[Int]] {
  def apply(x: Int): List[Int] = m(x)
}
f.apply(1)
```
<!-- .element: class="fragment" -->

--
## <span style="text-transform: none;">The Function* family</span>

- <!-- .element: class="fragment" --> From Function0 to Function22

``` scala
trait Function1[+T, -R] {
  def apply(x: T): R
}
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> New in Dotty, FunctionN for all N
  - <!-- .element: class="fragment" --> Compiler fiction: erases to FunctionXXL

``` scala
trait FunctionXXL {
  def apply(xs: Array[Object]): Object
}
```
<!-- .element: class="fragment" -->

--

## But wait, there's more
- <!-- .element: class="fragment" --> Implicit parameters:

``` scala
def m(implicit x: Int): List[Int]
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> Dependent:

``` scala
def m(x: Int): List[x.type]
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> Polymorphic:

``` scala
def m[T](x: T): List[T]
```
<!-- .element: class="fragment" -->

--
## Abusing refinement types

How do I represent this using Scala types ?

``` scala
[T] => (x: T) => List[T]
```


I want something that has an apply method with the correct type <!-- .element: class="fragment" --> 

Given a marker trait: <!-- .element: class="fragment" --> 

``` scala
trait PolyFunction {}
```
<!-- .element: class="fragment" -->

We can make up a refinement type: <!-- .element: class="fragment" --> 

``` scala
PolyFunction {
  def apply[T](x: T): List[T]
}
```
<!-- .element: class="fragment" -->

--
# WAT

<img src="images/wat.jpg" width="80%" style="border: none; margin-left: 8%;">

--
# No way

Yes way! <!-- .element: class="fragment" --> 

Take that thing: <!-- .element: class="fragment" --> 

``` scala
val f = [T] => (x: T) => List(x)
```
<!-- .element: class="fragment" -->

And desugar it to: <!-- .element: class="fragment" --> 

``` scala
val f = new PolyFunction {
  def apply[T](x: T): List[T] = List(x)
}
```
<!-- .element: class="fragment" -->

--
## Magic Erasure on types

Given the type:

``` scala
PolyFunction {
  def apply[T](x: T): List[T]
}

```


Erase it to: <!-- .element: class="fragment" -->

``` scala
Function1
```
<!-- .element: class="fragment" -->

--
## Magic Erasure on values

Given the value:

``` scala
new PolyFunction {
  def apply[T](x: T): List[T] = List(x)
}

```

Erase it to: <!-- .element: class="fragment" -->

``` scala
new Function1 {
  def apply(x: Object): Object = List(x)
}
```
<!-- .element: class="fragment" -->

--
## Let's talk about subtyping

```scala
val headSeq: Seq[Int] => Int = x => x.head
```
<!-- .element: class="fragment" -->

```scala
val headList: List[Int] => Int = headSeq
```
<!-- .element: class="fragment" -->

```scala
List[Int] <:< Seq[Int]
```
<!-- .element: class="fragment" -->

```scala
Seq[Int] => Int <:< List[Int] => Int
```
<!-- .element: class="fragment" -->

--
## Going polymorphic

```scala
val headSeq: [T] => Seq[T] => T = [T] => x => x.head
```
<!-- .element: class="fragment" -->

```scala
val headList: [T] => List[T] => T = headSeq
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> parameters of methods in refinements
  are invariant:

```scala
PolyFunction { def apply[T](x: Seq[T]): T }

                  !<:<

PolyFunction { def apply[T](x: List[T]): T }
```
<!-- .element: class="fragment" -->

--
## But why though ?

- <!-- .element: class="fragment" --> Structural type members correspond to Java methods
- <!-- .element: class="fragment" --> The JVM does not have a notion of variance built-in:

```scala
def apply[T](x: Seq[T]): T
def apply[T](x: List[T]): T
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> These are just overloads

--
## Making up our own rules

- <!-- .element: class="fragment" --> Special-case subtyping of refined
  PolyFunction:

```scala
PolyFunction { def apply[T](x: Seq[T]): T }

                  <:<

PolyFunction { def apply[T](x: List[T]): T }
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> This is OK because in the end we're just
  calling the `apply` method defined in `Function1`

--
## This isn't very satisfying

- <!-- .element: class="fragment" --> Lots of function types: polymorphic,
  dependent, implicit, arity >= 23, ...
- <!-- .element: class="fragment" --> All have their own encoding with their own
  limitations
- <!-- .element: class="fragment" --> Combinatorics: how do you encode a polymorphic implicit
  dependent function ?
--
## One encoding to rule them all

- <!-- .element: class="fragment" --> The semantics of `PolyFunction` aren't
  really specific to polymorphic functions.
- <!-- .element: class="fragment" --> Let's rename it to `Function`.

```scala
Int => Int
```
<!-- .element: class="fragment" -->

```scala
// Old and tired
Function1[Int, Int]
```
<!-- .element: class="fragment" -->

```scala
// New hotness!
Function {
  def apply(x: Int): Int
}
```
<!-- .element: class="fragment" -->

--
## Economy of concepts

```scala
(x: Int) => List[x.type]
```

<!-- .element: class="fragment" -->
```scala
Function {
  def apply(x: Int): List[x.type]
}
```
<!-- .element: class="fragment" -->

```scala
(implicit x: Int) => Int
```

<!-- .element: class="fragment" -->
```scala
Function {
  def apply(implicit x: Int): Int
}
```
<!-- .element: class="fragment" -->

--
## Source and binary compatibility

```scala
type Function0[+R] = Function { def apply(): R }
type Function1[-T1, +R] = Function { def apply(a: T1): R }
// ...
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> Reinterpret types referenced in sources or classfiles
- <!-- .element: class="fragment" --> Erase them to their original representation!

--
## Complication: subclassing

```scala
abstract class MyFun extends Function1[Int, Int] {
  def zero: Int = apply(0)
}
```
<!-- .element: class="fragment" -->

```scala
abstract class MyFun extends (Function { def apply(x: Int): Int }) {
  def zero: Int = apply(0)
}
```
<!-- .element: class="fragment" -->

--
## Complication: inference

```scala
def foo[B](f: Int => B): Unit
val dep: (x: Int) => List[x.type]
foo(dep) // B = ?
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> Constraint:

```scala
B >: List[x.type]
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> Can't infer `B = List[x.type]`, because `x` is not in scope!
- <!-- .element: class="fragment" --> Need to _avoid_ types which are not in
  scope

```scala
B = List[Int]
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> Type avoidance is a common operation in
  the typechecker.

--
## Is this actually a good idea ?

- <!-- .element: class="fragment" --> Not sure yet
- <!-- .element: class="fragment" --> Functions are pretty fundamental to Scala,
  changing them in any way is pretty risky
- <!-- .element: class="fragment" --> Things that could go wrong:
  - <!-- .element: class="fragment" --> Compile-time performance hit
  - <!-- .element: class="fragment" --> Runtime performance hit
  - <!-- .element: class="fragment" --> Existing semantics too hard to preserve exactly
  - <!-- .element: class="fragment" --> Everything I haven't thought about yet

--
## Questions ?
- More info: [dotty.epfl.ch](dotty.epfl.ch)
- Come chat with us: [gitter.im/lampepfl/dotty](http://gitter.im/lampepfl/dotty)
- Contributors welcome!
