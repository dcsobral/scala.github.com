---
layout: overview-large
title: How do I get around erasure?

disqus: true

partof: FAQ
num: 9
---
If you don't know what erasure is, consult [this question](erasedTypeParameters.html).

While Scala, as Java, implements _erasure_, there's a feature in Scala that
lets you get around it. It’s the **Manifest**. A Manifest is class whose
instances are objects representing types. Since these instances are objects,
you can pass them around, store them, and generally call methods on them. With
the support of implicit parameters, it becomes a very powerful tool. Take the
following example, for instance:

    object Registry {
      import scala.reflect.Manifest
      
      private var map= Map.empty[Any,(Manifest[_], Any)] 
      
      def register[T](name: Any, item: T)(implicit m: Manifest[T]) {
        map = map.updated(name, m -> item)
      }
      
      def get[T](key:Any)(implicit m : Manifest[T]): Option[T] = {
        map get key flatMap {
          case (om, s) => if (om <:< m) Some(s.asInstanceOf[T]) else None
        }
      }
    }
    
    scala> Registry.register("a", List(1,2,3))
    
    scala> Registry.get[List[Int]]("a")
    res6: Option[List[Int]] = Some(List(1, 2, 3))
    
    scala> Registry.get[List[String]]("a")
    res7: Option[List[String]] = None

When storing an element, we store a "Manifest" of it too. A Manifest is a class
whose instances represent Scala types. These objects have more information than
JVM does, which enable us to test for the full, parameterized type.

Note, however, that a `Manifest` is still an evolving feature. As an example of
its limitations, it presently doesn't know anything about variance, and assumes
everything is co-variant. I expect it will get more stable and solid once the
Scala reflection library, presently under development, gets finished.

This answer is based on an answer to [this question](http://stackoverflow.com/q/1094173/53013) on Stack Overflow.

