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
--
## Design principles
- <!-- .element: class="fragment" --> Based on reusable primitives
- <!-- .element: class="fragment" --> Editor-agnostic
- <!-- .element: class="fragment" --> Easy to use (and to install!)
--
## Making Dotty interactive
<img class="fragment" style="margin-left: 130px;" src="images/phases.png">
- <!-- .element: class="fragment" --> Store trees after Typer
- <!-- .element: class="fragment" --> Respond to IDE queries by traversing trees
--
## TASTY: Typed AST serialization format
- <!-- .element: class="fragment" --> Original motivation: solve the [binary compatibility problem](https://www.slideshare.net/Odersky/scalax)
  - <!-- .element: class="fragment" --> Always use JVM bytecode: breaks when compiler encoding changes
  - <!-- .element: class="fragment" --> Always recompile source code: breaks when the typechecker changes
- <!-- .element: class="fragment" --> Can also be used to provide interactive features!

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
## sbt integration
- <!-- .element: class="fragment" --> Analyze the build to find Dotty projects
- <!-- .element: class="fragment" --> Compile these projects
- <!-- .element: class="fragment" --> Generate configuration files
- <!-- .element: class="fragment" --> Launch the IDE
--
## Configuration files
- `.dotty-ide-artifact`, used by the IDE extension to launch the DLS:
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
