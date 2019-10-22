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

Then we'll create the `src/main/scala` directory, and add a new source file `polynote/breeze/BreezeReprs.scala`.

TODO: finish this