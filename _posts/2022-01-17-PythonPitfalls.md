---
layout: post
title: Python pitfalls for functional programmers
---

![Python]({{ site.baseurl }}/images/python.jpg)

I have been using [Python](https://www.python.org/) on and off for scripting, glue programs and simple ML notebooks. 
As I got more curious to get a deeper understanding of the langage and how it would scale on larger programs,
 I decided to use it for the [advent of code 2021](https://adventofcode.com/).

This post is the results of my observations: 
as I have been using mostly functional langages recently, I had quite a few surprises with Python. 
Although it has functional constructs, some of its non functional behaviors initially through me off and I thought it was worth sharing.

As a development environment I use [VS code](https://code.visualstudio.com/) with the Python extension. 
It worked very well, it was lightweight and included a step by step debugger.

# Side effects and mutable collections !

Coming with funtional habits, I took it for granted that collections were immutable and that operations on collections returned a 'new' copy of the collections.
 This is obviously a mental model, the implementation in functional langages is smart and based on the fact that collections are immutable, 
 there is [possible memory reuse](https://en.wikipedia.org/wiki/Persistent_data_structure#Examples_of_persistent_data_structures).

In Python, the only method on a list that returns a list is 'copy()', c.f. https://www.w3schools.com/python/Python_ref_list.asp. 
The main drawback of that approach is that it is not possible to 'chain' operation.
For example if I want the first 5 smallest integers in a list, in scala I would chain `sort()` and `take()` as in `l.sort().take(5)`. 
In Python I would to first 'sort()' (and mutate) the list, then take the first elements.

Actually Python does have some sort of immutable collections in the form of ['tuples'](https://www.w3schools.com/python/python_tuples.asp).
They cannot be used as immutable list because there is only one operation possible on them:
With the '+' operator it is possible to create a new tuple from an existing one and an element to the list, 
For all other operations which would return a new tuple (like `sort()`), they need to be converted to a list and then a new tuple needs to be constructed.


# Generators !

[Python generators](https://wiki.python.org/moin/Generators) are a generalization of [ie(rators](https://en.wikipedia.org/wiki/Iterator)) 
and as such they are inherently *not* functional: the `next()` operation on an iterator can return a different value every time it is called.

They are used extensively in python as they perform well, even in 'functional' modoules like [`functools`](https://docs.python.org/3/library/functools.html#module-functools).

For example, both [`map()`]() and [`reduce()`](https://docs.python.org/3/library/functools.html#functools.reduce) return an object which is an iterator. That means the resulting collection can only be `used' (or more precisely iterated only once).

It is very clear in the following session where the first `sum(l2)` returns the expected value (12=1*2+3*2+3*2) 
but the second call returns 0 because there are no more values to iterate, it is equivallent to evaluatiing `sum([])`. 
`l2` is not a list but an `map` class that implements an iterator.
```
$ python3
>>> l = [1,2,3]
>>> l2 = map(lambda e:e*2,l)
>>> sum(l2)
12
>>> sum(l2)
0
>>> type(l2)
<class 'map'>
```

Another side effect of this implementation (pardon the pun ;-)) is that evaluating some expressions while debugging will change the state of the program.
For example, evaluation `sum(l2)` in the debugger will 'run' the iterator.

As a consequence, to make the code more readable (or at least behaves more like I was expecting), I used [list comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)
as they make it *explicit* the results is an iterator. They can be used instead of `map()`, `filter()` and even `flatmap()` for some limited cases.

# Dynamic typing !

Dynamic typing does exist in some functional langages (like Lisp or Scheme) but as most recent functional langages like Scala or Haskell are statically typed, it is another difference that is worth keeping in mind.

The main consequence of dynamic typing is that quite a few errors are only caught when running the program (especially when refactoring). 
As a consequence, I started adding more unit tests, especially to test functions/methods input parameters. 
It is not great as more unit tests mean more code to keep up to date which clearly has a cost.

Another place where dynamic typing has an impact is that it blurs the line between tuples and lists.
In statically typed functional langages, all elements of a list have the same type (I deliberately ignore inheritance that allow 'Object' list in Java as all the casting is very impractical and leads to buggy code and [HList](and [HList](https://jto.github.io/articles/getting-started-with-shapeless/) as they are not yet? mainstream) and the main advantage of a tuple is that it allows elements of different types.

In Python, a list can contain elements of any type (but is mutable) like a tuple (which is immutable).


# Conclusion

Python was easy to use (especially with VS Code and its integrated debugger), quick to learn and the libraries are very mature and efficient. 
A nice surprise was the implementation of [interger](https://docs.python.org/3/library/stdtypes.html#typesnumeric), which has unlimited (well only by the memory !) precision. 

I'm going to keep python as useful for quick prototyping/scripting but for most projects I still prefer to use a langage 
with static typing (especially one using a [Hindley-Milner type system](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system)) as the 
compiler catches a lot of issues and the code is still very readable.