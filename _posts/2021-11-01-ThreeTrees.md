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
The first impression is, that despite having some functional elements like [Closures](https://groovy-lang.org/closures.html), the lack of modern functiomal constructs like [ADT](https://en.wikipedia.org/wiki/Abstract_data_type) make the language feel a bit outdated.

Actually modeling the tree, the most idiomatic design seem to use inheritance (similar to the design with [Java 6](https://github.com/benoitpas/java6-tree/tree/main/src/main/java/org/benoit)):
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

Also it worth noticing that as Groovy is an interpreted language, it took me a few runs to get all types ironed out, some of the errors would have been caught by a compiler with a fully static types language.
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
With Scala and Java experience, [Kotlin](https://kotlinlang.org/)) feels very familiar. Coming from Scala which thanks to Martin Odersky has very solid theoritical foundation, I was expecting a more ad-hoc feeling like with Groovy.

Let's see if my preconceptions are confirmed.

To model the tree, ADT support is good and the language supports immutable structures so not need to implement equals/hashcode:
``
sealed class Node<T> {
    data class Leaf<T>(val dummy: Int) : Node<T>() // Cannot use object there because of generic
    // and dataclass has to have a value

    data class Branch<T>(
        val v: T,
        val left: Node<T>,
        val right: Node<T>
    ) : Node<T>()
}
``

One limitation compared to Scala is that 'object' cannot be used with 'generics'. So although we need only one instance of 'Leaf', to make it compile we need to add a dummy parameter to make it a class. Clearly here Scala is a lot better thought out. That is an edge case but usually it is where the complexity lies, [it is the last 20% that takes 80% of the time to implement](https://en.wikipedia.org/wiki/Pareto_principle). The [implementation in Scala 3](https://github.com/benoitpas/dotty-tree/blob/main/src/main/scala/MyTree.scala#L3) is really neat in comparison.

The pattern matching, altought not as extensive as in Scala is still well thought out, in the expression after the ``is``, it is possible to directly access the fields of the type: for example although ``tree`` has type ``Node<T>``, in the ``Branch`` expresion we can use ``tree.left``, ``tree.right`` and ``tree.v``.

Tupple support is good but a bit verbose in my opinion.

``
fun <T> addId(tree: Node<T>, index: Int): Pair<Node<Pair<T, Int>>,Int> {
    return when (tree) {
        is Leaf -> Pair(Leaf<Pair<T, Int>>(0), index)
        is Branch -> {
            val newLeft = addId<T>(tree.left, index)
            val newValue = Pair(tree.v, newLeft.second)
            val newRight = addId(tree.right, newLeft.second+1)
            Pair(Branch(newValue, newLeft.first, newRight.first), newRight.second)
        }
    }
``

So Koltin is clear more than another super java and as you would expect, the support for it is great in Intellij. For other languages, especially on small projects, I prefer to use VS Code, there is just less fan noise with it ;-).

Clojure
-------
Clearly not Java !
but still same family (probably via smalltalk), first optimized VM+very good libraries (common lisp)

Used by an infamous british tabloid

Conclusion
----------

Out of curiosity I checked [Google trends](https://trends.google.com/trends/explore?date=all&q=%2Fm%2F02js86,%2Fm%2F03yb8hb,%2Fm%2F0_lcrx4) for all three languages on Tuesday 2 November 2021 and here are the resuls:

![Google trends]({{ site.baseurl }}/images/groovyClojureKotlin.png)

As expected Groovy has been the longest around and interest seems fairly constant. Clojure started to get visibility end of 2008, plateaued until beginning of 2017 and then has been going down since.

Kotlin success has been a lot faster, interest started to show beginning of 2015 (the first version was released in July 2011) but interest really shot up when Google [announced the support of Kotlin](https://techcrunch.com/2017/05/17/google-announces-the-first-preview-of-android-studio-3-0-puts-emphasis-on-speed-and-smarts/?guccounter=1&guce_referrer=aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS8&guce_referrer_sig=AQAAAHRBshWy2-8Pc1YVPvcBH1ZE9c293xHMvX1QZ0gOJ0INDNfrQs0nXCypj5u-DtsC6xAl1o1uu5S5ulpPgXmVPhAQ3JdNkIyz_9gK7WD3Yq7DOlKd5d89rJGR4XYFka9577I-2Hj7WKPYKoFooR2mgfPrIILn7XNGewavZvQ1Ndvn) in [Android Studio](https://en.wikipedia.org/wiki/Android_Studio) 3 in 2017. Since then interest has been quite high.

From a geographical perspective, I was surprised to discover that:
* Groovy is the most popular of the three in the US, Australia, Ireland and the UK.
* The only 'Clojure' majority area is Finland. As you would expect you can find some discussions on why in [reddit](https://www.reddit.com/r/Clojure/comments/ewlenr/slides_from_presentation_about_clojure_success_in/) which point to this [presentation](https://docs.google.com/presentation/d/1nCZ-GmWLcH8Dcz3XJHY00xbm42V9-rBL9n2PGkKYp0k/edit?folder=0AM-3FGnMl8asUk9PVA#slide=id.g7633f1c25d_1_155).
* Kotlin dominates the rest of the world.

