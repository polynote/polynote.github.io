---
layout: post
excerpt_separator: <!--more-->
title: Extending Polynote with your own visualizations
---

We've just published our first snapshot artifacts to Sonatype! These artifacts, `polynote-runtime` and `polynote-spark-runtime`,
are some small dependencies that support integration of other libraries with Polynote's visualization system.

In this article, we'll look at how you can use these artifacts to build a small library which lets you automatically
render rich visualizations of your own custom data structures in Polynote.

<!--more-->

## The goal

As an example, we're going to add a visualization for [Breeze](https://github.com/scalanlp/breeze) which will display
a `Matrix` using nice mathematical notation. In the end, we want a cell that looks like this:

```scala
DenseMatrix(
  DenseVector(1.0, 2.0, 3.0),
  DenseVector(2.0, 3.0, 4.0),
  DenseVector(3.0, 4.0, 5.0)
)
```

to display its result like this:

$$
\begin{bmatrix}
1.0 & 2.0 & 3.0 \\
2.0 & 3.0 & 4.0 \\
3.0 & 4.0 & 5.0
\end{bmatrix}
$$

## The code

First, create a new SBT project:

```
$ sbt new scala/scala-seed.g8
...
A minimal Scala project.

name [Scala Seed Project]: polynote-viz-example

Template applied in ./polynote-viz-example

$ cd polynote-viz-example 
```

We'll have to edit `build.sbt` in order to set up a few things:

- Change `scalaVersion` to `2.11.12`, since that's what Polynote currently builds against (hopefully it will be a more recent version soon, but it's limited by what Spark supports).
- Add the Sonatype snapshots repository – where we're currently publishing artifacts: `repositories += Resolver.sonatypeRepo("snapshots")`
- Add Breeze and `polynote-runtime` to the libraryDependencies: `libraryDependencies ++= Seq("org.scalanlp" %% "breeze" % "1.0", "org.polynote" %% "polynote-runtime % "0.2.5-SNAPSHOT")`

## The `ReprsOf` typeclass

The machinery behind Polynote's visualizations is driven by a typeclass, [`ReprsOf`](https://github.com/polynote/polynote/blob/master/polynote-runtime/src/main/scala/polynote/runtime/ReprsOf.scala).
It has one method to be implemented:

```scala
trait ReprsOf[T] extends Serializable {
  def apply(value: T): Array[ValueRepr]
}
```

Given a value of its target type, it returns an array of `ValueRepr` objects. [`ValueRepr`](https://github.com/polynote/polynote/blob/master/polynote-runtime/src/main/scala/polynote/runtime/ValueRepr.scala) is a sealed trait
which gives several options for how to represent data to the client:

* `StringRepr` is simply a text value.
* `MIMERepr` is a string value with an associated MIME type string.
* `DataRepr` is a binary-encoded data value with an associated data type.
* `LazyDataRepr` is similar to `DataRepr`, but is lazily evaluated (and each client must ask for it, while `DataRepr`s are eagerly broadcast).
* `StreamingDataRepr` represents a stream of binary data values; each client must request items from the stream. This is used for table-like data.

For the Breeze matrix, we're going to use `MIMERepr` with a LaTeX MIME type `application/x-latex`.

## Orphan instances

In order for Polynote to find `ReprsOf[DenseMatrix[Double]]`, it has to be in implicit scope for that type. So, right away
it seems we have a problem: we can't place this instance in the companion of `ReprsOf` without making `polynote-runtime`
have a dependency on Breeze; nor can we place it in the companion of `DenseMatrix` without making Breeze have a
dependency on `polynote-runtime`. We want to instead define the instance in our new module, which has a dependency
on both. But how can we get the instance into implicit scope?

The `ReprsOf` typeclass is designed to allow for this. It has a macro in implicit scope, which will search the
dependency classpath for metadata files `META-INF/expanded-scopes/polynote.runtime.ReprsOf`. This metadata file
should contain the fully-qualified name of a trait which extends `ReprsOf`, and that trait's implicit scope will be
effectively added to `ReprsOf`'s.

Let's create the `src/main/scala` directory, and add a new source file `polynote/breeze/BreezeReprsOf.scala`:

```scala
package polynote.breeze

import polynote.runtime.{ReprsOf, ValueRepr, MIMERepr}
import breeze.linalg.DenseMatrix

trait BreezeReprsOf[T] extends ReprsOf[T]

object BreezeReprsOf {
  implicit val doubleDenseMatrix: BreezeReprsOf[DenseMatrix[Double]] =
    new BreezeReprsOf[DenseMatrix[Double]] {
      def apply(value: DenseMatrix[Double]): Array[ValueRepr] = Array(
        MIMERepr(
          "application/x-latex",
          
        )
      )
    }
} 
```