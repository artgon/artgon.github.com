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

When I say correctness, I mean the ability to easily and consistently write code that 
works as inteded (not the academic definition of correctness). The key Scala features 
that increase correctness are immutability by default, easy concurrency, and no null values.[^1]


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
brogrammer. What I mean by the word "easy" is that it's easy
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

The most common exception in Java applications is the _NullPointerException_ (otherwise 
know as the [billion dollar mistake](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)).
Java has always had issues separating objects and primitives. Primitives cannot be set to
null, while objects get set to null all the time -- that's their default value. When 
you look at a String -- it's either a String or a null and that goes for _any_ object.

This is not a solved problem, the majority of Java programmers probably use null 
values as a control flow strategy. This fundamentally __breaks__ the
entire type system. Being able to set an object to, what should be, a completely 
different type is madness.

JDK 8 has alleviated some of this pain with an _Optional<T\>_ class that allows you
to wrap null types and unwrap them safely. Unfortunately, the _Optional<T\>_ object can 
be set to null! This means that you may have to null check the object that's supposed
to save you from getting null errors.

There are two ways Scala mitigates the null reference issue. Firstly, you can't set
a Value Type (i.e. _AnyVal_) to null. For example: if you try to set an _Int_ to _null_, you'll 
get a compile error.

{% highlight scala %}
scala> val x: Int = null
<console>:7: error: an expression of type Null is ineligible for implicit conversion
       val x: Int = null
{% endhighlight %}

Sadly, this is not true for Reference Types (i.e. _AnyRef_). Scala allows you to set references 
to null. I assume it only exists for Java inter-op and for giving _var_ declarations an initial
value. It is generally recommended to avoid using _var_, but if you must, you can set the type 
to _Option[T]_ and the initial value to _None_.

The second way Scala mitigates null reference errors is simple -- none of the standard APIs use 
the _Null_ trait. Since the _Option[T]_ type has been built into Scala from the beginning, every 
part of the standard library supports returning optional values, _None_ or _Some[T]_, when 
required. For example when accessing a map:

{% highlight scala %}
scala> Map("a" -> 1, "b" -> 2, "c" -> 3).get("a")
res2: Option[Int] = Some(1)

scala> Map("a" -> 1, "b" -> 2, "c" -> 3).get("asdf")
res3: Option[Int] = None
{% endhighlight %}

Notice the type of the _get_ method: it returns an _Option[Int]_, which clearly signifies that
the return may or may not have a value. The equivalent method in Java, either returns the value
or _null_ if the value does not exist. This does not force the developer to handle the optional
type and may result in sporadic failures, which are difficult to test and debug.

#### Correctness Matters

The examples above may seem like unlikely selling points for switching to a new language. The reason
I picked them is because they are daily tasks that are fundamental for writing software that works.
Yes, you can write more concise code; yes, you have a more advanced type system; yes, you can pattern 
match. There are hundreds of other reasons that Scala makes a great language.[^3] When a language 
can offer me constructs to write more correct code, I'll always be willing to deal with the learning curve. 

Joshua Bloch and Venkat Subramaniam are big names in the Java community and they've
devoted books to working around problems like these:

Choose immutability:

>The functional approach may appear unnatural if you’re not familiar with it, but it enables immutability, 
>which has many advantages. __Immutable objects are simple. An immutable object can be in exactly one state, 
>the state in which it was created__. If you make sure that all constructors establish class invariants, 
>then it is guaranteed that these invariants will remain true for all time, with no further effort on your 
>part or on the part of the programmer who uses the class.

- Joshua Bloch, Effective Java (2008)

Choose actors:

>Shared mutability — the root of concurrency problems — is where multiple threads can modify a variable. Isolated 
>mutability — a nice compromise that removes most concurrency concerns — is where only one thread (or actor) can 
>access a mutable variable, ever.

- Venkat Subramaniam, Programming Concurrency on the JVM (2011)

Avoid null values:

>If an API allows nulls to exist longer, it isn't doing you any favor. It's just pushing the exception off 
>to the next API that you pass the thing to. Often, it's better to just enforce the rules uniformly. Some 
>people will complain, especially because the convention isn't completely universal. There are APIs that 
>do let you pass around nulls as an abbreviation for zero length arrays or for an empty string, etc. And those 
>APIs are in a sense bad citizens, because once you mix them with APIs that don't, you're in trouble.
>This is one of these few places where I feel like some sort of puritan. __But I have found that it's easier 
>to write robust correct systems__ if you are maybe a little less forgiving on input.

- Joshua Bloch, [In Conversation with Bill Venners](http://www.artima.com/intv/blochP.html) (2002)


Increasing correctness increases productivity. Producing code that works is something we all strive
for and it's hard to argue against tools that make it easier. There will always be skepticism about
moving off of Java, it is after all the language the JVM is designed for. However, when
you have to hack around daily programming tasks for correctness, it may be worth re-evaluating your
technology stack. Despite the stability and maturity of Java, I would still choose Scala.


[^1]: When calling Java code you may get null values but you cannot set a value to null in Scala code unless it has the _Null_ type. 
[^2]: Akka is also available for Java but it's not part of idiomatic Java, whereas it's the standard approach in Scala.
[^3]: There are certainly some pain points as well.
