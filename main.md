# Interactive Development using the Dotty Compiler
[Guillaume Martres](http://guillaume.martres.me) - EPFL

<!-- .element: style="text-align: center !important" -->
--
## What is Dotty?
- <!-- .element: class="fragment" --> A new, experimental compiler for Scala developed at EPFL
- <!-- .element: class="fragment" --> Type system internals redesigned, inspired by DOT, but externally very similar
- <!-- .element: class="fragment" --> More info: [dotty.epfl.ch](dotty.epfl.ch)
--
## A chance to redesign components
- <!-- .element: class="fragment" --> Improved incremental compilation (avoid undercompilation)
- <!-- .element: class="fragment" --> Better pattern matching checks
  ([Fengyun 16](https://infoscience.epfl.ch/record/225497/files/p61-liu.pdf),
  [reused for Swift!](https://github.com/apple/swift/pull/8908))

--
## Tooling
A good developer experience requires good tools:
- <!-- .element: class="fragment" --> A REPL (with syntax highlighting!)
- <!-- .element: class="fragment" --> Dottydoc (used to generate [dotty.epfl.ch/docs](http://dotty.epfl.ch/docs))
- <!-- .element: class="fragment" --> **IDE support**

-- <!-- .element: data-transition="slide-in"  -->
## State of the art
- <!-- .element: class="fragment" data-fragment-index="1" --> Based on the [Scala Presentation Compiler](https://github.com/scala/scala/tree/2.12.x/src/interactive/scala/tools/nsc/interactive)
  - <!-- .element: class="fragment" --> Scala-IDE
  - <!-- .element: class="fragment" --> ENSIME
- <!-- .element: class="fragment" --> Reimplementation of the Scala typechecker
  - <!-- .element: class="fragment" --> Scala plugin for IntelliJ IDEA
-- <!-- .element: data-transition="slide-out"  -->
## State of the art
- Based on the [Scala Presentation Compiler](https://github.com/scala/scala/tree/2.12.x/src/interactive/scala/tools/nsc/interactive) (3 KLOC)
  - Scala-IDE (66 KLOC)
  - ENSIME (server: 15 KLOC, emacs client: 10 KLOC)
- Reimplementation of the Scala typechecker
  - Scala plugin for IntelliJ IDEA (230 KLOC)
-- <!-- .element: data-transition="slide-in"  -->
## Design principles
1. <!-- .element: class="fragment" --> Code reuse
2. <!-- .element: class="fragment" --> Editor-agnosticity
3. <!-- .element: class="fragment" --> Easy to use (and to install!)
-- <!-- .element: data-transition="slide-out"  -->
## Design principles
1. <span style="color: orangered;"> Code reuse </span>
2. <span style="opacity: 0.5;"> Editor-agnosticity </span>
3. <span style="opacity: 0.5;"> Easy to use (and to install!) </span>
<!-- -- -->
<!-- # Part 1 -->
<!-- ## Interactive API<span style="text-transform: none;">s</span> based on reusable primitives -->
--<!-- .element: data-transition="slide-in"  -->
## Querying the compiler
<img class="fragment" style="margin-left: 130px; border:none;" src="images/phases-black.png">
- <!-- .element: class="fragment"  --> Each phase progressively simplify trees until they can be emitted as JVM bytecode
--<!-- .element: data-transition="slide-out"  -->
## Querying the compiler
<img style="margin-left: 130px; border:none;" src="images/phases-black-typer.png">
- Each phase progressively simplify trees until they can be emitted as JVM bytecode
-- <!-- .element: data-transition="slide-in"  -->
## Source code
```scala
  final val elem = 1

  val foo = elem🚩 + 1
```

- <!-- .element: class="fragment" -->  Cursor position = 🚩
- <!-- .element: class="fragment" -->  Query: jump to definition
-- <!-- .element: data-transition="none"  -->
## Tree after <span style="text-transform: none; font-family: monospace;">Typer</span>
```scala
  final val elem: 1 = 1

  val foo: Int = elem + 1
```
- <!-- .element: class="fragment" --> Every tree node has a type and a position
- <!-- .element: class="fragment" --> Query can be answered
-- <!-- .element: data-transition="slide-out"  -->
## Tree after <span style="text-transform: none; font-family: monospace;">FirstTransform</span>
```scala
  final val elem: 1 = 1

  val foo: Int = 2
```
- <!-- .element: class="fragment" -->  Information lost by constant folding
- <!-- .element: class="fragment" --> Impossible to answer query
--
## Querying the compiler
- <!-- .element: class="fragment" --> Store trees after `Typer`
- <!-- .element: class="fragment" --> Respond to IDE queries by traversing trees
- <!-- .element: class="fragment" --> What about code that has already been compiled?
--
## Pickling
<img style="margin-left: 130px; border:none; margin-bottom: -20px;" src="images/phases-black-pickler.png">
- <!-- .element: class="fragment" --> In Scala 2: store methods signatures (for separate compilation)
- <!-- .element: class="fragment" --> In Dotty: store full trees
--
## TASTY: Typed AST serialization format
- <!-- .element: class="fragment" --> Original motivation: solve the [binary compatibility problem](https://www.slideshare.net/Odersky/scalax)
  - <!-- .element: class="fragment" --> Always use JVM bytecode: breaks when compiler encoding changes
  - <!-- .element: class="fragment" --> Always recompile source code: breaks when the typechecker changes
- <!-- .element: class="fragment" --> Can also be used to provide interactive features: deserialize and query trees
--
## Interactive APIs
-  <!-- .element: class="fragment" --> Convenience methods for tree traversal, compiler lifecycle management
-  <!-- .element: class="fragment" --> Used both in the IDE and the REPL
-  <!-- .element: class="fragment" --> In the future: indexing, resource management, interruption handling, partial typechecking, ...
-  <!-- .element: class="fragment" --> 0.5 KLOC
--
## Design principles
1. <span style="opacity: 0.5;"> Code reuse </span>
2. <span style="color: orangered;"> Editor-agnosticity </span>
3. <span style="opacity: 0.5;"> Easy to use (and to install!) </span>
<!-- -- -->
<!-- # Part 2 -->
<!-- ## Editor-agnosticity -->
--
## The IDE Portability Problem
Getting <em style="font-family: serif;">m</em> IDEs to support <em style="font-family: serif;">n</em> programming languages requires <em style="font-family: serif;">n*m</em> IDE plugins.
<!-- Developing IDE plugin require expertise both in compiler internals and IDE internals. -->
--
## The Language Server Protocol
<img src="images/interaction-diagram.png">

<!-- .element: style="text-align: center !important" -->
--
## Basics of the LSP
- <!-- .element: class="fragment" --> First implemented in Visual Studio Code
- <!-- .element: class="fragment" --> JSON-RPC
- <!-- .element: class="fragment" --> IDE notifies the language server about user actions
- <!-- .element: class="fragment" --> LS maintains internal representation of code
- <!-- .element: class="fragment" --> LS notify IDE about warnings/errors
- <!-- .element: class="fragment" --> IDE can send requests usually triggered by user actions
- <!-- .element: class="fragment" --> Asynchronous, cancelable
--
<!-- ## Limitations -->
<!-- - Few IDEs support it well currently -->
<!-- - Only one kind of refactoring: renaming -->
<!-- -- -->
## Implementing the Dotty Language Server
- <!-- .element: class="fragment" --> Low-level message handling done by [Eclipse LSP4J](https://github.com/eclipse/lsp4j)
- <!-- .element: class="fragment" --> Relies on interactive APIs
- <!-- .element: class="fragment" --> 0.5 KLOC
-- <!-- .element: data-transition="slide-in"  -->
```scala
override def definition(params: TextDocumentPositionParams) =













  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>












  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)











  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)
    val driver = driverFor(uri)










  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)
    val driver = driverFor(uri)
    implicit val ctx = driver.currentCtx









  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)
    val driver = driverFor(uri)
    implicit val ctx = driver.currentCtx

    val pos = sourcePosition(driver, uri, params.getPosition)







  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)
    val driver = driverFor(uri)
    implicit val ctx = driver.currentCtx

    val pos = sourcePosition(driver, uri, params.getPosition)
    val uriTrees = driver.openedTrees(uri)






  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)
    val driver = driverFor(uri)
    implicit val ctx = driver.currentCtx

    val pos = sourcePosition(driver, uri, params.getPosition)
    val uriTrees = driver.openedTrees(uri)
    val sym = Interactive.enclosingSourceSymbol(uriTrees, pos)





  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)
    val driver = driverFor(uri)
    implicit val ctx = driver.currentCtx

    val pos = sourcePosition(driver, uri, params.getPosition)
    val uriTrees = driver.openedTrees(uri)
    val sym = Interactive.enclosingSourceSymbol(uriTrees, pos)

    val classTree =
      SourceTree.fromSymbol(sym.topLevelClass.asClass).toList


  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)
    val driver = driverFor(uri)
    implicit val ctx = driver.currentCtx

    val pos = sourcePosition(driver, uri, params.getPosition)
    val uriTrees = driver.openedTrees(uri)
    val sym = Interactive.enclosingSourceSymbol(uriTrees, pos)

    val classTree =
      SourceTree.fromSymbol(sym.topLevelClass.asClass).toList
    val defTree = Interactive.definition(classTree, sym)

  }
```
-- <!-- .element: data-transition="none"  -->
```scala
override def definition(params: TextDocumentPositionParams) =
  computeAsync { cancelToken =>
    val uri = new URI(params.getTextDocument.getUri)
    val driver = driverFor(uri)
    implicit val ctx = driver.currentCtx

    val pos = sourcePosition(driver, uri, params.getPosition)
    val uriTrees = driver.openedTrees(uri)
    val sym = Interactive.enclosingSourceSymbol(uriTrees, pos)

    val classTree =
      SourceTree.fromSymbol(sym.topLevelClass.asClass).toList
    val defTree = Interactive.definition(classTree, sym)
    defTree.map(d => location(d.namePos)).asJava
  }
```

--
## Design principles
1. <span style="opacity: 0.5;"> Code reuse </span>
2. <span style="opacity: 0.5;"> Editor-agnosticity </span>
3. <span style="color: orangered;"> Easy to use (and to install!) </span>
--
<img src="images/sbt.png" width="70%" style="border: none;">

<!-- .element: style="text-align: center !important" -->
--
## sbt integration
- <!-- .element: class="fragment" --> Analyze the build to find Dotty projects
- <!-- .element: class="fragment" --> Compile these projects
- <!-- .element: class="fragment" --> Generate configuration files
- <!-- .element: class="fragment" --> Install the Dotty VSCode extension
- <!-- .element: class="fragment" --> Launch VSCode
--
## Configuration files
- `.dotty-ide-artifact`, used by the IDE extension to launch the Dotty Language Server:
```plain
ch.epfl.lamp:dotty-language-server_0.4:0.4.0-RC1
```
--
## Configuration files
- `.dotty-ide.json`, used by the DLS to launch compiler instances:

```json
[
  {
    "id" : "root/compile",
    "compilerVersion" : "0.4.0-RC1",
    "compilerArguments" : [ ],
    "sourceDirectories" : [ "src/main/scala" ],
    "dependencyClasspath" : [ ... ],
    "classDirectory" : "target/scala-0.4/classes"
  },
  {
    "id" : "root/test",
     ...
  },
  ...
]

```
--
## Future work
- <!-- .element: class="fragment" --> Optimizations
- <!-- .element: class="fragment" --> More features
  - <!-- .element: class="fragment" --> Documentation on hover
- <!-- .element: class="fragment" --> Better build tool integration (sbt server mode)
- <!-- .element: class="fragment" -->  Debugging support (based on the [Java Debug Server](https://github.com/Microsoft/java-debug))
  - <!-- .element: class="fragment" --> Evaluating Scala expressions in the debugger
--
## Conclusion
- <!-- .element: class="fragment" --> Design your compiler with interactivity in mind
- <!-- .element: class="fragment" --> Design your build tool with interactivity in mind
- <!-- .element: class="fragment" --> Interactivity should go beyond what IDEs and REPLs currently offer
  - <!-- .element: class="fragment" --> [Type Driven Development with Idris](https://www.manning.com/books/type-driven-development-with-idris)
--
## Questions ?
- More info: [dotty.epfl.ch](dotty.epfl.ch)
- Come chat with us: [gitter.im/lampepfl/dotty](http://gitter.im/lampepfl/dotty)
- Contributors welcome!
