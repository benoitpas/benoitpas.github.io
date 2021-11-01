---
layout: post
title: Three trees
---

![3-trees]({{ site.baseurl }}/images/3-trees.jpg)

There are a few JVM based languages that I have either been using on and off or curious to know about so I decided to cover them in an article where I implement the tree based algorithm already covered in the previous articles on [Scala3](https://benoitpas.github.io/Dotty/), [various versions of Java](https://benoitpas.github.io/Java/) and [Rust](https://benoitpas.github.io/Rust/).

This time I decided to cover:
* [Groovy](https://groovy-lang.org/), full implementation [here](https://github.com/benoitpas/groovy-tree)
* [Kotlin](https://kotlinlang.org/), full implementation [here](https://github.com/benoitpas/kotlin-tree)
* [Clojure](https://clojure.org/)

Groovy
------
Groovy is one of the first JVM based languages and thanks to its use as Jenkins [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) has quickly widely known. I had used it a bit over the years but mostly as DSL, also when writing [Gradle](https://gradle.org/) build scripts.

I thought that would be interesting to do something a little more complicated with it.
The first impression is that despite having some functional elements like [Closures](https://groovy-lang.org/closures.html), the lack of modern functiomal constructs like [ADT](https://en.wikipedia.org/wiki/Abstract_data_type) make the language feel a bit outdated.

Actually modelling the tree, the most idiomatic design seem to use inheritance (similar to the design with [Java 6](https://github.com/benoitpas/java6-tree/tree/main/src/main/java/org/benoit)):
``
interface Tree<T> {
    Tuple2<Tree<Tuple2<T,Integer>>,Integer> addId(Integer index);
}
``
One advantage of Groovy is that types are already available for tuples which allows a more elegant implementation.

Another nice extension of Groovy is the ``@Immutable`` notation with which the implementation of 'HashCode' and 'equals' are automatically added.
``
@Immutable
class Leaf<T> implements Tree<T> {
    Tuple2<Tree<Tuple2>, Integer> addId(Integer index) {
        return new Tuple2<Tree<Tuple2>, Integer>(this,index)
    }
}
``

Still for the ``Node`` implementation we cannot use ``@Immutable`` because all class variables need to also be immutable.
So here we have to implement ``equals`` and ``hashCode`` (not implemeted here as not necessary).
``
class Node<T> implements Tree<T> {
    final private T value
    final private Tree<T> left
    final private Tree<T> right

    Node(T value, Tree left, Tree right) {
        this.value = value
        this.left = left
        this.right = right
    }

    @Override
    boolean equals(o) {
        if (this.is(o)) return true
        if (getClass() != o.class) return false

        def node = (Node<T>) o
        this.value == node.value && left == node.left && right == node.right
    }

    @Override
    Tuple2<Tree<Tuple2<T, Integer>>, Integer> addId(Integer index) {
        Tuple2<Tree<Tuple2<T,Integer>>,Integer> newLeft = left.addId(index)
        Tuple2<Tree<Tuple2<T,Integer>>,Integer> newRight = right.addId(newLeft.second)

        return new Tuple2<>(
                new Node(new Tuple2<T, Integer>(value, newRight.second),
                        newLeft.first,
                        newRight.first),
                newRight.second + 1)
    }
}
``

Another nice feature of Groovy is that it is more lax with semi-columns making the code easier to read.

All in all, Groovy is a also a good super Java and very powerful to implement DSLs. Because of its extensive use in Jenkins and as a glue language, it is not clear if it can evolve.

Kotlin
------
Another super java ?



Clojure
-------
Clearly not Java !
but still same family (probably via smalltalk), first optimized VM+very good libraries (common lisp)

Used by an infamous british tabloid