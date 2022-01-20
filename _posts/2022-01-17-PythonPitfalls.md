---
layout: post
title: Python pitfalls for functional programmers
---

![Python]({{ site.baseurl }}/images/python.jpg)

I have been using [Python](https://www.python.org/) on and off for scripting, glue programs and simple ML notebooks. As I got more curious to get a deeper understanding of the langage and how it would scale on larger programs and I decided to use it for the [advent of code 2021](https://adventofcode.com/).

This post is the results of my observations: as I have been using mostly functional langages recently, I had quite a few surprises with Python. Although it has functional constructs, some of its non functional behaviors initially through me off and I thought it was worth sharing.

As a development environment I use VS code with the Python extension which worked very well, it was lightweigth and included a step by step debugger.

Side effects and mutable collections !

Coming with funtional habits, I took it for granted that collections were immutable and that operations on collections returned a 'new' copy of the collections. This is obviously a mental model, the implementation in functional langages is smart and based on the fact that collections are immutable, there is [possible memory reuse](https://en.wikipedia.org/wiki/Persistent_data_structure#Examples_of_persistent_data_structures).

In Python, the only method on a list that returns a list is 'copy()', c.f. https://www.w3schools.com/python/Python_ref_list.asp. The main drawback of that approach is that it is not possible to 'chain' operation, for example if I want the first 5 smallest integers in a list, in scala I would chain `sort()` and `take()` as in `l.sort().take(5)`. In Python I'll first 'sort()' (and mutate) the list, then take the first elements.

Actually Python does have some sort of immutable collection in the form of ['tuples'](https://www.w3schools.com/python/python_tuples.asp). They cannot be used as immutable list as although with the '+' operator it is possible to create a new tuple from an existing one and an element to add, for all other operations (like 'sort()') the tuple has to be converted to a list, modified and then a new one is created.


Generators !

Dynamic typing !

Dynamic typing does exist in some functional langages (like Lisp or Scheme) but as most recent functional langages like Scala or Haskell are statically typed, it is another difference that is worth keeping in mind.

The main consequence of dynamic typing is that quite a few errors are only caught when running the program (especially when refactoring). As a consequence, I started adding more unit tests, especially to test functions/methods input parameters.

Another place where dynamic typing has an impact is that it blurs the line between tuples and lists.
In statically typed functional langages, all elements of a list have the same type (I deliberately ignore inheritance that allow 'Object' list in Java as all the casting is very impractical and leads to buggy code and [HList](and [HList](https://jto.github.io/articles/getting-started-with-shapeless/) as they are not yet? mainstream) and the main advantage of a tuple is that it allows elements of different types.

In Python, a list can contain elements of any type (but is mutable) like a tuple (which is immutable).

Conclusion

Good surprise: integer