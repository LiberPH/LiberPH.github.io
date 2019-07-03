---
layout: post
title: Pain free tail recursion in Scala
subtitle: “Tail recursion is its own reward”
tags: [Scala, tail-recursion, recursion]
---

I've just strated to study Scala with the aim to develop in Scala Spark. Coming from a Pyspark background 
(devoted mainly to data science related programming) I've find really hard to catch the notion of tail recursion,
so I decided to write a post in order to explain it in the easiest a most painless way possible.

I can handle recursion. However, as indicated in the course [Beginning Scala Programming](https://www.udemy.com/beginning-scala-programming/)
and the really useful post [Tail-Recursive Algorithms in Scala](https://alvinalexander.com/scala/fp-book/tail-recursive-algorithms), by Alvin Alexander,
when you need deep levels of recursion (*i.e.* multiple recursion iterations), a simple recursion function will lead you directly
to a StackOverflow (not the website :P) error.

# How can we solve this?

Well, there is tail recursion to save the day. In a very general way we can say that:

**A tail recursive function is one where the very last action is to call itself**

**Main advantage** (in Scala): Only one stack frame in a tail recursive function vs one stack frame for each
level of recursion in other recursive functions.

So... now, it may seems that any function calling itself at the last step is tail recursive, but as shown by alvin Alexander, that may be misleading. He gives an excelent example, with the function:

```Scala
def sum(list: List[Int]) : Int = list match{
  case Nil => 0
  case x :: xs => x + sum(xs)
}
``` 
One way to realize that it is not (or it is) a tail recursive function is by writing each statement one by one, where you can notice that the last thing that happens before returning is the addition of x plus the sum result. Also there a re two ways to detect if your function is tail recursive. The first one is suggested by both Alvin Alexander and the Beginning Scala course: using the annotation @tailrec as follows.

```Scala
import scala.annotation._
  def estimatePiTail(n: Int): Double = {
    @tailrec
    def helper(n: Int, sum: Int): Double = {
      if (n < 1) sum else {
        val x = math.random
        val y = math.random
        helper(n-1, sum+(if (x * x + y * y < 1) 1 else 0))
      }
    }
    helper(n, 0)/n*4
  }
```
The previous function is the tail recursive version of pi calculation. If the function was not tail recursive an error message "recursive call not in tal position" would be shown.


# Resources:

* [Programming in Scala](https://booksites.artima.com/programming_in_scala_3ed), Odersky et al. Chapter 8. Functions and closures.
* [Tail-Recursive Algorithms in Scala](https://alvinalexander.com/scala/fp-book/tail-recursive-algorithms) Accesed on July 3rd 2019.
* Web course [Beginning Scala Programming](https://www.udemy.com/beginning-scala-programming/) in Udemy, by infinite skills.
