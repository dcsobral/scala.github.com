---
layout: overview-large
title: Why can't I match the type parameter of my collections?

disqus: true

partof: FAQ
num: 8
---

It's a sad fact of life on Scala that if you instantiate a List[Int], you can
verify that your instance is a List, and you can verify that any individual
element of it is an Int, but not that it is a List[Int], as can be easily
verified:

    scala> List(1,2,3) match {
         | case l : List[String] => println("A list of strings?!")
         | case _ => println("Ok")
         | }
    warning: there were unchecked warnings; re-run with -unchecked for details
    A list of strings?!

The -unchecked option puts the blame squarely on type erasure:

    scala>  List(1,2,3) match {
         |  case l : List[String] => println("A list of strings?!")
         |  case _ => println("Ok")
         |  }
    <console>:6: warning: non variable type-argument String in type pattern is unchecked since it is eliminated by erasure
            case l : List[String] => println("A list of strings?!")
                     ^
    A list of strings?!

As indicated by the warning, this happens because of _erasure_: that is, the
type parameters of a class, trait or definition are _erased_ from its type at
runtime. Let's see some examples:

    // Compile time
    val x: Option[Int] = Some(2)
    // Run time
    val x: Option = Some(Integer.valueOf(2))

    // Compile time
    def f[T](l: List[T]): T = l.head
    // Run time
    def f(l: List): Object = l.head

    // Compile time
    class C[T](val v: T)
    // Run time
    class C(val v: Object)

In short, all type parameters disappear. Whenever the type of something was a type
parameter, it gets converted to `java.lang.Object`. And if an `AnyVal`, such as
`Int`, was being passed as parameter, it gets boxed as in the first example.

And why does Scala implements this _erasure_?

Scala was defined with Type Erasure because the Java Virtual Machine (JVM),
unlike Java, did not get generics. This means that, at run time, only the type
exists, not its parameters. In the example, JVM knows it is handling a
`scala.collection.immutable.List`, but not that this list is parameterized with
`Int`.

Scala _could_ have _reified_ the type parameters, but that would require adding
an overhead to every class, trait and method with a type parameter.
Furthermore, it would be difficult to use them from Java, and Java classes and
interfaces would not benefit from that and remain erased. The benefits were not
deemed enough to compensate for these disadvantages.

There's a way to [get around type erasure](erasure.html) described in this FAQ.

This question and its answer are based on [this question](http://stackoverflow.com/q/1094173/53013) at Stack Overflow.

