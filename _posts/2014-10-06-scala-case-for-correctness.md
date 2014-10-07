---
layout: post
title: "Scala: the Case for Correctness"
meta-description: Choosing Scala should not just be for brevity or speed -- it's for correctness
---

It's only a matter of time before Java shop managers are faced with eager developers 
trying to push a new language on them. It's a hard sell. Java has mature tooling,
tons of documentation, countless stackoverflow articles, and lots of senior
developers available to fill vacancies. Combine that with your team's expertise in
Java and the rampup cost for new language, on a project that was supposed to be 
done yesterday.

Needless to say, the benefits should be substantial enough that the investment is
worth it. The typical selling points developers are going to throw out are the concise 
syntax or the functional programming but for me the biggest benefit with 
Scala is __correctness__. 

<!--more-->

The key Scala features that ensure correctness are immutability by default, easy 
concurrency, and no null values.[^1]


#### Immutability by Default


_Labelling all variables as final_

In Java, in order to make sure that methods do not mutate given parameters, they need
to be marked final. The inner variables also need to be marked as final to ensure their 
immutability.

{% highlight java %}
public void doStuff(final Integer thing1, final String thing2, final List<Integer> thing3)
{
  final String someCalculation = thing1 + thing2 + thing3.size();
  final String someOtherCalculation = thing1 + thing2;
  System.out.println(someCalculation + someOtherCalculation);
}
{% endhighlight %}

You need to jump through a few hoops to make sure everything is immutable and 
you quickly get semantic satiation for the word "final." Eventually, you'll use your
IDE to enforce this rule and probably some static analysis to verify your builds, as 
well. In Scala, everything is immutable by default:

{% highlight scala %}
def doStuff(thing1: Int, thing2: String, thing3: List[Int]) {
  val someCalculation = thing1 + thing2 + thing3.size
  val someOtherCalculation = thing1 + thing2
  println(someCalculation + someOtherCalculation)
}
{% endhighlight %}

Scala goes a step further, by defaulting to immutable collections, immutable class
members, and even immutable singletons. As a language, it has been designed with this
theme in mind.

_Primitive blocks are not expressions_

Primitive blocks in Java are executed imperatively without a return value. This is
problematic when trying to set the result to a value. For example: You cannot set 
the result of a try/catch to an immutable value.

{% highlight java %}
Integer parsedResults = 0;
try
{
  final List<Integer> results = new ArrayList<Integer>() { 
    { add(1); add(2); add(3); add(4); add(5); } 
  };
  // do some processing that may throw an exception
  parsedResults = results.size();
}
catch (final Exception e)
{
  parsedResults = 0;
}
{% endhighlight %}

In the above example, you cannot mark _parsedResults_ final because it can't get set right away.
The only workaround is to abstract the try/catch into its own method.

{% highlight scala %}
val parsedResults = try {
  val list = List(1,2,3,4,5)
  // do some processing that may throw an exception
  list.size
}
catch {
  case e: Exception => 0
}
{% endhighlight %}

Having the result be an expression simplifies the code and allows you to set the result to an immutable 
variable. Other primitive blocks like _for_, _if_ and _while_ work much the same way in Scala.

#### Easy Concurrency

If you've read Brian Goetz's 
[Java Concurrency in Practice](http://www.amazon.ca/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601)
, you realize fairly quickly that writing multi-threaded code in 
Java is terrifying. I'm pretty sure that most threading code out 
there is broken, especially my own. This is not because of bad 
programmers, it's because _writing concurrent code is hard!_

"Easy concurrency" sounds like the famous last words of a cocky 
brogrammer. What I mean by the word "easy" is that the it's easy
to reason about. The concurrency method encouraged in Scala is the actor 
model, via Akka.[^2] The biggest advantage of the actor model is not one of 
performance or clustering -- it's correctness. Having a guarantee
that the code inside an actor will only ever be run by one thread
makes the code incredibly easy to reason about.

Here's a simple threaded counter example in Scala:

{% highlight scala %}
class SimpleCounter extends Actor {
  var count = 0
  def receive = {
    case "add" => count += 1
    case "echo" => println(count)
    case _     => // do nothing
  }
}

val system = ActorSystem("system")
val counter = system.actorOf(Props(new SimpleCounter), "counter")

counter ! "add"
counter ! "add"
counter ! "add"
counter ! "echo" // 3
{% endhighlight %}

Simple, intuitive, correct. Although this is a trivial example, you'll notice
that you don't have to worry about the shared state of the _count_ variable. You
don't have to make it atomic or make the addition to it synchronized. You have a 
guarantee that only one thread will execute that code at a time. 

This doesn't mean that designing concurrent programs is necessarily easier. You still
have to figure out contention, backpressure, responsiveness and so on. However, when 
looking at any given actor, you know exactly what's going on.

#### No Null Values

The most common exception in Java application is the NullPointerException. Java has 
always had issues separating objects and primitives. Primitives cannot be set to
null, while objects get set to null all the time -- that's their default value! When 
you look at a String -- it's either a String or a null and that goes for _any_ object.

This is not a solved problem, the majority of Java programmers probably use null 
values as a control flow strategy. This fundamentally __breaks__ the
entire type system. Being able to set an object to, what should be, a completely 
different type is madness.

JDK 8 has alleviated some of this pain with an _Optional<T\>_ class that allows you
to wrap null types and unwrap them safely. Unfortunately, the _Optional<T\>_ object can 
be set to null! This means that you may have to null check the object that's supposed
to save you from getting null errors.

This is of course part of the bigger problem that Scala intrinsically solves. _Null_ is 
a type in Scala just like any other.[^3] You cannot set an object to _Null_ unless that is 
its type. If you try to set an _Int_ to _Null_, you'll get a compile error.

{% highlight console %}
scala> val x: Int = null
<console>:7: error: an expression of type Null is ineligible for implicit conversion
       val x: Int = null
{% endhighlight %}

Furthermore, because the _Option[T]_ type has been built into Scala from the beginning, every part
of the standard library supports returning optional values, _None_ or _Some[T]_, when required.

#### Correctness Matters

Increasing correctness increases productivity. Producing code that works is something we all strive
for and it's hard to argue against tools that make it easier. Despite the stability and maturity of 
Java, I would still choose Scala.


[^1]: When calling Java code you may get null values but you cannot set a value to null in Scala code unless it has the _Null_ type. 
[^2]: Akka is also available for Java but it's not part of idiomatic Java, whereas it's the standard approach in Scala.
[^3]: In reality you should never use the _Null_ type. It only exists for Java interop compatibility. Ideally you should always use the _Option[T]_ class.
