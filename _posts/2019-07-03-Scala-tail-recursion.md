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
when you need deep levels of recursion (**i.e.** multiple recursion iterations), a simple recursion function will lead you directly
to a StackOverflow (not the website :P) error.

*How can we solve this?*

Well, there is tail recursion to save the day. In a very general way we can say that:

*A tail recursive function is one where the very last action is to call itself*

Main advantage (in Scala): Only one stack frame in a tail recursive function vs one stack frame for each
level of recursion in other recursive functions.
