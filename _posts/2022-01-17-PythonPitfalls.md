---
layout: post
title: Python pitfalls for functional programmers
---

![Python]({{ site.baseurl }}/images/python.jpg)

I have been using [Python](https://www.python.org/) on and off for scripting, glue programs and simple ML notebooks. 
To get a deeper understanding of the langage and evaluate how it would scale on larger programs,
 I decided to use it for the [advent of code 2021](https://adventofcode.com/).

This post is the results of my observations: 
as I have been using mostly functional langages recently, I had quite a few surprises with Python. 
Although it has functional constructs, some of its non functional behaviors initially through me off and I thought it was worth sharing.

As a development environment I used [VS code](https://code.visualstudio.com/) with the Python extension. 
VS code worked very well on both Linux and Mac, it is lightweight and includes a step by step debugger.

# Side effects and mutable collections !

Coming with funtional habits, I took it for granted that collections were immutable and that operations on collections returned a 'new' copy of the collection.
This is obviously a mental model, the implementation in functional langages is optimised. Based on the fact that collections are immutable, 
 there is [possible memory reuse](https://en.wikipedia.org/wiki/Persistent_data_structure#Examples_of_persistent_data_structures).

In Python, the only method on a list that returns a list is 'copy()', c.f. [https://www.w3schools.com/python/Python_ref_list.asp](https://www.w3schools.com/python/Python_ref_list.asp). 
The main drawback of that approach is that it is not possible to 'chain' operation.

For example, if I want the first 5 smallest integers of a list, in [Scala](https://www.scala-lang.org/) I would chain `sort()` and `take()` as in `l.sort().take(5)`. 
In Python I would to first 'sort()' (and mutate) the list, then take the first elements.
```
l.sort()
l[:5]
```

Immutable collections and wether they contain a value or a reference to value can also introduce subtle bugs.
Here is an interesting example:
```
>>> m = [[False] * 3] * 3
>>> m
[[False, False, False], [False, False, False], [False, False, False]]
>>> m[0][0]=True
>>> m
[[True, False, False], [True, False, False], [True, False, False]]
```
When `m` is created, it looks like a 2 dimensional boolean array (well list of lists to be precise !).
But when we try to modify one element (`m[0][0]`)we realise there is actually only 1 row which is referenced 3 times in the top level list !
This is because the operator `*` behaves differently whether the list on the left contains either:
* values, like with `[False] * 3` where a copy of the value False is created
* or references, like with `[[False False False] * 3]` where two new references to `[False False False]` are used,
not a [copy](https://docs.python.org/3/library/copy.html) of the list.

This behavior is consistent with the semantic of the assigment operator (`=`) but it was surprising none the less.

Actually Python does have some sort of immutable collections in the form of ['tuples'](https://www.w3schools.com/python/python_tuples.asp).
They cannot easily be used as immutable lists because there is only one operation possible on them.

With the '+' operator it is possible to create a new tuple from an existing one and an element to the list.

For all other operations which would return a new tuple (like `sort()`), they need to be converted to a list and then a new tuple needs to be constructed.


# Generators !

[Python generators](https://wiki.python.org/moin/Generators) are a generalization of [iterators](https://en.wikipedia.org/wiki/Iterator)) 
and as such they are inherently *not* functional: the `next()` operation on an iterator can return a different value every time it is called.

They are used extensively in Python as they perform well, even in 'functional' modoules like [`functools`](https://docs.python.org/3/library/functools.html#module-functools).

For example, both [`map()`]() and [`reduce()`](https://docs.python.org/3/library/functools.html#functools.reduce) return an object which is an iterator. That means the resulting collection can only be `used' (or more precisely iterated) only once.

It is very clear in the following session where the first `sum(l2)` returns the expected value `(12=1*2+3*2+3*2)` 
but the second call returns 0 because there are no more values to iterate, it is equivallent to evaluatiing `sum([])`. 
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
`l2` is not a list but an `map` class that implements an iterator.

Another side effect of this implementation (pardon the pun ;-)) is that evaluating some expressions while debugging will change the state of the program.
For example, evaluating `sum(l2)` in the debugger will 'run' the iterator.

As a consequence, to make the code more readable (or at least behaves more like I was expecting), I used [list comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)
as they make it *explicit* the result is an iterator. They can be used instead of `map()`, `filter()` and even `flatmap()` for some limited cases.
That also makes the code more Python idiomatic.

The change is quite visible in my advent of code 2021 [solutions](https://github.com/benoitpas/advent-of-code-2021).

# Dynamic typing !

Dynamic typing does exist in some functional langages (like [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)) or [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)))
but as most recent functional langages like Scala or [Haskell](https://www.haskell.org/) are statically typed, it is another difference that is worth keeping in mind.

One of the main drawbacks of dynamic typing is that quite a few errors are only caught when running the program (especially when refactoring).
For example, if I forget the paranthesis in `l.sort`, the code will run but the list will not be sorted:
```
>>> l = [3,1,4]
>>> l.sort
<built-in method sort of list object at 0x7fe670a21cc0>
>>> l
[3, 1, 4]
```

As a consequence, I started adding more unit tests, especially to test functions/methods input parameters. 
It is not great when the project gets larger as more unit tests mean more code to keep up to date which clearly has a cost.

Another place where dynamic typing has an impact is that it blurs the line between tuples and lists.
In statically typed functional langages, all elements of a list have the same type. 

I deliberately ignore inheritance that allow 'Object' list in Java as all the casting is very impractical and leads to buggy code and [HList](and [HList](https://jto.github.io/articles/getting-started-with-shapeless/) as they are not yet? mainstream) and the main advantage of a tuple is that it allows elements of different types.

In Python, a list can contain elements of any type (but is mutable) like a tuple (which is immutable).


# Conclusion

Python was easy to use (especially with VS Code and its integrated debugger), quick to learn and the libraries are very mature and efficient. 
A nice surprise was the implementation of [integer](https://docs.python.org/3/library/stdtypes.html#typesnumeric), which has unlimited (well only by the memory !) precision. 

I'm going to keep Python as a useful tool for quick prototyping/scripting but for most projects I still prefer to use a langage 
with static typing (especially one using a [Hindley-Milner type system](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system)) as the 
compiler catches a lot of issues and the code is still very readable.