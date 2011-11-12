--- 
layout: post
title: Scala and CodeKata
published: true
meta: 
  _edit_last: "2"
tags: 
- binary chop
- codekata
- Java
- JVM
- scala
type: post
status: publish
---
Recently, my buddy <a href="http://twitter.com/#!/applejack1337">Quy</a> has been raving about <a href="http://www.scala-lang.org/">Scala</a> and <a href="http://codekata.pragprog.com/">CodeKata</a>. So after much arm-twisting, I decided to check them out. Scala is a language built on top of the JVM, focusing mainly on functional programming (with the option to do imperative as well) -- all put together into a statically-typed, object-oriented package. After some reading and playing, I was sold on Scala and wanted to do some exercises to really get a feel for it. Enter CodeKata. 

CodeKata is actually a really brilliant idea. Usually programmers practice and learn on the job, but really they should be practicing all the time and honing their skills. From the site:

<blockquote>Code Kata is an attempt to bring this element of practice to software development. A kata is an exercise in karate where you repeat a form many, many times, making little improvements in each. The intent behind code kata is similar. Each is a short exercise (perhaps 30 minutes to an hour long). Some involve programming, and can be coded in many different ways. Some are open ended, and involve thinking about the issues behind programming. These are unlikely to have a single correct answer. I add a new kata every week or so. Invest some time in your craft and try them.</blockquote>

So I decided to try the <a href="http://codekata.pragprog.com/2007/01/kata_two_karate.html">karate chop</a> exercise.

<!--more-->

My first take on the binary chop was an iterative approach.

{% highlight scala %}
def chop(search: Int, searchSet: List[Int]): Int = {
  // base case -- empty set
  if (searchSet.length <= 0) {
    return -1
  }

  var min = 0
  var max = searchSet.length - 1
  do {
    // recalculate mid based on max and min
    val mid = min + (max - min) / 2

    val x = searchSet.apply(mid)
    if (x == search) {
      return mid;
    }
    else if (x > search) {
      max = mid - 1
    }
    else if (x < search) {
      min = mid + 1
    }
  } while (min <= max)

  // nothing found
  return -1;
}
{% endhighlight %}

It seems pretty standard, no functional programming at all. The next take on the algorithm was to do it in a recursive (slightly more functional) way.

{% highlight scala %}
def chop(search: Int, searchSet: List[Int]): Int = {
  // base case -- empty set
  if (searchSet.length <= 0) {
    return -1
  }

  val mid = (searchSet.length - 1) / 2
  val x = searchSet.apply(mid)

  // other base case -- found
  if (x == search) {
    return mid
  }
  if (x < search) {
    // drop the left side of the list
    chop(search, searchSet.drop(mid + 1)) match {
      case -1 => return -1
      /* since function returns the position,
       have to restore the values chopped */
      case i => return (i + mid + 1)
    }
  }
  // x > search
  else {
    // drop the right side of the list
    // (don't need to worry about position since it's not altered)
    chop(search, searchSet.dropRight(mid + 1))
  }
}
{% endhighlight %}

After these first two exercises, I still feel like I'm coding with a Java mindset. You can see from the examples, Scala is somewhat more concise but these examples don't really show its true power. From reading about it and seeing glimpses of awesome code, it definitely has the potential to be the next big player on the heels of Java. 

I'm going to try and get another implementation of this algorithm that's more concise and/or functional. If you're reading this I would love some feedback on either my implementation, or on Scala or CodeKata.
